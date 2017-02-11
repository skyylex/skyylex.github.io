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



### pagestuff

> pagestuff - Mach-O file page analysis tool
>
> -- `man pagestuff

**Original project of the Carnegie Mellon University**

1. http://www.cs.cmu.edu/afs/cs/project/mach/public/www/mach.html - Unfortunately, a lot of the links are broken and some of the documentation is lost.

**References to Mach-O specification:**

1. http://www.cilinder.be/docs/next/NeXTStep/3.3/nd/DevTools/14_MachO/MachO.htmld/index.html
2. https://github.com/aidansteele/osx-abi-macho-file-format-reference

**Apple Developer documentation:**

1. https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html#//apple_ref/doc/uid/TP40001827-SW1
