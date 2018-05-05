---
layout: default
title: Exploring Mach-O binaries. Tools - nm
description: Checking out how to deal with mach-o symbol table using nm
tags: mach-o disassembly disassembly-tools nm binaries command-line
---

**Mach-O exploration. Tools - nm**

**Introduction**

Today's plan is to take a look at another command-line tool which is helpful for binary analysis. I've already used it in a couple of posts to find specific method declared in binary. This tool is `nm`. Short description from man as usually:

> nm - display name list (symbol table). 

<br>**What is a symbol table?**

Symbol table represents identifiers from source code mapped to specific addresses. Identifiers for C-languages are functions and global variables that are defined and referenced in a program. Objective-C uses class and instance methods, but basically it's the same, if we're talking about their representation in binary.

> Note:
>
> Interesting fact which I had found during reading `nm(1)` man-page that since Xcode 8.0 release there are 2 versions of `nm`: `nm-classic` and `llvm-nm`. At the moment, default version is llvm-nm, which seems is used in this post.

<br>**Trying out**

I think the most simple way to understand something is to try and see with your own eyes. `nm` has different options that differ in formatting and additional information. However, to start it is more than enough to use call without parameters. As a target binary I'll use [binary](https://github.com/skyylex/sampler/raw/master/exploring_mach-o_binaries-tools_pagestuff/output/sampler) that was used for [previous post about pagestuff]({{site.url}}/exploring_mach-o_binaries). Also you might want to check source code before going further [here](https://github.com/skyylex/sampler/blob/master/exploring_mach-o_binaries-tools_pagestuff/sampler/main.m) it is.

```
$ nm sampler
```

Output will be:

```
0000000100000d90 t -[SampleClass .cxx_destruct]
0000000100000d30 t -[SampleClass property]
0000000100000d50 t -[SampleClass setProperty:]
                 U _NSLog
                 U _OBJC_CLASS_$_NSObject
0000000100001208 S _OBJC_CLASS_$_SampleClass
00000001000011d8 s _OBJC_IVAR_$_SampleClass._property
                 U _OBJC_METACLASS_$_NSObject
00000001000011e0 S _OBJC_METACLASS_$_SampleClass
                 U ___CFConstantStringClassReference
0000000100000000 T __mh_execute_header
                 U __objc_empty_cache
0000000100000dd0 T _main
                 U _objc_autoreleasePoolPop
                 U _objc_autoreleasePoolPush
                 U _objc_getProperty
                 U _objc_msgSend
                 U _objc_setProperty_nonatomic_copy
                 U _objc_storeStrong
                 U dyld_stub_binder
```

Basically, we have 3 columns: *address*, *type* and *symbol*. First distinct difference which I noticed is that some items from the list have no addresses, moreover these items also all have *U* type. Also we see familiar names, such as `SampleClass` class name, `property` name of property, `NSLog` function, some stuff mixed with preffixes and some `_objc_` implementation functions.

Let's examine what we have in the documentation:

> U - referenced, but not defined in the file (undefined).

All items below (marked as *U*) are functions and constants defined outside binary. `NSLog` and `NSObject` are provided by Foundation framework, most of other functions are either part of ARC implementation, Objective-C runtime and etc, and were inserted by clang.

```
U _NSLog
U _OBJC_CLASS_$_NSObject
U _OBJC_METACLASS_$_NSObject
U ___CFConstantStringClassReference
U __objc_empty_cache
U _objc_autoreleasePoolPop
U _objc_autoreleasePoolPush
U _objc_getProperty
U _objc_msgSend
U _objc_setProperty_nonatomic_copy
U _objc_storeStrong
U dyld_stub_binder
```

> T - Global function (text) object

```
0000000100000000 T __mh_execute_header
0000000100000dd0 T _main
```

`main` is clearly global function. Story with `__mh_execute_header` is a bit more complicated. I've found answer in the [ldsyms.h file](https://opensource.apple.com/source/cctools/cctools-466/include/mach-o/ldsyms.h.auto.html). Which basically says it's an address for mach header for Mach-O executable file. In other words it's border line for header. 

> t - Local function (text) object

```
0000000100000d90 t -[SampleClass .cxx_destruct]
0000000100000d30 t -[SampleClass property]
0000000100000d50 t -[SampleClass setProperty:]
```

As we'll see later, all these methods are locally defined. `property` is the property we defined and compiler generated implementation for them. `.cxx_destruct` is more interesting guest. Initially, it was part of the Objective-C++ implementation, however currently it serves as a deallocation function for both Objective-C and Objective-C++ implementations.

Surprisingly, there was no definition for `S` / `s`. So I took definition from man `nm(1)`

> S - symbol in a section other than those above

```
0000000100001208 S _OBJC_CLASS_$_SampleClass
00000001000011d8 s _OBJC_IVAR_$_SampleClass._property
00000001000011e0 S _OBJC_METACLASS_$_SampleClass
```

If you remember what `pagestuff` produced in the previous post, all these symbols were on the page 1 and were related to .DATA. Meanwhile all `t`-symbols were in `.TEXT` (page 0).

So all types are checked and now we see that types define how parts of code are linked together. Internal symbols marked as `t` / `T` / `s` / `S` are defined right in place, they all are placed inside binary. `U` symbols require more attention and should be linked to the external resource (outside binary).

The only part which left untouched is address. The address field provides information where symbol lies relatively to the assembler code. So it doesn't make any sense to consider it in isolation. We need disassembled code around existing symbol address. To do that we need to use another tool which provides disassembly code, for example `lldb `. But it's one more tool, so we'll check it out in the next post.

**More examples**

Another formatting option that I found extremely useful:
```
$ nm ./Temp/sampler -m
```

Output:

```
0000000100000d90 (__TEXT,__text) non-external -[SampleClass .cxx_destruct]
0000000100000d30 (__TEXT,__text) non-external -[SampleClass property]
0000000100000d50 (__TEXT,__text) non-external -[SampleClass setProperty:]
                 (undefined) external _NSLog (from Foundation)
                 (undefined) external _OBJC_CLASS_$_NSObject (from libobjc)
0000000100001208 (__DATA,__objc_data) external _OBJC_CLASS_$_SampleClass
00000001000011d8 (__DATA,__objc_ivar) non-external (was a private external) _OBJC_IVAR_$_SampleClass._property
                 (undefined) external _OBJC_METACLASS_$_NSObject (from libobjc)
00000001000011e0 (__DATA,__objc_data) external _OBJC_METACLASS_$_SampleClass
                 (undefined) external ___CFConstantStringClassReference (from CoreFoundation)
0000000100000000 (__TEXT,__text) [referenced dynamically] external __mh_execute_header
                 (undefined) external __objc_empty_cache (from libobjc)
0000000100000dd0 (__TEXT,__text) external _main
                 (undefined) external _objc_autoreleasePoolPop (from libobjc)
                 (undefined) external _objc_autoreleasePoolPush (from libobjc)
                 (undefined) external _objc_getProperty (from libobjc)
                 (undefined) external _objc_msgSend (from libobjc)
                 (undefined) external _objc_setProperty_nonatomic_copy (from libobjc)
                 (undefined) external _objc_storeStrong (from libobjc)
                 (undefined) external dyld_stub_binder (from libSystem)
```

Here you can see, types are replaced by segment and section, also additional column appeared that points where this symbol is defined (framework name). I used this option a lot with `grep` to find specific symbol in the [post about differences of tryLock and lock]({{site.url}}/trylock_vs_lock).

**Thank you for reading!**

**References:**

- [Understanding Object Destruction. David Chisnall](http://www.informit.com/articles/article.aspx?p=1806938&seqNum=12)
- [llvm-nm documentation](https://llvm.org/docs/CommandGuide/llvm-nm.html) 
- [Introduction to Low Level Programming and Computer Organization, Kitty Reeves](http://web.cse.ohio-state.edu/~reeves.92/CSE2421au12/SlidesDay52.pdf)
- [Symbol table at Wikipedia](https://en.wikipedia.org/wiki/Symbol_table)
- [Understanding and Analyzing Application Crash Reports](https://developer.apple.com/library/content/technotes/tn2151/_index.html)
- [Object Files and Symbols. Nick Desaulniers](http://nickdesaulniers.github.io/blog/2016/08/13/object-files-and-symbols/#disqus_thread)