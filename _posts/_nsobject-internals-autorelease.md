**NSObject Internals - `autorelease` and `@autoreleasePool`**

*I would like to thank my friend Mike Litvinetz, who helped me with the investigation of AutoreleasePool source code*

**Introduction**

This is 3rd episode of the NSObject internals and today I will talk about autorelease and autorelease pool implementation in the Objective-C.
What is it all about? Objective-C memory management is built upon reference counting (RC) technique. RC uses counter to manage object lifetime.
Counter could be increased and decreased by some specified operations. In Objective-C it's done by sending `retain` and `release` messages to object correspondingly. 
Basically, it should be enough to create programs, there is an explicit way to control lifetime and free memory. 
However, there are situations when such explicity is inconvinient in terms of logic. The most known example is returning allocated object from function (method). 
Apple documentation provides ownership as an example of the model that could be used for memory management. If we apply such model, function which create object should also
be responsible for balancing object counter with appropriate `release` in the end of object life time. 
However, it's not easy to do without putting additional mess. If function uses `release` before returning value, memory will be freed earlier than needed.
One of the possible solutions here is to provide delayed way to `release` objects. That's exactly what `autorelease` do.

**Checking documentation**



**NSAutoreleasePool**

Autorelease mechanism is available for developers since iOS 2.0 and macOS 10.0. Initially, it was incapsulated in the NSAutoreleasePool class, which represents a collector for autoreleased objects.
Mike Ash has great article with explanation of the possible implementation for NSAutoreleasePool. I suggest to take a look at it, it definitely worth your time.

References

- [1] Reference counting on Wikipedia - https://en.wikipedia.org/wiki/Reference_counting
- [2] NSAutoreleasePool class reference - https://developer.apple.com/reference/foundation/nsautoreleasepool
- [3] Using Autorelease Pool Blocks - https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmAutoreleasePools.html
- [4] Mike Ash, Let's build an NSAutoreleasePool - https://www.mikeash.com/pyblog/friday-qa-2011-09-02-lets-build-nsautoreleasepool.html
