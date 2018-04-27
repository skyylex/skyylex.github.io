---
layout: default
title: NSObject Internals. Episode 3 - autorelease and @autoreleasePool
description: New episode covers one of the aspects of memory management in Objective-C - autorelease and autorelease pool
tags: objective-c nsobject nsobject-internals autorelease autorelease-pool memory-management
---
**NSObject Internals. Episode 3 - `autorelease` and `@autoreleasePool`**

*I would like to thank my friend Mike Litvinets, who helped me with the investigation of AutoreleasePool source code*

**Introduction**

This is 3rd episode of the `NSObject` internals and today I will talk about autorelease and autorelease pool implementation in the Objective-C.
What is it all about? Objective-C memory management is built upon reference counting (RC) technique. RC uses counter to manage object lifetime.
Counter could be increased and decreased by some specified operations. In Objective-C it's done by sending `retain` and `release` messages to object correspondingly. 
Basically, it should be enough to create programs, there is an explicit way to control lifetime and free memory. 
However, there are situations when such explicity is inconvinient in terms of logic. The most known example is returning allocated object from function (method). 
Apple documentation provides ownership as an example of the model that could be used for memory management. If we apply such model, function which create object should also
be responsible for balancing object counter with appropriate `release` in the end of object life time. 
However, it's not easy to do without putting additional mess. If function uses `release` before returning value, memory will be freed earlier than needed.
One of the possible solutions here is to provide delayed way to `release` objects. That's exactly what `autorelease` do.

**NSAutoreleasePool**

Autorelease mechanism is available for developers since iOS 2.0 and macOS 10.0. Initially, it was incapsulated in the `NSAutoreleasePool` class, which represents a collector for autoreleased objects.
Mike Ash has great article with explanation of the possible implementation for `NSAutoreleasePool`. I suggest to take a look at it, it definitely worth your time.

**Source code**

Notice #1: this part is written based on the Objective-c source code: https://opensource.apple.com/source/objc4/objc4-706/

Notice #2: Objective-C source code contains a lot of details related to edge cases and optimizations. And in terms of time it's inefficient trying to cover all these specifics, that's why I will be focused on the most basic parts.

Let's start with the most known part. It's definitely `autorelease` method of `NSObject`. If we skip edge cases such as fast autorelease and tagged pointers and other optimizations, then we will find autorelease core in the `AutoreleasePoolPage::autorelease` call:

```c++
__attribute__((noinline,used)) static id _objc_rootAutorelease2(id obj) {
    // ...
    return AutoreleasePoolPage::autorelease(obj);
}
```

From the first look `AutoreleasePoolPage` class is very similar to the functionality that we were searching for. Let's check what fields this C++ class contains:

```c++
class AutoreleasePoolPage 
{

#define POOL_SENTINEL 0
    static pthread_key_t const key = AUTORELEASE_POOL_KEY;
    static uint8_t const SCRIBBLE = 0xA3;  // 0xA3A3A3A3 after releasing
    static size_t const SIZE = 
#if PROTECT_AUTORELEASEPOOL
        4096;  // must be multiple of vm page size
#else
        4096;  // size and alignment, power of 2
#endif
    static size_t const COUNT = SIZE / sizeof(id);

    magic_t const magic;
    id *next;
    pthread_t const thread;
    AutoreleasePoolPage * const parent;
    AutoreleasePoolPage *child;
    uint32_t const depth;
    uint32_t hiwat;
...
}
```

- ` static pthread_key_t const key = AUTORELEASE_POOL_KEY` - `key` field is obviously a key. To what? Variable type `pthread_key_t` is a part of the pthread library, which is implemenation of the POSIX Threads. Quick search over the Internet gave me the following related functions:
  - `int pthread_key_create(pthread_key_t *key, void (*destructor)(void*));`
  - `void *pthread_getspecific(pthread_key_t key);`
  - `int pthread_setspecific(pthread_key_t key, const void *value);`

POSIX Threads were implemented in a numerous amount of Linux and BSD derivatives. So we can refer to the man pages to get description of the expected behaviour.

> The pthread_setspecific() function	associates a thread-specific value
> with a key	obtained via a previous	call to	pthread_key_create().  Different 
> threads can bind different values to the same key.  These values are
> typically pointers	to blocks of dynamically allocated memory that have
> been reserved for use by the calling thread.
>
> ------ FreeBSD Man Pages

Even at this moment we can assume that autoreleasepool is stored using these pthread functions. However, let's postpone all assumptions and go back when we have more facts to back them up. 

- `static size_t const SIZE = PAGE_MAX_SIZE;` - actual size of the allocated pool page.
- `id *next` - pointer to the current autoreleased object. When autorelease pool needs to keep track on one more autoreleased object, it shifts `next` pointer value with and store object pointer: `*next++ = obj;`.
- `pthread_t const thread;` - initialized with `pthread_self()` value, which is POSIX descriptor of the current thread. Used as a guard variable for verification purposes. `AutoreleasePoolPage` has a lot of verifications inside implementation to crash execution thread in case of any diff between initial stored thread descriptor and actual execution thread.
- `AutoreleasePoolPage *const parent;` and `AutoreleasePoolPage *child;` - pointers which provide linked list implementation of the autoreleasePoolPages.

This information was retrieved by investigation of the separate fields usage and analysis of their declaration. However, we have also available methods and execution flow, which could provide us another side of the picture.

Let's go back to the `AutoreleasePoolPage::autorelease`.

- `static inline id autorelease(id obj)` - is just a wrapper with a few simple verifications calls `autoreleaseFast`
    - `static inline id autoreleaseFast(id obj)` - this function presents actual logic to us:
    
`AutoreleasePoolPage` provides a pagination mechanism to fill autorelease pool with objects using page portions. There are two types of pages:
    
- `static inline AutoreleasePoolPage *hotPage();` - this page is the most recent AutoreleasePoolPage instance stored right into the thread specific memory.
- `static inline AutoreleasePoolPage *coldPage();` - is the most "old" page.

Pages are used as a containers for autorelease object pointers. What does it mean? Page has explicit size defined with `SIZE` field. It rather simple pointer arithmetic to get object memory bounds. Initially pointer has start address of the allocated for class memory, it's lower bound. If we know size (and we know), we can calculate upper bound by simple addition. So we know how much objects can we place. From the class declaration we know that `next` is responsible for keeping pointers to the autoreleased objects. To simplify explanation I created rough layout of the `AutoreleasePoolPage`:

![AutoreleasePoolPage Layout]({{ site.url }}/assets/autoreleasepoolpage_layout.png)

Initially, `next` is empty and points to the first slot for `(id *)autoreleased` object. It's right behind the values of the instance variables section . When object arrives for autoreleasing `next` pointer value is filled with the pointer to the autoreleased object and after that shifted to the next cell using C-based pointers arithmetic.

Ok, the most usual case when there is at least one existing pool page is explained. But if initially there is no pool page, what's in that case? And what about full page issue? The answer is pretty clear new empty `AutoreleasePoolPage` should be created. In both cases it will stored as a hot page, the difference is that the 2nd case with full page will set pointers for `page->parent` and `page->child` to keep linked list sequence. Linked list is a way to hold all pages using correct execution order.

So the execution tree looks like:

- `autorelease(obj)`
    - `autoreleaseFast(obj)`
        - `page = hotPage()`
        - `if (page && !page->full())`
            - `id *add(id obj)`
        - `else if (page)`
            - `autoreleaseFullPage(obj, page);`
                - `page = new AutoreleasePoolPage(page);`
                - `setHotPage(page);`
                - `page->add(obj)`
        - `else `
            - `id *autoreleaseNoPage(id obj)`
                - `page = new AutoreleasePoolPage(nil);`
                - `setHotPage(page);`
                - `page->add(obj);`
    
However, `autorelease` is about delayed releasing and at the moment we checked how pool collects information about autoreleased objects. From documentation it's clear that releasing should be performed at the end of scope. Source code contains the following method:

- `static inline void pop(void *token)`, which performs clean up of the pool pages that are later than page by token.

The simplified execution tree looks like:

- `pop(token)`
    - `page = pageForPointer(token);` // get boundary page
    - `page->releaseUntil((id *)token);` // release all related autoreleased objects
        - `while token != page-> next
            - `while (page->empty())`
                - `page = page->parent;`
                - `setHotPage(page);`
            - `id obj = *--page->next;`
            - `objc_release(obj);` // the most important part
        - `setHotPage(this);`
     - `page->child->kill();` // destroy all child pages
        - `while (page->child) { page = page->child; }` // go to the deepest page
            - `deathptr = page;`
            - `page = page->parent;`
            - `page->child = nil;`
            - `delete deathptr;`
        - `while (deathptr != this);` // go back and destroy if all later pages

It's 2 step clean up. First move deeply to the newest pages which are below the boundary page and perfom release for all stored there object. After all required autorelease objects will be released, destroy all related pages. Pretty simple, isn't it?

**Summary.**

As it was described basic ideas are pretty simple: linked list data structure for storing autoreleased objects and pages, pagination to use only required amount of memory and allocate more if needed, and simple pointers arithmetic. Implementation off course contains a lot of hidden optimizations and verifications, some parts are just magic values.

**Tips and tricks**

About magic. As you probably noticed there was a magic in the `AutoreleasePoolPage`. I mean real magic. I mean real `magic` field. Remember?

- `magic_t const magic;`

And related declaration of the `magic_t`:

```c++
struct magic_t {
    static const uint32_t M0 = 0xA1A1A1A1;
#   define M1 "AUTORELEASE!"
    static const size_t M1_len = 12;
    uint32_t m[4];
    
    magic_t() {
        assert(M1_len == strlen(M1));
        assert(M1_len == 3 * sizeof(m[1]));

        m[0] = M0;
        strncpy((char *)&m[1], M1, M1_len);
    }

    ~magic_t() {
        m[0] = m[1] = m[2] = m[3] = 0;
    }

    bool check() const {
        return (m[0] == M0 && 0 == strncmp((char *)&m[1], M1, M1_len));
    }

    bool fastcheck() const {
#if DEBUG
        return check();
#else
        return (m[0] == M0);
#endif
    }

#   undef M1
};
```

I'm not expert in magic, so here it's only assumption. First of all it's clear that this `struct` itself isn't very useful. Initializing variable with equal values and checking them for equality doesn't make a lot of sense. The only reason I see when something is going wrong with memory and allocated memory borders are broken, memory could be overwritten. 

So how this field is used? 
```c++
void check(bool die = true) 
{
    if (!magic.check() || !pthread_equal(thread, pthread_self())) {
        busted(die);
    }
}
```

And `busted` even more plainly confirms that:

```c++
void busted(bool die = true) {
// ...
_objc_fatal("autorelease pool page %p corrupted\n"
             "  magic     0x%08x 0x%08x 0x%08x 0x%08x\n"
             "  should be 0x%08x 0x%08x 0x%08x 0x%08x\n"
             "  pthread   %p\n"
             "  should be %p\n")
// ... 
}
```

**P.S.**

The most basic details were covered here, however, there are some difficultiles with mapping current picture of available source code with disassembly. ARC runtime support functions are actual citizens there. So it's interesting challenge to collect puzzle from two such different sides. Who knows, may be in some of the future posts?

**References**

- [Reference counting on Wikipedia](https://en.wikipedia.org/wiki/Reference_counting)
- [Clang Automatic Reference Counting](https://clang.llvm.org/docs/AutomaticReferenceCounting.html)
- [NSAutoreleasePool class reference](https://developer.apple.com/reference/foundation/nsautoreleasepool)
- [Mike Ash, Let's build an NSAutoreleasePool](https://www.mikeash.com/pyblog/friday-qa-2011-09-02-lets-build-nsautoreleasepool.html)
- [FreeBSD Man pages](https://www.freebsd.org/cgi/man.cgi?query=pthread_setspecific&sektion=3)
