---
layout: default
---

**NSObject Internals. Episode 1 - alloc**

**Introduction**

Today I want to start a series of posts related to investigation of the available `NSObject`'s source code. `NSObject` is one of the root classes in Objective-C and most of the core concepts of Objective-C are located there. The format of investigation will evolve, at this moment I think that the most useful description should be a combination of the info from source code and official documentation.

**Official alloc**

Let's start from the available API for the `alloc`:

`+ (id)alloc;`

`alloc` is a class method, which takes no explicit arguments and return basic `id` object. Simple enough, but not too much information here. [Apple Developer documentation](https://developer.apple.com/library/content/documentation/General/Conceptual/CocoaEncyclopedia/ObjectAllocation/ObjectAllocation.html#//apple_ref/doc/uid/TP40010810-CH7-SW1) provides more information about what alloc should do.

It says that `alloc`:

1. Calculate the required size and allocate memory for the object
2. Set retain count to 1
3. Set isa to the class pointer
4. Initialize all variables to zero
5. Return "raw" (uninitialized) pointer

Also there are some references to the aligning, virtual memory, zones.

**Under the hood**

The provided information is based on the objc4-706.tar.gz source code, available [here](https://opensource.apple.com/tarballs/objc4).

`alloc` implementation starts from the sequence of calls with passing `Class` object (pointer to struct instance of `objc_class`). It will be logical to assume that it's the key factor for object's size and layout in memory.

There are multiple branches of execution exist in the source code according to the compilation config and used objects. I will provide the flow which I think is used for current Objectice-C 2.0.

- `+ (id)alloc` -> 
- `id _objc_rootAlloc(Class cls)` -> 
- `id callAlloc(Class cls, bool checkNil, bool allocWithZone=false)` ->
- `+ (id)allocWithZone:(struct _NSZone *)zone` -> 
- `id _objc_rootAllocWithZone(Class cls, malloc_zone_t *zone)` ->
- `id _objc_rootAllocWithZone(Class cls, malloc_zone_t *zone)` ->
- `id class_createInstance(Class cls, size_t extraBytes)` ->
- `static id _class_createInstance(Class cls, size_t extraBytes)` ->
- `id _class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone)` ->
- `id _class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone)` ->
- `id objc_constructInstance(Class cls, void *bytes)`

Basically, last two C functions cover the most part of the job, so I will provide their code to go through.

    id _class_createInstanceFromZone(Class cls, size_t extraBytes, void *zone)
	{
	    void *bytes;
	    size_t size;

	    // Can't create something for nothing
	    if (!cls) return nil;

	    // Allocate and initialize
	    size = cls->alignedInstanceSize() + extraBytes;

	    // CF requires all objects be at least 16 bytes.
	    if (size < 16) size = 16;

	    if (zone) {
	        bytes = malloc_zone_calloc((malloc_zone_t *)zone, 1, size);
	    } else {
	        bytes = calloc(1, size);
	    }

	    return objc_constructInstance(cls, bytes);
    }

`_class_createInstanceFromZone` function contains 3 parts: 
- Size calculation. Basic part is retrieved in `cls->alignedInstanceSize()`. `alignedInstanceSize` doesn't perform any essential calculations, instance size is already calculated in the Class object. The only additional action over size is to align it to pointer boundary.
- Memory allocation. The default flow is a `calloc` call. One important notice that both flows use calloc versions, that means that memory not only allocated, but also is zeroed.
- Call of the `objc_constructInstance` is the final stage to work over the prepared memory.

	    id objc_constructInstance(Class cls, void *bytes)
	    {
	        if (!cls  ||  !bytes) return nil;
	        
	        id obj = (id)bytes;
	        
	        obj->initIsa(cls);

	        if (cls->hasCxxCtor()) {
	            return object_cxxConstructFromClass(obj, cls);
	        } else {
	            return obj;
	        }
	    }

Main part of `objc_constructInstance` function is `obj->initIsa(cls)` call, where `isa` struct's field is set with `Class` value. The call `cls->hasCxxCtor()` at first looks like something unclear. However, if to look over the rest of the objc project for `cxx` names it becomes clear, that all such functionality is related to Objective-C++ implementation.

**Summary**

Documentation for allocation process contains detailed description of the `NSObject's` behaviour.
However I didn't mentioned anywhere above anything about setting retain counter to 1. The reason of such miss is that `retain` doesn't exist in the `alloc` functionality at all. `NSObject` contains `SideTable` struct and related methods responsible for support of reference counting in `NSObject`. Actual implementation of the `retainCount` function has initial value equal 1. So there is no a lot of sense to mess counter with `alloc`. In reality, it doesn't touch the programmer, because from API point there is no visible difference.

**Implementation tricks and details**

- **Branch prediction.** There are a lot of places, where `fastpath()` and `slowpath()` macros are used. Macro declaration of fastpath: `#define fastpath(x) (__builtin_expect(bool(x), 1));` `__builtin_expect` could be used in `if-else` statements, to tell compiler (optimizer) improve order of instructions by providing expected value of the variable. More information could be found in the official [LLVM docs](http://llvm.org/docs/BranchWeightMetadata.html)`.

- **Disable warnings for unused function arguments**. `id _objc_rootAllocWithZone(Class cls, malloc_zone_t *zone)` doesn't use `zone` argument in the __OBJC2__ build, so to avoid warning (which in this case is useless) additional code line was added.
	
	    // allocWithZone under __OBJC2__ ignores the zone parameter
        (void)zone;

Thank you for reading!