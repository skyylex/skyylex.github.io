**NSObject Internals - `autorelease` and `@autoreleasePool`**

*I would like to thank my friend Mike Litvinets, who helped me with the investigation of AutoreleasePool source code*

**Introduction**

This is 3rd episode of the NSObject internals and today I will talk about autorelease and autorelease pool implementation in the Objective-C.
What is it all about? Objective-C memory management is built upon reference counting (RC) technique. RC uses counter to manage object lifetime.
Counter could be increased and decreased by some specified operations. In Objective-C it's done by sending `retain` and `release` messages to object correspondingly. 
Basically, it should be enough to create programs, there is an explicit way to control lifetime and free memory. 
However, there are situations when such explicity is inconvinient in terms of logic. The most known example is returning allocated object from function (method). 
Apple documentation provides ownership as an example of the model that could be used for memory management. If we apply such model, function which create object should also
be responsible for balancing object counter with appropriate `release` in the end of object life time. 
However, it's not easy to do without putting additional mess. If function uses `release` before returning value, memory will be freed earlier than needed.
One of the possible solutions here is to provide delayed way to `release` objects. That's exactly what `autorelease` do.

**Checking documentation**



**NSAutoreleasePool**

Autorelease mechanism is available for developers since iOS 2.0 and macOS 10.0. Initially, it was incapsulated in the NSAutoreleasePool class, which represents a collector for autoreleased objects.
Mike Ash has great article with explanation of the possible implementation for NSAutoreleasePool. I suggest to take a look at it, it definitely worth your time.

**Source code**

Notice #1: this part is written based on the Objective-c source code: https://opensource.apple.com/source/objc4/objc4-706/

Notice #2: Objective-C source code contains a lot of details related to edge cases and optimizations. And in terms of time it's inefficient trying to cover all these specifics, that's why I will be focused on the most basic parts.

Let's start with the most known part. It's definitely `autorelease` method of NSObject. If we skip edge cases such as fast autorelease and tagged pointers and other optimizations, then we will find autorelease core in the `AutoreleasePoolPage::autorelease` call:

```objective-c
__attribute__((noinline,used)) static id _objc_rootAutorelease2(id obj) {
    // ...
    return AutoreleasePoolPage::autorelease(obj);
}
```

From the first look AutoreleasePoolPage class is very similar to the functionality that we were searching for. Let's check what fields this C++ class contains:

```objective-c
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

POSIX Threads were implemented in numerous amount of Linux and BSD derivatives. So we can refer to the man pages to get description of the expected behaviour.

> The pthread_setspecific() function	associates a thread-specific value
> with a key	obtained via a previous	call to	pthread_key_create().  Different 
> threads can bind different values to the same key.  These values are
> typically pointers	to blocks of dynamically allocated memory that have
> been reserved for use by the calling thread.
>
> ------ FreeBSD Man Pages

Even at this moment we can assume that autoreleasepool is stored using these pthread functions. However, let's postpone all assumptions and go back when we have more facts to back them up.

- `static uint8_t const SCRIBBLE = 0xA3;` - usage of this key is not so obvious. And will be described later.
- Skipping `SIZE` and `COUNT`.


```
static inline void *tls_get_direct(tls_key_t k) 
{ 
    assert(is_valid_direct_key(k));

    if (_pthread_has_direct_tsd()) {
        return _pthread_getspecific_direct(k);
    } else {
        return pthread_getspecific(k);
    }
}
static inline void tls_set_direct(tls_key_t k, void *value) 
{ 
    assert(is_valid_direct_key(k));

    if (_pthread_has_direct_tsd()) {
        _pthread_setspecific_direct(k, value);
    } else {
        pthread_setspecific(k, value);
    }
}
```

Out of scope:

```
#if SUPPORT_RETURN_AUTORELEASE
    assert(tls_get_direct(AUTORELEASE_POOL_RECLAIM_KEY) == NULL);

    if (callerAcceptsFastAutorelease(__builtin_return_address(0))) {
        tls_set_direct(AUTORELEASE_POOL_RECLAIM_KEY, self);
        return self;
    }
#endif
```

**References**

- [1] Reference counting on Wikipedia - https://en.wikipedia.org/wiki/Reference_counting
- [2] - https://clang.llvm.org/docs/AutomaticReferenceCounting.html
- [3] NSAutoreleasePool class reference - https://developer.apple.com/reference/foundation/nsautoreleasepool
- [4] Using Autorelease Pool Blocks - https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html
- [5] Mike Ash, Let's build an NSAutoreleasePool - https://www.mikeash.com/pyblog/friday-qa-2011-09-02-lets-build-nsautoreleasepool.html
- [6] FreeBSD Man pages - https://www.freebsd.org/cgi/man.cgi?query=pthread_setspecific&sektion=3
