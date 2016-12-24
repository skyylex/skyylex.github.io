## Swift ideas before Swift. Protocol default implementation.

### Introduction

Let's take a look at one of the most interesting features from OOD point - protocols.
In non-Swift world protocols are called interfaces and make possible most of the design templates to exist.
Moreover some people say that OOP should operate on the interface level and not on class.

But today I'm not going to discuss old as a world topic of OOD and it's relation to interfaces (may be one day).
Currently I want to point your attention to more specific area of Swift's protocols - protocol default implementation.
To be completely accurate the statement below that Swift protocols aren't the same as interfaces isn't quite true in terms of common sense of OOD. Protocols have default implementation and at that point they are closer to abstract classes. 
They have some default behaviour which could be inherited, however protocol instance couldn't be created. 
So this behaviour might appear only in the structs or classes which conform to the protocol.

Default protocol implementation is a way to re-use common functionality without strong dependency on the single parent class,
because Swift also as Objective-C doesn't support multiple inheritance explicitly. And it's the way Apple engineers provide for you to achieve some of the benefits of multiple inheritance. Most of the questions related to multiple inheritance are answered in the way the real need of this tool don't occure quite often. And in such cases you could use composition and win in a flexibility and simplicity of the result solution.

However the idea that leads me to write this post is that Swift features aren't new (mostly). And many of the things appeared in Swift, were available in the Objective-C frameworks made by enthusiasts. Smart people know what they need. And default protocol implementation also isn't an exception of this rule.

### Objective-C

### Swift we have problems.



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

##### Xcode console output:
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
