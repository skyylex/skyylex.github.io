Actual storage is placed in the static `unsigned char *` pointer called SideTableBuf with proper size capable to fit StripedMap<SideTable>.

```c++
alignas(StripedMap<SideTable>) static uint8_t SideTableBuf[sizeof(StripedMap<SideTable>)];
```

It's not good solution, but it works. Thanks to Apple engineers for putting comments of reasons.

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

I'm not C++ man, so it was a little bit tricky for me to understand. `new` operator have a lot of variations. 
Current call is similar to so-called *placement new* operator. 

```c++
void* operator new(std::size_t, void*)
```

This version takes additional pointer parameter, which is used to refer to pre-allocated storage. 
So C++ class constructor uses this storage without allocating any other. The other effect, that the result of `new` isn't used. 
It has the same reason, because *placement new* operator use pre-allocated storage the value (void *pointer) will be the same as parameter.

And in order to use this storage, `uint8_t *` pointer is casted to actual stored class:

```c++
static StripedMap<SideTable>& SideTables() {
    return *reinterpret_cast<StripedMap<SideTable>*>(SideTableBuf);
}
```

**References:**

- [Reference page for `new`](http://en.cppreference.com/w/cpp/language/new)
