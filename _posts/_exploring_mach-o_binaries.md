---
layout: default
title: Exploring Mach-O binaries. Tools - pagestuff
---

### Introduction

Most of the programmers start their way in the software development by learning specific language, trying hard to understand documentation which describes key feaures and specific details, investigating available tools such as standard language libraries, sometimes they even check source code of the language implementation or standard library in order to get full understanding how things work. And I think it's correct way and all of the listed activities are valuable, but there is a problem. Documentation rarely covers all aspects, also as source code isn't always available for all of the critical parts. What to do in this case? I believe that reverse engineering and exploration of the binaries could help completing the picture in such situations. That's why today I start new series of post related to binaries in the macOS (OS X) and iOS.

### Mach-O binaries

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

### pagestuff

I've choosed probably the most simple tool for start, it's `pagestuff`, which has only few input parameters. The name of the tool isn't very obvious, so the short description will not be redundant. Because of the fact that I like man-pages for their clarity, let's just check the description from `man pagestuff`:

> `pagestuff`  displays  information  about  the specified logical pages of a file conforming to the Mach-O executable format.  For each specified page of code, symbols (function and static data structure names) are displayed.  If no pages are specified, symbols for all pages in the __TEXT, __text section are displayed.

Ok. `man` states that Mach-O format contains from the logical pages, which could be displayed by `pagestuff`. Let's try and see. It's obvious that for binary analysis we need a binary. Basically, for our goals it will enough a simple console application.

- Link to the sample project: https://github.com/skyylex/sampler
- Executable file could be found at: `sampler/exploring_mach-o_binaries-tools_pagestuff/output/`

If the repo is downloaded, we are ready to try (in Terminal):

`pagestuff executable_filepath -a`

where `executable_filepath` in my case is equal `exploring_mach-o_binaries-tools_pagestuff/output/sampler`

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

**Original project of the Carnegie Mellon University**

1. http://www.cs.cmu.edu/afs/cs/project/mach/public/www/mach.html - Unfortunately, a lot of the links are broken and some of the documentation is lost.

**References to Mach-O specification:**

1. http://www.cilinder.be/docs/next/NeXTStep/3.3/nd/DevTools/14_MachO/MachO.htmld/index.html
2. https://github.com/aidansteele/osx-abi-macho-file-format-reference

**Apple Developer documentation:**

1. https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1
2. https://developer.apple.com/library/content/documentation/Performance/Conceptual/ManagingMemory/Articles/VMPages.html#//apple_ref/doc/uid/20001985-CJBJFIDD
