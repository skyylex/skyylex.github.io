---
layout: default
title: Macros in C / Objective-C 
description: Macro is a quite different programming technique. Why do we need to use it nowadays? And what types of tasks could be solved using macros? And a few real-world examples. 
tags: objective-c c-preprocessor macros 
---

**Macros in C / Objective-C**

> Even for C, the trick is suspicious: it uses the preprocessor to define new syntax. C programmers have learned through bitter experience that inventing new syntax confuses maintenance programmers, and therefore causes bugs. I think, however, that the benefits in this case outweigh the potential problems, since the amount of repetitive code saved is so great.
> 
> Lars Wirzenius

**C Preprocessor**

C preprocessor is a crucial part of any compilation target (assuming that target is C or C-based language) larger than one file. It's used most commonly via `#include` / `#import` constructions, to reuse already implemented function or class from a different file. Sometimes, there is a code that should be isolated and we want to limit its scope to a specific target, conditional inclusion is helpful here via `#if`, `#ifdef` and so on. Comments are the other point that helps to make code clearer, but comments aren't needed in the binary and the preprocessor silently removes them for us. We also can use `#define` for the preprocessor to define a constant. But it's more rarely used that way nowadays. In Objective-C such style became obsolete and was replaced with a specifying `static` variables. However, preprocessor could also do very powerful things from completely different perspective using macros. And recently I've found some interesting ideas in the libextc framework, and I'd like to share them with you.


**MACROS**

*NOTE:* This article doesn't try to compete with the documentation about C Preproccessor, so if you're not familiar with the subject it worths to check something like [GCC manual](https://gcc.gnu.org/onlinedocs/cpp/index.html)

A macro is a rule that defines how specific pattern is replaced with a specified construction. There are several types of macros function-like macros, with or without parameters. Nevertheless, they all have in common the following structure:

```
#define expression_to_replace replacement
```

`#define` define is the key word (generally all starting with `#` expressions are preprocessor-related) that starts the macro definition, as I already mentioned, you can define the constant that way or you can try to build more advanced expressions hidden under the short expression 

As they say, better once to see, than hundred times to hear. So for example, the macro:

```
#define AppDelegate [[UIApplication sharedApplication] delegate] 
```

was very popular some time ago in iOS project on Objective-C and actually [it's not very good style](https://www.cocoawithlove.com/2008/11/singletons-appdelegates-and-top-level.html). What this snippet does is whenever the preprocessor finds `AppDelegate` used in the code, `AppDelegate` would be replaced with the right part `[[UIApplication sharedApplication] delegate]`. 

`[AppDelegate window]` => `[[[UIApplication sharedApplication] delegate] window]`

Usually I think about macros as an advanced *find & replace* tool. Macros could be used for different purposes like reusing code, code generation and others. The most painful macros characteristic is that you can not debug code inside macro, because macro is a rule and actual code needs to be produced by expanding macros using preprocessor. However, it's interesting technique that helps you to think in a different terms. So, for this article I've chosen to play with `metamacros.h` from `libextc`. The metamacros from this file are used in a several other open-source projects, so we'll have some real examples why these macros were used and not only some fancy theories. All for real.

**You just don't know C99 well enough, boy**

Let's try with this one:

```
#define metamacro_exprify(...) \
((__VA_ARGS__), true)
```

What it does:

1. Places all passed arguments inside the first pair of round brackets.
2. Add `true` in the end of a wrapped expression. That surprisingly is the same as to provide `true` result of expression or `return true` in anonymous function in the other language. I suspect I don't know enough about C99 standard. So you basically can do the following:

```
metamacro_exprify({});
```

And preprocessor will produce the following:

```
(({}), 1);
```

And it's completely valid expression (compiler may produce warning to insert `(void)` cast before `{}`, but other than that there is no error), so for example: `printf("%d", ((({})), 1));` will have the following output: 1.

From the [C99 draft](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf):

> The left operand of a comma operator is evaluated as a void expression; there is a sequence point after its evaluation. Then the right operand is evaluated; the result has its type and value.97) If an attempt is made to modify the result of a comma operator or to access it after the next sequence point, the behavior is undefined.

**SSSSTRINGIFY!**

```
#define metamacro_stringify(VALUE) # VALUE
```

C preprocessor treats `#` as a special symbol. The value right to it will be stringinized. Example:

**Note:** I've retrieved an example from the concrete protocol implementation provided by `libextobjc`. To those who are new to Objective-C, protocol is a name Apple uses to name interfaces. Objective-C supports only abstract protocols out-of-the-box, which are interfaces that only declare some methods, but do not provide their implementation. Concrete protocol is an interface that provides not only methods declaration, but also implementation. Swift supports conrete protocols from the 2nd version. This technique is one of the ways to get a multiple inheritance behaviour. However, as we all know all such hierarchies have some limitations [link]({{ site.baseurl }}{% link _posts/2016-11-15-Swift_and_Multiple_inheritance.md %}).
 
Back to the [macro](https://github.com/jspahrsummers/libextobjc/blob/master/extobjc/EXTConcreteProtocol.h):

```
#define concreteprotocol(NAME) \
    /*
     * create a class used to contain all the methods used in this protocol
     */ \
    @interface NAME ## _ProtocolMethodContainer : NSObject < NAME > {} \
    @end \
    \
    @implementation NAME ## _ProtocolMethodContainer \
    /*
     * when this class is loaded into the runtime, add the concrete protocol
     * into the list we have of them
     */ \
    + (void)load { \
        /*
         * passes the actual protocol as the first parameter, then this class as
         * the second
         */ \
        if (!ext_addConcreteProtocol(objc_getProtocol(metamacro_stringify(NAME)), self)) \
            fprintf(stderr, "ERROR: Could not load concrete protocol %s\n", metamacro_stringify(NAME)); \
    } \
	
	// Something else that was removed as insignificant
	
	@end
```

Basically, here we have a macro that produces class (in Objective-C inteface is a class) body for the provided protocol name. The line `objc_getProtocol(metamacro_stringify(NAME)` is our target. As you can see `NAME` is provided as a parameter and used in the declaration of the interface and as a part of the same class name `@implementation`. And it is also passed to the `objc_getProtocol()` as a parameter, this function expects `const char *name`, so stringinizing will wrap the expanded `NAME` value in a double quotes. If don't do that, the `NAME` value will placed as it is and compiler will use it as an expression (variable or will try to calculate required value).


- https://developer.apple.com/documentation/objectivec/1418870-objc_getprotocol?language=objc
- https://gcc.gnu.org/onlinedocs/cpp/Stringizing.html#Stringizing

**Just concat it**

```
#define metamacro_concat_(A, B) A ## B
```

You already know it, right? In the example below there were 2 usages of the `##`. This symbol concatenates left provided argument with right one. So if for previous concrete protocol macro we would use *SuperProtocol* as a `NAME`, the part of the produced code for `@interface NAME ## _ProtocolMethodContainer : NSObject < NAME >` will be presented after preprocessing as: 

```
@interface SuperProtocol_ProtocolMethodContainer : NSObject < SuperProtocol >
```

Or using provided macro `metamacro_concat_(123, 456)`, will produce 123456.

**Subscript, but not an array**

                                                                              
Here is	the macro:

```
#define metamacro_at(N, ...) \                                                                                         
        metamacro_concat(metamacro_at, N)(__VA_ARGS__)
```

The description says that N should be numerical from 0 to 20, and the rest of arguments will be used as a source of subscripting. For example: `metamacro_at(2, a, b, c)`,  will produce => c. How does it work? Here we see another interesting thing - macro could be reused inside another macro. And this is great way to build something advanced. Back to the example, as we know `metamacro_concat` connects 2 provided arguments, and as a result macro will be => 

```
metamacro_atN(__VA_ARGS__)
```

Aha, looks like specific macro for the specific N

```
// metamacro_at expansions
#define metamacro_at0(...) metamacro_head(__VA_ARGS__)
#define metamacro_at1(_0, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at2(_0, _1, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at3(_0, _1, _2, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at4(_0, _1, _2, _3, ...) metamacro_head(__VA_ARGS__)
#define metamacro_at5(_0, _1, _2, _3, _4, ...) metamacro_head(__VA_ARGS__)
...
#define metamacro_at20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...) metamacro_head(__VA_ARGS__)
```

Consequently:

```
#define metamacro_head(...) \
        metamacro_head_(__VA_ARGS__, 0)
```

```
#define metamacro_head_(FIRST, ...) FIRST
```

So, basically the whole logic is based on the macro that receives the first argument and pre-generated sequence of arguments. Pre-generated sequence shift the element initially to the right(each pre-generated element cuts related element from `__VAR_ARGS__`), so the N-1 items will be cut and the Nth will be the first from left to right.

Example?

**Accounting**

```
#define metamacro_argcount(...) \
        metamacro_at(20, __VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)
```

Reasonable would be to be able to count all provided arguments. The logic the same pregenerated sequence provides the actual count and the `__VAR_ARGS__` shifts the subscript to the appropriate number. Let's take an example:

`metamacro_argcount(a, b, c, d)` => 

`metamacro_at(20, a, b, c, d, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1)`

So, we are getting 20th element from: 

*a, b, c, d, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, **4**, 3, 2, 1*

Which is actually the arguments count and it equals 4. The whole logic is build on a predefined numbers and limited shift. Beautiful, isn't it? 

**Summary:**

Macro is yet another tool for embodying your ideas in the source code. Preprocessor was there for a long time, helping with a basic stuff, however this tool could be used for much more advanced ideas like a code generation. metamacros.h header hints about that fact. I think that concrete protocol implementation is an interesting example of the real-world usage. Such challenge otherwise requires some interaction with the Objective-C runtime, and it's arguable which one is better. I don't think macros are good or bad, the price of the generated code without ability to debug is quite high, but sometimes they are quite good and are less evil comparing to the alternatives. Also I want to mention [quite interesting example, provided in 2000 by Lars Wirzenius](http://liw.iki.fi\
/liw/texts/cpp-trick.html), where data types are redefined based on the context, but used under general name provided by macro. I think currently we use generics / templates for such things, but still the solution opens your mind to something new and add one more idea about how you can do something. Probably, that is what important, to extend your mind and maybe next time the problem you have could be interpreted differently using new experience for completely unrelated tool and language. Who knows?  

**P.S.** There are a lot of other things that worth you attention in the `libextc` / `libextobjc`. Check them out! 

**Thank you for reading!**

**References:**

1. [Github repository of `libextc`](https://github.com/jspahrsummers/libextc) 
2. [C Preprocessor documentation (by GNU)](https://gcc.gnu.org/onlinedocs/cpp/index.html)
3. [Reference for objc_getprotocol()](https://developer.apple.com/documentation/objectivec/1418870-objc_getprotocol?language=objc)
4. [Stringinizing](https://gcc.gnu.org/onlinedocs/cpp/Stringizing.html#Stringizing)
5. [C Preprocessor Trick For Implementing Similar Data Types](http://liw.iki.fi/liw/texts/cpp-trick.html)
