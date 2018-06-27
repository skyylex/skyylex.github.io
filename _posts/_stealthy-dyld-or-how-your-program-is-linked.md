Stealthy dyld or how your program is linked

In the [previous post about nm](link) I've met again `dyld_stub_binder`. And I thought, "What do I know about dynamic linking?". Mostly it's some usual stuff like that there are static and dynamic linkers, some other details about dynamic nature and a few pros and cons of both approaches. But usually I do not look out on linking at all, unless [when I have to](https://stackoverflow.com/questions/39701879/dyld-library-not-loaded-rpath-libswiftquartzcore-dylib). So, why don't I take this opportunity to check what it can and how it actually works?

Disclaimer: This post is written using [place version of dyld] and standard dyld on macOS Sierra.

Usually I use man as my personal preference. But not this time, this time it's too expressive: "dyld - the dynamic link editor". Undoubtedly, it's accurate,  but I'd vote for more detailed description.

Usually, if you're interested in anything low level on Mac or Objective-C related, try out to search trough Mike Ash blog. 99% you'll find everything that you need. 

**References**
- [Dynamic linker](https://en.wikipedia.org/wiki/Dynamic_linker)
- [Friday Q&A 2012-11-09: dyld: Dynamic Linking On OS X
by Gwynne Raskind] https://www.mikeash.com/pyblog/friday-qa-2012-11-09-dyld-dynamic-linking-on-os-x.html


*** Ideas ***

- man pages as a documentation
- articles about linking / linkers in general
- specifics about mac
- sources
- nocr?
