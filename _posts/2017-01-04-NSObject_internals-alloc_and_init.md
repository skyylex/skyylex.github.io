---
layout: default
---

**NSObject Internals - Episode 1. Alloc & Init.**

Introduction

Today I want to start a series of posts related to investigation of the available NSObject's source code. NSObject is one of the root classes in Objective-C and most of the core concepts of Objective-C are located there. The format of investigation will evolve, at this moment I think that the most useful description should be a combination of the info from source code and official documentation.

Memory in C

Objective-C memory management is built over the C foundation. To start speaking we assume that we are speaking about user mode (kernel mode is a different story) C programmer has access to a stack and heap memory. Heap memory is a memory allocated using malloc/calloc functions. 