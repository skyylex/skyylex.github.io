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
because Swift also as Objective-C doesn't support multiple inheritance explicitly. And it's the way Apple engineers provide for you to achieve some of the benefits of multiple inheritance. [It could be interesting to traverse Swiftimplementation to introspect difficulties of the default implementation. However not sure that could be able to complete this task :).]
Most of the questions related to multiple inheritance are answered in the way the real need of this tool don't occure quite often.
And in such cases you could use composition and win in a flexibility and simplicity of the result solution.

However the idea that leads me to write this post is that Swift features aren't new (mostly). And many of the things appeared in Swift, were available in the Objective-C frameworks made by enthusiasts. Smart people know what they need. And default protocol implementation also isn't an exception of this rule.
