---
layout: default
title: Exploring Mach-O binaries. Tools - pagestuff
---

**Exploring Mach-O binaries. Tools - pagestuff**

**Introduction**

Most of the programmers start their way in the software development by learning specific language, trying hard to understand documentation which describes key feaures and specific details, investigating available tools such as standard language libraries, sometimes they even check source code of the language implementation or standard library in order to get full understanding how things work. And I think it's correct way of self-development and all of the listed activities are valuable, but there is a problem. Documentation rarely covers all aspects, also as source code isn't always available for all of the critical parts. What to do in this case? I believe that reverse engineering and exploration of the binaries could help completing the picture in such situations. That's why today I start new series of posts related to analysis of binaries in the macOS (OS X) and iOS.

**Mach-O binaries**

Mach-O is a file format which is used in the most of the Mach-based operational systems. The history of Mach-kernel started in the Carnegie Mellon University and lately continued in the OS X / iOS as as part of hybrid XNU. There was a whole story about the transition from "classic" Mac OS to Mach-based operation system called OS X (and currently called macOS). I've put the links below to the original CMU project and related Wikipedia pages for curious. However, currently I'm mostly interested in the technical aspect located in the Mach-O structure.

> **Notice:**
> At the moment of writing this post I didn't find the Mach-O official reference at the Apple Developer. (The only document was there Mach-O Programming Topics which is helpfull, but can not replace specification completely). It's seems strange, because every file format has tons of details. So my current post is based on the information I found from unofficial sources. I hope that these documents still are relevant and the differences will not be too significant.

Specification isn't very exciting topic itselt, because such documents should provide very detailed information. Usually you will not read it as your favorite Lord of the Rings from start to the end. Most probably you will refer to the specification if you have some question. So I think it would be wise to use them in the similar way. I'll select one of the tool for binary file analysis and will try to clarify what output it produces. 

There is a whole bunch of tools to work with binaries. I prefer to start with the most standard set (it's described in the "Mach-O Programming Topics" documentation):

- /usr/bin/pagestuff
- /usr/bin/lipo
- /usr/bin/file
- /usr/bin/otool
- /usr/bin/nm

**pagestuff**

I've choosed probably the most simple tool for start, it's `pagestuff`, which has only few input parameters. The name of the tool isn't very obvious, so the short description will not be redundant. Because of the fact that I like man-pages for their clarity, let's check the description from `man pagestuff`:

> `pagestuff`  displays  information  about  the specified logical pages of a file conforming to the Mach-O executable format.  For each specified page of code, symbols (function and static data structure names) are displayed.  If no pages are specified, symbols for all pages in the __TEXT, __text section are displayed.

Ok. `man` states that Mach-O structure could be represented with a logical pages, which could be displayed by `pagestuff`. Well, let's try and see. It's obvious that for binary analysis we need a binary. Basically, for our goals it will enough a simple console application. So I've created a blank OS X console project in Xcode and executed it in order to produce an output.

Source code of the sample project:

```c
#import <Foundation/Foundation.h>

@interface SampleClass : NSObject

@property (nonatomic, copy) NSString *property;

@end

@implementation SampleClass

@end

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        // insert code here...
        NSLog(@"Hello, World!");
        
        SampleClass *class = [[SampleClass alloc] init];
        class.property = @"property223";
    }
    return 0;
}

```

- Link to the sample project: https://github.com/skyylex/sampler
- Executable file could be found at: `sampler/exploring_mach-o_binaries-tools_pagestuff/output/`

If the repo is downloaded, we are ready to try (run in Terminal).

`pagestuff executable_filepath -a`, where `executable_filepath` in my case is equal `exploring_mach-o_binaries-tools_pagestuff/output/sampler`.

The tool produces the following output:

```
File Page 0 contains Mach-O headers
File Page 0 contains contents of section (__TEXT,__text)
File Page 0 contains contents of section (__TEXT,__stubs)
File Page 0 contains contents of section (__TEXT,__stub_helper)
File Page 0 contains contents of section (__TEXT,__objc_classname)
File Page 0 contains contents of section (__TEXT,__objc_methname)
File Page 0 contains contents of section (__TEXT,__objc_methtype)
File Page 0 contains contents of section (__TEXT,__cstring)
File Page 0 contains contents of section (__TEXT,__unwind_info)
Symbols on file page 0 virtual address 0x100000d30 to 0x100000ff8
  0x0000000100000d30 -[SampleClass property]
  0x0000000100000d50 -[SampleClass setProperty:]
  0x0000000100000d90 -[SampleClass .cxx_destruct]
  0x0000000100000dd0 _main
File Page 1 contains contents of section (__DATA,__nl_symbol_ptr)
File Page 1 contains contents of section (__DATA,__la_symbol_ptr)
File Page 1 contains contents of section (__DATA,__cfstring)
File Page 1 contains contents of section (__DATA,__objc_classlist)
File Page 1 contains contents of section (__DATA,__objc_imageinfo)
File Page 1 contains contents of section (__DATA,__objc_const)
File Page 1 contains contents of section (__DATA,__objc_selrefs)
File Page 1 contains contents of section (__DATA,__objc_classrefs)
File Page 1 contains contents of section (__DATA,__objc_ivar)
File Page 1 contains contents of section (__DATA,__objc_data)
Symbols on file page 1 virtual address 0x100001000 to 0x100001230
  0x00000001000011d8 _OBJC_IVAR_$_SampleClass._property
  0x00000001000011e0 _OBJC_METACLASS_$_SampleClass
  0x0000000100001208 _OBJC_CLASS_$_SampleClass
File Page 2 contains dyld info for sliding an image
File Page 2 contains dyld info for binding symbols
File Page 2 contains dyld info for lazy bound symbols
File Page 2 contains dyld info for symbols exported by a dylib
File Page 2 contains data of function starts
File Page 2 contains symbol table for non-global symbols
File Page 2 contains symbol table for defined global symbols
File Page 2 contains symbol table for undefined symbols
File Page 2 contains indirect symbols table
File Page 2 contains string table
File Page 2 contains data of code signature
File Page 3 contains data of code signature
File Page 4 contains data of code signature
```

**Analysis**

We've got a list of "File page N ..." items with specific description for each item. I think it's good idea to check them one by one (in this post only pages 0 - 1 will be considered).

- `Mach-O headers`. Mach-O headers appears always at the beginning at the executable, and provide general information about file. Headers should be specified as:

```c
struct mach_header {
  unsigned long  magic;      /* Mach magic number identifier */
  cpu_type_t     cputype;    /* cpu specifier */
  cpu_subtype_t  cpusubtype; /* machine specifier */
  unsigned long  filetype;   /* type of file */
  unsigned long  ncmds;      /* number of load commands */
  unsigned long  sizeofcmds; /* size of all load commands */
  unsigned long  flags;      /* flags */
};
```

- `section (__TEXT,__text)` - contains executable machine code. In our case, `__text` will at least contain `main()` compiled source code and other generated by compiler procedures for `SampleClass`.
- `section (__TEXT,__stubs)` - section contains stubs with prefix `imp___stubs__`. That stubs are used in the code of `__text` section to compile procedures with external dependencies, such as system NSLog. dyld (dynamic linker) will replace such stubs on runtime with actual place in dynamic library.

- `section (__TEXT,__stub_helper)` - section is also used for the dynamic linker (dyld). It contains compiled code to jump to dyld_stub_binder implementation.
- `section (__TEXT,__objc_classname)` - contains declared in the code Objective-C class names, as string literals (for example "SampleClass")
- `section (__TEXT,__objc_methname)` - contains string literals for all generated or written Objective-C methods ("init")
- `(__TEXT,__objc_methtype)` - contains string literals for all method types (generated property method with type "NSString")
- `section (__TEXT,__cstring)` - contains string literals explicitly or implicitly declared in the code ("Hello, world!")
- `section (__TEXT,__unwind_info)`:

> The compact unwind information for the executable's code. Generated for exception handling on OS X.
>
> -- Mike Ash

- `Symbols on file page 0 virtual address 0x100000d30 to 0x100000ff8
  0x0000000100000d30 -[SampleClass property]
  0x0000000100000d50 -[SampleClass setProperty:]
  0x0000000100000d90 -[SampleClass .cxx_destruct]
  0x0000000100000dd0 _main`
  
  Specified addresses (0x0000000100000d30, 0x0000000100000d50, 0x0000000100000d90, 0x0000000100000dd0) are the actual addresses of the procedures for Objective-C methods implementations, which are placed in the `(__TEXT, __text)` section.
  
- `section (__DATA,__nl_symbol_ptr)` - contains non-lazy `dyld_stub_binder` address for `__stub_helper` section code.
- `section (__DATA,__la_symbol_ptr)` - contains addresses for dynamically lazy loaded stubs such `_NSLog` (imp___stubs__NSLog)
- `section (__DATA,__cfstring)` - contains CFString constants
- `section (__DATA,__objc_classlist)` - list of classes in form of `_OBJC_CLASS_$__ClassName (_OBJC_CLASS_$_SampleClass)`
- `section (__DATA,__objc_imageinfo)` - 

>  Specifically, OR the __DATA,__objc_imageinfo section with
   "00 00 00 00 02 00 00 00"; normally this section is all zeros.
   The __objc_imageinfo section corresponds to struct objc_image_info in:
   http://www.opensource.apple.com/source/objc4/objc4-551.1/runtime/objc-private.h
>
> -- "Fossies" - the Fresh Open Source Software Archive

- `section (__DATA,__objc_const)` - constains references for initialized constant variables declared with `const` + __objc_metaclass_xxx (__objc_metaclass_SampleClass_data) and __objc_class_xxx structs (__objc_class_SampleClass_properties:) declarations
- `section (__DATA,__objc_classrefs)` - constains references for class (objc_cls_ref_SampleClass)
- `section (__DATA,__objc_selrefs)` - constains references for selectors (@selector(setProperty:))
- `section (__DATA,__objc_selrefs)` - constains references for ivar (_OBJC_IVAR_$_SampleClass._property)
- `section (__DATA,__objc_data)` - initialized variables

It's seems that it. `__TEXT` and `__DATA` sections describe most of the executable structure. Other part (file pages 2 - 4), is absolutely another story, which we skip for today.

**Summary**

`pagestuff` provides simple approach to get the overview of the specific Mach-O file structure. We used `pagestuff filepath -a` options to get verbose form. More strict version `pagestuff filepath -p` provides it in the form of concrete offsets and lengths without description (internal section name is used instead). However, if the goal is to get detailed picture for each section/segment developer should consider using `otool` - object file displaying tool (which I will try to describe in the next post).

**Original project of the Carnegie Mellon University**

1. [Carnegie Mellon University - Mach project](http://www.cs.cmu.edu/afs/cs/project/mach/public/www/mach.html) - Unfortunately, a lot of the links are broken and some of the documentation is lost.

**Wikipedia pages**
1. [Mach kernel](https://en.wikipedia.org/wiki/Mach_(kernel))
2. [Classic Mac OS](https://en.wikipedia.org/wiki/Classic_Mac_OS#Transition_to_Mac_OS_X)

**References to Mach-O specification:**

1. [NextStep 3.3 - Mach-O specification](http://www.cilinder.be/docs/next/NeXTStep/3.3/nd/DevTools/14_MachO/MachO.htmld/index.html)
2. [Unofficial Mach-O specification](https://github.com/aidansteele/osx-abi-macho-file-format-reference)

**Other references**

1. [objc.io - Mach-O Executables](https://www.objc.io/issues/6-build-tools/mach-o-executables/)
2. [Mike Ash - Let's build Mach-O executable](https://www.mikeash.com/pyblog/friday-qa-2012-11-30-lets-build-a-mach-o-executable.html)
3. [StackOverflow question about stubs in Mach-O](http://reverseengineering.stackexchange.com/questions/8163/in-a-mach-o-executable-how-can-i-find-which-function-a-stub-targets)
4. [image_info at Fossie](https://fossies.org/linux/xscreensaver/OSX/enable_gc.c)

**Apple Open Source**

1. [Apple Open Source - unwind info](https://opensource.apple.com/source/libunwind/libunwind-35.3/include/mach-o/compact_unwind_encoding.h)

**Apple Developer documentation:**

1. [Mach-O Programming Topics](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1)
2. [Managing Memory](https://developer.apple.com/library/content/documentation/Performance/Conceptual/ManagingMemory/Articles/VMPages.html#//apple_ref/doc/uid/20001985-CJBJFIDD)
