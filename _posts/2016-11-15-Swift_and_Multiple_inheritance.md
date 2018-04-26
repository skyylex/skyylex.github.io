---
layout: default
title: Swift and Multiple Inheritance
---

<!-- tags: swift, objc, multiple_inheritance -->

**Swift and Multiple Inheritance.**

**Introduction**

I like to find out new edges in the well-known concepts. I think no need to prove that basic OOP concepts, such as encapsulation, inheritance, and polymorphism are well-known. They were discussed millions of times and I should not spend time on repeating this. However, I will put some references at the end to be precise enough for someone who needs some more info on the subject. And the subject today will be Multiple Inheritance in Swift, as the title says. Why? Because I thought that Objective-C has no support of a multiple inheritance. And Swift was extended with the default implementation for protocols which is a type of a multiple inheritance. So it's interesting to play with the old issues in the new environment.


**Notice**: before moving directly to the Swift details I want to make clear that inheritance could be classified according to the inherited subject: *inherited implementation* or *inherited interface*. Now it's clear that Objective-C has also multiple inheritance in terms of conforming to `@protocol`. However, it has no issues (described further) because `@protocol` is just a declaration of available methods for usage.

**How multiple inheritance is presented in Swift**

Swift has such feature as protocols. Mostly it has the same meaning as in Objective-C. The difference was made in Swift 3, a protocol allows to add default implementation. And class(struct) which conforms to such protocol will inherit this implementation. The idea is simply to share same functionality without creating a separate implementation for each of the methods in each inherited class or struct. And from the first look, it might look great. Unfortunately, there is a classic problem where two or more protocols (or protocol and class/struct) has exactly the same method signature. In that case, there is no way for the linker to choose what implementation to use in the inherited class.

**Swift, we have a problem**

The code below represents a problem. There are two protocols A and B with extensions that contain default implementations. These protocols were declared as conformed to class ConflictTarget. And Xcode output clearly displays that such configuration will not work.


```
protocol A {
    func sameFunctionName()
}

extension A {
    func sameFunctionName() {
        print("method from class A")
    }
}

protocol B {
    func sameFunctionName()
}

extension B {
    func sameFunctionName() {
        print("method from class B")
    }
}

class ConflictTarget: A, B {
    
}

```

**Xcode console output**

```
Playground execution failed: error: NamesConflict.playground:20:7: error: type 'ConflictTarget' does not conform to protocol 'A'
class ConflictTarget: A, B {
      ^

NamesConflict.playground:1:10: note: multiple matching functions named 'sameFunctionName()' with type '() -> ()'
    func sameFunctionName()
         ^

NamesConflict.playground:15:10: note: candidate exactly matches
    func sameFunctionName() {
         ^

NamesConflict.playground:5:10: note: candidate exactly matches
    func sameFunctionName() {
         ^

error: NamesConflict.playground:20:7: error: type 'ConflictTarget' does not conform to protocol 'B'
class ConflictTarget: A, B {
      ^

NamesConflict.playground:11:10: note: multiple matching functions named 'sameFunctionName()' with type '() -> ()'
    func sameFunctionName()
         ^

NamesConflict.playground:15:10: note: candidate exactly matches
    func sameFunctionName() {
         ^

NamesConflict.playground:5:10: note: candidate exactly matches
    func sameFunctionName() {
         ^


* thread #1: tid = 0x5f41a5, 0x000000010bf513c0 NamesConflict`executePlayground, queue = 'com.apple.main-thread', stop reason = breakpoint 1.2
  * frame #0: 0x000000010bf513c0 NamesConflict`executePlayground
    frame #1: 0x000000010bf509c0 NamesConflict`__37-[XCPAppDelegate enqueueRunLoopBlock]_block_invoke + 32
    frame #2: 0x000000010ca6b25c CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_A_BLOCK__ + 12
    frame #3: 0x000000010ca50304 CoreFoundation`__CFRunLoopDoBlocks + 356
    frame #4: 0x000000010ca4fa75 CoreFoundation`__CFRunLoopRun + 901
    frame #5: 0x000000010ca4f494 CoreFoundation`CFRunLoopRunSpecific + 420
    frame #6: 0x0000000111e5da6f GraphicsServices`GSEventRunModal + 161
    frame #7: 0x000000010d5fa964 UIKit`UIApplicationMain + 159
    frame #8: 0x000000010bf506e9 NamesConflict`main + 201
    frame #9: 0x000000010ffad68d libdyld.dylib`start + 1
    frame #10: 0x000000010ffad68d libdyld.dylib`start + 1
```

**Conclusion**

1. "Favor object composition over class inheritance". I agree that the case described above is rare, and if the user has control over the code related to protocols declaration and implementation it could be easily fixed. However, if all you have is binary framework there is no much to choose. 

2. The second issue I see is related to the functionality for protocols to inherit from the other protocols. When you use them only as API declaration it's ok. In such case, you just save time to not declare same methods twice. But if the extension with the default implementation might appear somewhere, it brings an additional mess to understand the system behavior during code analysis and debugging.

**Thank you for reading!**

**References**

1. [Multiple inheritance - Wikipedia](https://en.wikipedia.org/wiki/Multiple_inheritance)
2. [Composition over inheritance - Wikipedia](https://en.wikipedia.org/wiki/Composition_over_inheritance)
3. [Protocol-oriented programming - WWDC 2015](https://developer.apple.com/videos/play/wwdc2015/408/)
