**Introduction**

Today is the 4th episode of NSObject Internals and we'll touch `retain` and `release`. [Previous episode](https://skyylex.github.io/nsobject-internals-autorelease_and_autoreleasepool) has already described some of the reference counting specifics, so to not repeat myself I will immediately jump into the details.

**Notice:** Topic is very broad, so I will talk only about direct usage of `retain` and `release` methods. 

**Source code**

As usual let's see what is hidden inside `retain` by examination of the possible execution tree:

- `(id)retain`
    - `inline id objc_object::rootRetain()`
        - `// ... something about deprecated Garbage Collection`
        - `// ... something about tagged pointers`
        - `id objc_object::sidetable_retain()`
        
Here I should stop. We have already met with `objc_object`, not personally. Anyway, it's what lies behind black-boxed `id` struct.

```c++
typedef struct objc_object *id;
```

But today is not `objc_object` day, it's interesting topic deserves a separate post and we'll cover it one day. Go back to the `retain`, its core implementation lies in the `sideTable_retain()` method:

```c++
id objc_object::sidetable_retain()
    // ..
    SideTable& table = SideTables()[this];

    if (table.trylock()) {
        size_t& refcntStorage = table.refcnts[this];
        if (! (refcntStorage & SIDE_TABLE_RC_PINNED)) {
            refcntStorage += SIDE_TABLE_RC_ONE;
        }
        table.unlock();
        return (id)this;
    }
    return sidetable_retain_slow(table);
}
```

Actually, if we skip some implementation details such as what is `SideTable`, `refcnts`, `SIDE_TABLE_RC_PINNED`, we'll see synchronized way to increase counter. So retain is the same as what is written in the documentation, just increasing counter by one. But we came here, not only to verify that, so let's try to look around and see what we can find.

/// #TBD SIDE_TABLE_RC_PINNED

Interesting point here is that initially `sidetable_retain` uses "fast" way to increase counter via `.tryLock()` and if it's locked, then jump to `sidetable_retain_slow()`. But `sidetable_retain_slow` is almost the same. Key differences are that `sidetable_retain_slow` gets `table` as a parameter (versus direct call to SideTables in `sidetable_retain`) and uses `lock` instead of `tryLock`. 

Actual storage is placed in the static `unsigned char *` pointer called `SideTableBuf` with a proper size capable to fit `StripedMap<SideTable>`.

```c++
alignas(StripedMap<SideTable>) static uint8_t SideTableBuf[sizeof(StripedMap<SideTable>)];
```

The solution above is not good, but it works. Thanks to Apple engineers for putting comments of reasons.

> // We cannot use a C++ static initializer to initialize SideTables because <br>
> // libc calls us before our C++ initializers run. We also don't want a global <br>
> // pointer to this struct because of the extra indirection. <br>
> // Do it the hard way.

Initialization is performed using simple `static` function. 

```c++
static void SideTableInit() {
    new (SideTableBuf) StripedMap<SideTable>();
}
```

I'm not C++ man, so it was a little bit tricky for me to understand. Generally, `new` operator has a lot of variations. Current variant is similar to so-called *placement new* operator. 

```c++
void* operator new(std::size_t, void*)
```

This version takes additional pointer parameter, which is used to refer to pre-allocated storage. C++ class constructor uses this storage without allocating any other. The other effect, that there is no need to use returned result of `new`. 
It has the same reason, *placement new* operator uses pre-allocated storage via passed pointer, so returned pointer will be the same as argument.

And in order to use this storage, `uint8_t *` pointer is casted to actual stored class:

```c++
static StripedMap<SideTable>& SideTables() {
    return *reinterpret_cast<StripedMap<SideTable>*>(SideTableBuf);
}
```

What's about `StripedMap`?
```c++
template<typename T>
class StripedMap {
    enum { CacheLineSize = 64 };
// ..
    enum { StripeCount = 64 };
// ..
    struct PaddedT {
        T value alignas(CacheLineSize);
    };

    PaddedT array[StripeCount];

    static unsigned int indexForPointer(const void *p) {
        uintptr_t addr = reinterpret_cast<uintptr_t>(p);
        return ((addr >> 4) ^ (addr >> 9)) % StripeCount;
    }
    
 public:
    T& operator[] (const void *p) { 
        return array[indexForPointer(p)].value; 
    }
    const T& operator[] (const void *p) const { 
        return const_cast<StripedMap<T>>(this)[p]; 
    }
// ...
};
```

`StripedMap` is the implementation of a dictionary (map) class, where `void *` pointer is a key, and `T` is a template value. Usually, programmers deal with a hash-map. Hash-map has two valuable specifics: 

- this kind of map depends heavily on the quality of hash-function
- there is a mechanism for conflict's solving, when two keys have the same hash. One of the ways is to put store linked list of key-values, instead of value.

`StripedMap` has something like a hash-function, it is the following expression:

```c++
((addr >> 4) ^ (addr >> 9)) % StripeCount
```

Right shift (`>>`) and xor (`^`) operators can be executed as a few instructions on the most of the modern CPUs, and the modulus operator (`%`) also rather trivial. It means that this expression is calculated pretty fast. To get some overview of the statistical distribution, I used naive simulation to see how pseudo hashes are distributed over the array. 

I have created ordinary for loop and used malloc to create new unique pointer, and passed this pointer address as input parameter to pseudo-hash function. (Important point here to not free these memory, because you OS will re-use freed memory and you'll get the same value multiple times.). I varied parameters such as pointer values count (amount of iterations), different chunk sizes for allocations. In all cases distribution was close to the discrete uniform distribution (equal or almost equal amount of matches on all positions). Consequently, we can think about this pseudo-hash as a hash function.

Second point is the most interesting. As I said previously, hash-maps usually contains mechanism for handling conflicts during filling value for key and keeping data structure able to return value for key later. `StripedMap` is a different thing. It is an internal storage implemented as an statically-defined array. That means that internal storage will be allocated and filled at the end of StripeMap creation. Also it's clear that this is read-only data structure, developer can only get value by key using overloaded operators in `public:` class interface section. So if we collect these facts and keep in mind word `striping`, it's become clear what this class actually does. It splits access to certain recources based on the pointer. Resources are counted as equal, so the main goal to have stable repeatable access to the same resources each time (no matter what exactly this resource will be, the main thing that it's the same). This info was partially described in the comment in source code:

> // StripedMap<T> is a map of void* -> T, sized appropriately <br/>
> // for cache-friendly lock striping. <br/>
> // For example, this may be used as StripedMap<spinlock_t> <br/>
> // or as StripedMap<SomeStruct> where SomeStruct stores a spin lock.

**For curious:** if you take a look at the earlier versions of Objective-C source code you will see no StripedMap. Most probably that this striping performance optimization was included for actuall necessity. SideTable uses synchronization which become a bottle neck in programming language with reference counting.

**References:**

- [Reference for `alignas`](http://en.cppreference.com/w/cpp/language/alignas)
- [Reference page for `new`](http://en.cppreference.com/w/cpp/language/new)
