---
layout: default
title: NSObject Internals. Episode 2 - Properties (accessors)
tags: objective-c nsobject nsobject-internals property
---

**NSObject Internals. Episode 2 - Properties (accessors)**

**Introduction**

Today I continue `NSObject` investigation and take a look on the `objc-accessors.mm` file, which brings functionality related to the synthesized properties. Property is a standard part of the OOP world. In Objective-C properties are an encapsulated approach for accessing `ivar`.

**Sources analysis**

Most of the functionality is covered by the following functions:

- `id objc_getProperty(id self, SEL _cmd, ptrdiff_t offset, BOOL atomic)`
- `static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)`

These two functions provide basic operations for work with variable: to get value from variable and to set value to the variable. There is a term *mutator method*, but programmers usually use *setter* and *getter*.

The main idea of the *mutator method* is to restrict direct access to the variable and provide specific functions for that purpose. This approach allows injecting additional logic inside *getter/setter* functions. For example, inject mechanism to enable thread-safe access or manage memory allocated for `ivar`.

**Setter**

Related code:

```c++
static inline void reallySetProperty(id self, SEL _cmd, id newValue, ptrdiff_t offset, bool atomic, bool copy, bool mutableCopy)
{
    if (offset == 0) {
        object_setClass(self, newValue);
        return;
    }

    id oldValue;
    id *slot = (id*) ((char*)self + offset);

    if (copy) {
        newValue = [newValue copyWithZone:nil];
    } else if (mutableCopy) {
        newValue = [newValue mutableCopyWithZone:nil];
    } else {
        if (*slot == newValue) return;
        newValue = objc_retain(newValue);
   }

   if (!atomic) {
        oldValue = *slot;
        *slot = newValue;
    } else {
        spinlock_t& slotlock = PropertyLocks[slot];
        slotlock.lock();
        oldValue = *slot;
        *slot = newValue;        
        slotlock.unlock();
    }

    objc_release(oldValue);
}
```

Use of the property in program flow starts from receiving first value in the setter method (`reallySetProperty` in our case). Basically, the simplest form of setter function needs to have only a pointer to the target `ivar` and value to set.
`reallySetProperty` contains more parameters:

- `id self` - required part (1st) of the target field pointer
- `SEL _cmd` - argument isn't used in the function.
- `id newValue` - value to set
- `ptrdiff_t offset` - required part (2nd) of the target field pointer
- `bool atomic` - additional logic
- `bool copy` - additional logic
- `bool mutableCopy` - additional logic

However, all required parts (composite pointer to the target field + value to set) are still there. Additional parameters will be described later.

The `reallySetProperty` could be split into several steps:

1. Handle request to update class if needed
2. Build a pointer to the target field
3. Prepare new value + memory management
4. Set new value in the appropriate way (`atomic` / `nonatomic`)
5. Release old value

**Setter step #1. Class update**

Related code:
```c++
if (offset == 0) {
    object_setClass(self, newValue);
    return;
}
```

Clear explanation of this step requires additional information. Target variable for update is located in the `id self`, if you check `id` type declaration you will see:
```c++
typedef struct objc_object *id;
```

Therefore, it's a C-struct. And this gives us more information for analysis. Because struct has essential feature related to layout in memory (C99 standard):

> Within a structure object, the non-bit-ﬁeld members and the units in which bit-ﬁelds reside have addresses that increase in the order in which they are declared.

That means that first declared member will have the first position in memory (zero offset). If you take a look at the `objc_object` declaration, you will see `isa_t isa`. It's easy to find what is that from the Apple documentation:

> When a new object is created, memory for it is allocated, and its instance variables are initialized. First among the object’s variables is a pointer to its class structure. This pointer, called isa, gives the object access to its class and, through the class, to all the classes it inherits from.

Ok, now it's clear, that request for update using offset == 0 is a class variable update. However, why is it required to call separate method `object_setClass`? There are several assumptions for that. First one, that classes are runtime entities and they need to be handled by runtime system (they should be loaded and initialized, and etc), so Objective-C runtime should prevent direct access to isa. Second one, `isa_t` type isn't used in the programmer space. Objective-C programmers operate with `Class`.

**Notice**: it's worth to mention, that child classes of `NSObject` and `NSObject` are not identical. However, `isa` is located at the beginning of layout there too. It's easy to check in *Xcode lldb*: create an object of NSObject subclass. And compare output of the `isa` address and `object` address.
- `po &(object->isa)`
- `po &(object)`

**Setter step #2. Build a pointer to the target field**

Related code:

```c++
id oldValue;
id *slot = (id*) ((char*)self + offset);
```

Calculation of the pointer is rather trivial, just need to calculate necessary address using a base address and offset. The only interesting issue for me is why does Objective-C implementation use offset at all? (Another option using direct reference). I have no facts here, my assumption that calculation of the instance variable address doesn't cost too much in terms of processor time, in the same time such referencing is very flexible against address changes and could be easily verified. Also, such technique potentially will use less memory, because offset can have a small type, based on the known total layout size. **Update to the assumption is provided at the end of the article**

**Setter step #3. Prepare new value + memory management**

Another branching in the code takes place. The same as it's described in the apple docs, if you place copy qualificator - value is copied and isn't retained, because copy already set to 1 reference counter during creation.

```c++
if (copy) {
    newValue = [newValue copyWithZone:nil];
} else if (mutableCopy) {
    newValue = [newValue mutableCopyWithZone:nil];
} else {
    if (*slot == newValue) return;
    newValue = objc_retain(newValue);
}
```

**Setter step #4. Set new value in the appropriate way (`atomic` / `nonatomic`)**

It's the core functionality to change value of `ivar`. There is a probability of getting into this code line from multiple threads. So `atomic` behavior prevents from Read-Write and Write-Write classic conflicts, and your application will not crash because of that reasons. However, thread-safety here doesn't make any guaranty that your code will work properly. Synchronization logic of the specific application is a programmer's responsibility.

**Notice**: `atomic` is the default behaviour for all properties. And the downside is common for all thread synchronization techniques, all locks make a code a little bit slower.

```c++
if (!atomic) {
    oldValue = *slot;
    *slot = newValue;
} else {
    spinlock_t& slotlock = PropertyLocks[slot];
    slotlock.lock();
    oldValue = *slot;
    *slot = newValue;        
    slotlock.unlock();
}
```

Before setting the value, an old one is saved to handle related memory. After that pointer is dereferenced and a new value is applied.

**Setter step 5. Release old value**

```c++
objc_release(oldValue);
```

Step #4 saved old pointer to the memory and current step is to release it. All setter branches of execution create a pointer with +1 at step #3 via copy or retain, so release pair should be applied in setter method explicitly. Another place requires release is at the end of the `self` pointer life-cycle. An earlier versions of Objective-C (with MRC) required this job to be done in `-(void)dealloc`. However, current Apple documentation ensures that runtume handles it itself.

> Do I still need to write dealloc methods for my objects?
Maybe.
Because ARC does not automate malloc/free, management of the lifetime of Core Foundation objects, file descriptors, and so on, you still free such resources by writing a dealloc method.
You do not have to (indeed cannot) release instance variables, but you may need to invoke [self setDelegate:nil] on system classes and other code that isn’t compiled using ARC.
dealloc methods in ARC do not require—or allow—a call to [super dealloc]; the chaining to super is handled and enforced by the runtime.

**Getter**

Related code:

```c++
id objc_getProperty(id self, SEL _cmd, ptrdiff_t offset, BOOL atomic) {
    if (offset == 0) {
        return object_getClass(self);
    }
    
    // Retain release world
    id *slot = (id*) ((char*)self + offset);
    if (!atomic) return *slot;
        
    // Atomic retain release world
    spinlock_t& slotlock = PropertyLocks[slot];
    slotlock.lock();
    id value = objc_retain(*slot);
    slotlock.unlock();
    
    // for performance, we (safely) issue the autorelease OUTSIDE of the spinlock.
    return objc_autoreleaseReturnValue(value);
}
```

The flow for the getter is quite similar. Steps #1 and #2 are almost the same as in setter. The difference is the atomic/nonatomic behavior. Thread-safety isn't taken into account. It follows setter logic (to prevent read-write conflict). I mean that atomic object is retained and nonatomic is not. Basically, setter is already retaining the `value`, so in the simple case value doesn't need to retained/released once again. And nonatomic approach probably was planned as a flow when another thread doesn't release the `value`. `atomic` was built over the idea to make able to grab value inside the lock and to allow to return it from function ignoring other threads activity.

**UPDATE October 14, 2017**

Thanks to Denis Morozkin, who provided explanation and references about why offset is used. The reason is that Objective-C is able to change class implementation on runtime and obviously such significant changes to class require additional flexibility from the related data. Offset that could be verified and recalculated on runtime provides required support.

More information - [Non-fragile ivars](http://www.sealiesoftware.com/blog/archive/2009/01/27/objc_explain_Non-fragile_ivars.html)

**Tips and tricks:**
- All locks are stored in the map called `PropertyLocks` of type `StripedMap`, where `void *` is the key for the lock value. However, main map storage, specified as static array, contains only up to 64 items. It means that limit for properties is 64 per class or that each item in array contains also additional structure (for example: linked list).

**References**:

- [Apple Developer Documentation. EncapsulatingData](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/EncapsulatingData/EncapsulatingData.html)
- [Apple Developer Documentation. Transition to ARC. FAQ](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011226-CH1-SW17)
- [Apple Developer Documentation. Accessor methods](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW4)
- [Apple Developer Documentation. Objective-C Runtime Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide)
- [Wikipedia. Mutator method](https://en.wikipedia.org/wiki/Mutator_method)

**Thank you for reading!**

**[<<<< previous episode]({{site.url}}/NSObject_internals_-_alloc) - [next episode >>>>]({{site.url}}/nsobject-internals-autorelease_and_autoreleasepool)**
