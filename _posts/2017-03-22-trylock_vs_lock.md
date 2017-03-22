---
layout: default
title: tryLock vs lock
---
### **tryLock vs lock**

**Introduction**

In previous post about retain and release I have found that synchronized access to the reference counters is built over the lock which uses `tryLock` function first. And if `tryLock` doesn't lock then "slow" path is used with `lock` function. However, I didn't have any concrete information about why `lock` is slower than `tryLock`. So in this post I will try to find and explain why.

**Analysis**

Our start point is defined like this:

- `spinlock_t slock;`

The declaration of the `spinlock_t` is the following:

```c++
class spinlock_t {
    os_lock_handoff_s mLock;
 public:
    spinlock_t() : mLock(OS_LOCK_HANDOFF_INIT) { }
    
    void lock() { os_lock_lock(&mLock); }
    void unlock() { os_lock_unlock(&mLock); }
    bool trylock() { return os_lock_trylock(&mLock); }
    
    // ...
}
```

So the next target of the journey is `os_lock_handoff_s`, `os_lock_lock` and `os_lock_unlock`. What we can say about `os_lock_handoff_s`? Quick search in the Internet says that this type is defined in the system private header `lock_private.h`. This is most probably true, because that header is included in the `NSOject.m`:

```c++
#   include <os/lock_private.h>
```

Private header means that described functionality isn't supposed to be used by developers, also that these types aren't covered in the documentation from the official sources. Ok, if we cannot get sources and more or less oficial documentation let's take a look at reverse engineering tools. I'll start from the most obvious way. We know that `retain` and `release` is a part of the `Objective-C` language, more explicitly it's part of the `NSObject` implementation. And `NSObject` is one of the root classes in the `Foundation` framework. Each platform has it's own build of `Foundation`:

- /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/System/Library/Frameworks/Foundation.framework 
- /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk/System/Library/Frameworks/Foundation.framework

**Notice:** current post is written entirely based on Xcode Version 8.2.1 (8C1002) and macOS Sierra.

I will use binary for iPhoneSimulator platform, and `nm` for initial symbol search.

`nm Foundation -m | grep "os_lock_lock"`, - where Foundation is a binary which is located at Foundation.framework/Foundation. The output will be the following:

> (undefined) external _os_lock_lock (from libSystem)

We found `_os_lock_lock` and it's located in the library *libSystem*. Each binary contains list of the required dependencies which we can find using `otool`:

```bash
otool -L Foundation | grep "libSystem"
```

> /usr/lib/libSystem.dylib (compatibility version 1.0.0, current version 1238.0.0)

Exactly what we need, then let's try to find this symbol in the discovered dylib using provided path.

```bash
nm /usr/lib/libSystem.dylib -m | grep "os_lock_lock"
``` 

and we get empty result. `libSystem` refers to the multiple libraries inside, and most probably is some kind of a global entry point for system functionality.

```bash
otool -L /usr/lib/libSystem.dylib
```

> /usr/lib/libSystem.dylib:
>	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1238.0.0) <br>
>	/usr/lib/system/libcache.dylib (compatibility version 1.0.0, current version 79.0.0) <br>
>	/usr/lib/system/libcommonCrypto.dylib (compatibility version 1.0.0, current version 60092.30.2)
>	/usr/lib/system/libcompiler_rt.dylib (compatibility version 1.0.0, current version 62.0.0) <br>
>	/usr/lib/system/libcopyfile.dylib (compatibility version 1.0.0, current version 1.0.0) <br>
>	/usr/lib/system/libcorecrypto.dylib (compatibility version 1.0.0, current version 442.30.20) <br> 
>	/usr/lib/system/libdispatch.dylib (compatibility version 1.0.0, current version 703.30.5) <br>
>	/usr/lib/system/libdyld.dylib (compatibility version 1.0.0, current version 421.2.0) <br>
>	/usr/lib/system/libkeymgr.dylib (compatibility version 1.0.0, current version 28.0.0) <br>
>	/usr/lib/system/liblaunch.dylib (compatibility version 1.0.0, current version 972.30.7) <br>
>	/usr/lib/system/libmacho.dylib (compatibility version 1.0.0, current version 894.0.0) <br>
>	/usr/lib/system/libquarantine.dylib (compatibility version 1.0.0, current version 85.0.0) <br>
>	/usr/lib/system/libremovefile.dylib (compatibility version 1.0.0, current version 45.0.0) <br>
>	/usr/lib/system/libsystem_asl.dylib (compatibility version 1.0.0, current version 349.30.2) <br>
>	/usr/lib/system/libsystem_blocks.dylib (compatibility version 1.0.0, current version 67.0.0) <br>
>	/usr/lib/system/libsystem_c.dylib (compatibility version 1.0.0, current version 1158.30.7) <br>
>	/usr/lib/system/libsystem_configuration.dylib (compatibility version 1.0.0, current version 888.30.2) <br>
>	/usr/lib/system/libsystem_coreservices.dylib (compatibility version 1.0.0, current version 41.4.0) <br>
>	/usr/lib/system/libsystem_coretls.dylib (compatibility version 1.0.0, current version 121.31.1) <br>
>	/usr/lib/system/libsystem_dnssd.dylib (compatibility version 1.0.0, current version 765.30.11) <br>
>	/usr/lib/system/libsystem_info.dylib (compatibility version 1.0.0, current version 503.30.1) <br>
>	/usr/lib/system/libsystem_kernel.dylib (compatibility version 1.0.0, current version 3789.41.3) <br>
>	/usr/lib/system/libsystem_m.dylib (compatibility version 1.0.0, current version 3121.4.0) <br>
>	/usr/lib/system/libsystem_malloc.dylib (compatibility version 1.0.0, current version 116.30.3) <br>
>	/usr/lib/system/libsystem_network.dylib (compatibility version 1.0.0, current version 1.0.0) <br>
>	/usr/lib/system/libsystem_networkextension.dylib (compatibility version 1.0.0, current version 1.0.0) <br>
>	/usr/lib/system/libsystem_notify.dylib (compatibility version 1.0.0, current version 165.20.1) <br>
>	/usr/lib/system/libsystem_platform.dylib (compatibility version 1.0.0, current version 126.1.2) <br>
>	/usr/lib/system/libsystem_pthread.dylib (compatibility version 1.0.0, current version 218.30.1) <br>
>	/usr/lib/system/libsystem_sandbox.dylib (compatibility version 1.0.0, current version 592.31.1) <br>
>	/usr/lib/system/libsystem_secinit.dylib (compatibility version 1.0.0, current version 24.0.0) <br>
>	/usr/lib/system/libsystem_symptoms.dylib (compatibility version 1.0.0, current version 1.0.0) <br>
>   /usr/lib/system/libsystem_trace.dylib (compatibility version 1.0.0, current version 518.30.7) <br>
>   /usr/lib/system/libunwind.dylib (compatibility version 1.0.0, current version 35.3.0) <br>
>   /usr/lib/system/libxpc.dylib (compatibility version 1.0.0, current version 972.30.7) <br>

Via simple enumeration (check and search symbol table for each .dylib) we can find out that `/usr/lib/system/libsystem_platform.dylib` is our next stop.

```bash
nm /usr/lib/system/libsystem_platform.dylib -m | grep "os_lock_lock"
```

> 00000000000022c4 (__TEXT,__text) external _os_lock_lock

That means that `libsystem_platform.dylib` contains `_os_lock_lock` string in the symbol table. So we can try to disassemble it.

```bash
objdump -macho -disassemble /usr/lib/system//libsystem_platform.dylib > libsystem_platform_disasm.out
```

I stored whole output into the file, hoping to work closely with the other symbol strings. But the result which I saw was a surprise for me. Output file was about 8k lines and I used search to find `_os_lock_lock`. And found this:

```asm
_os_lock_lock:
    22c4:	48 8b 07 	movq	(%rdi), %rax
    22c7:	ff 60 08 	jmpq	*8(%rax)
```

Yes, it is only 2 line of asm code and it definitely isn't a full implementation of lock. What can we get from this code? The first thing I should check is what syntax type is used in the code. Nowadays we have AT&T and Intel syntaxes and to determine syntax type, I should look at the presence of "%" prefix on registers, if registers have it - you have AT&T syntax. Why it's important? Because each syntax type has it's own order of the source and destination register. In our case it's AT&T and this type use initial operand as source.

This following line could be read as move quad word from address stored in `%rdi` and save it to `%rax`

```asm
movq (%rdi), %rax
```

Using ABI for x86_64 we could find that %rdi is used to pass 1st argument to function. Recalling usage of the lock function in the Objective-C source code confirms that point:

`os_lock_lock(&mLock);`

so `%rdi` contains actual allocated lock data, which is refered by `os_lock_handoff_s` pointer.

The next line describes that current execution should transfered to another point via jump to specific address calculated in expression `*8(%rax)`. There are two parts in the expression, first part it's a calculated value using so-called `base-relative` address mode, so `8(%rax)` could be read as value at the memory location eight bytes above the address indicated by `%rax`. We know that `rax` contains actual lock data. Combining these two facts together, we can assume that lock is a kind of struct which contains lock function. Unfortunately, that's all we have to say. Search for `os_lock_handoff_s` doesn't produce any result. The only more or less similar type which I've found was:

```bash
nm -m /usr/lib/system//libsystem_platform.dylib | grep "handoff"
```

> 0000000000009210 (__DATA,__const) external __os_lock_type_handoff

but still these stored values have no meaning for our investigation, because we don't know how to decode them. 

```bash
otool -dV -s __DATA __const /usr/lib/system//libsystem_platform.dylib
```

> 0000000000009210	73 79 00 00 00 00 00 00 9d 25 00 00 00 00 00 00 
> 0000000000009220	09 29 00 00 00 00 00 00 b4 25 00 00 00 00 00 00

So, currently, we have no explicit calls of the lock function. But, for example, we can assume that such lock function should be somewhere in this library or other loaded libraries.

I've started with "lock" searching and got 73 result, so probably it isn't efficient way to guess. After that I found that "lock" word is used not only as a main part of a function name, but also as a prefix. So probably, `tryLock` may be better:

``` 
nm -m /usr/lib/system//libsystem_platform.dylib | grep "trylock"
```

> 0000000000005a2f (__TEXT,__text) non-external (was a private external) __os_lock_eliding_trylock <br>
> 0000000000002909 (__TEXT,__text) non-external (was a private external) __os_lock_handoff_trylock <br>
> 0000000000005924 (__TEXT,__text) non-external (was a private external) __os_lock_nospin_trylock <br>
> 0000000000005388 (__TEXT,__text) non-external (was a private external) __os_lock_spin_trylock <br>
> 0000000000005dee (__TEXT,__text) non-external (was a private external) __os_lock_transactional_trylock <br>
> 0000000000005aed (__TEXT,__text) non-external (was a private external) __os_lock_transactional_trylock$VARIANT$rtm <br>
> 0000000000005aea (__TEXT,__text) non-external __os_lock_transactional_trylock_abort <br>
> 000000000000576f (__TEXT,__text) non-external (was a private external) __os_lock_unfair_trylock <br>
> 000000000000586d (__TEXT,__text) external __os_nospin_lock_trylock <br>
> 0000000000002903 (__TEXT,__text) external _os_lock_trylock <br>
> 00000000000054b2 (__TEXT,__text) external _os_unfair_lock_trylock <br>

Here we see multiple versions of the try lock and the most compatible by name is `__os_lock_handoff_trylock`.

```asm
__os_lock_handoff_trylock:
    2909:   movl %gs:24, %ecx 
    2911:	xorl %eax, %eax
    2913:	lock
    2914:	cmpxchgl %ecx, 8(%rdi)
    2918:	sete %al
    291b:	retq
```
Key operation in the code above is `lock cmpxchgl`, this instruction compares operands and loads one of the operand in destination or in the %eax register. `lock` attribute provides atomic behaviour.

Almost the same implementation in the `__os_lock_handoff_lock`. The key difference of this code is in the last few lines. If `cmpxchgl` isn't able to load into `%rdi`, conditional jump to 0x25af will be performed (`jne 0x25af`). 

```asm
__os_lock_handoff_lock:
000000000000259d	movl	%gs:0x18, %ecx
00000000000025a5	xorl	%eax, %eax
00000000000025a7	lock
00000000000025a8	cmpxchgl	%ecx, 0x8(%rdi)
00000000000025ac	jne	0x25af
00000000000025ae	retq
00000000000025af	jmp	__os_lock_handoff_lock_slow
```

So we move to the `__os_lock_handoff_lock_slow` label.

```asm
__os_lock_handoff_lock_slow:
000000000000291c	pushq	%rbp
000000000000291d	movq	%rsp, %rbp
0000000000002920	pushq	%r15
0000000000002922	pushq	%r14
0000000000002924	pushq	%r13
0000000000002926	pushq	%r12
0000000000002928	pushq	%rbx
0000000000002929	pushq	%rax
000000000000292a	movq	%rdi, %r15
000000000000292d	movl	%gs:0x18, %r14d
0000000000002936	addq	$0x8, %r15
000000000000293a	movl	$0x4, %ebx
000000000000293f	movl	$0x1, %r12d
0000000000002945	movl	$0xffffff9c, %r13d
000000000000294b	jmp	0x2973
000000000000294d	testl	%r13d, %r13d
0000000000002950	movl	$0x5, %ecx
0000000000002955	cmovel	%ecx, %ebx
0000000000002958	movl	%eax, %edi
000000000000295a	movl	%ebx, %esi
000000000000295c	movl	%r12d, %edx
000000000000295f	callq	0x741a ## symbol stub for: _thread_switch
0000000000002964	cmpl	$0x5, %ebx
0000000000002967	sete	%al
000000000000296a	movzbl	%al, %eax
000000000000296d	addl	%eax, %r12d
0000000000002970	incl	%r13d
0000000000002973	movl	(%r15), %eax
0000000000002976	testl	%eax, %eax
0000000000002978	jne	0x2983
000000000000297a	xorl	%eax, %eax
000000000000297c	lock
000000000000297d	cmpxchgl	%r14d, (%r15)
0000000000002981	je	0x2990
0000000000002983	cmpl	%r14d, %eax
0000000000002986	jne	0x294d
0000000000002988	movl	%r14d, %edi
000000000000298b	callq	__os_lock_recursive_abort
0000000000002990	addq	$0x8, %rsp
0000000000002994	popq	%rbx
0000000000002995	popq	%r12
0000000000002997	popq	%r13
0000000000002999	popq	%r14
000000000000299b	popq	%r15
000000000000299d	popq	%rbp
000000000000299e	retq
000000000000299f	nop
```

As you see the disassembled code for slow path is quite difficult for reading. I'll use Hopper to disassemble again and convert to pseudo code.

```c
int __os_lock_handoff_lock_slow(int arg0) {
    r15 = arg0 + 0x8;
    rbx = 0x4;
    r12 = 0x1;
    r13 = 0xffffff9c;
    goto loc_2973;

loc_2973:
    rax = *(int32_t *)r15;
    if (rax != 0x0) goto loc_2983;

loc_297a:
    rax = 0x0;
    *(int32_t *)r15 = lock intrinsic_cmpxchg(*(int32_t *)r15, *(int32_t *)0x18);
    if (*(int32_t *)r15 == 0x0) goto .l3;

loc_2983:
    if (rax != *(int32_t *)0x18) goto loc_294d;

loc_2988:
    rax = __os_lock_recursive_abort(*(int32_t *)0x18);
    return rax;

.l3:
    return rax;

loc_294d:
    if (r13 == 0x0) {
            rbx = 0x5;
    }
    thread_switch(rax, rbx, r12);
    r12 = r12 + ((rbx == 0x5 ? 0x1 : 0x0) & 0xff);
    r13 = r13 + 0x1;
    goto loc_2973;
}
```

One of the interested for us flows is the next: `loc_2973` -> `loc_2983` -> `loc_294d` -> `thread_switch -> loc_2973`. Other flows defined either comparisons via cmpxchg or exit from this code. The main interest for us is `thread_switch` procedure which I'm going to look at.

**thread_switch**

This symbol string is defined in the /usr/lib/system/libsystem_kernel.dylib:

```asm
_syscall_thread_switch:
0000000000012488         mov        r10, rcx                                    ; CODE XREF=_thread_switch
000000000001248b         mov        eax, 0x100003d
0000000000012490         syscall
0000000000012492         ret
```

It's clear that `thread_switch` is peformed via `syscall` and the most importand here is address 0x100003d. `thread_switch`
function is declared in the XNU (hybrid core of the OS X). Usually syscalls are mapped in some kind of a table, where each system call has it's own address. I assumed the same approach and converted `3d` into decimal number `61`. And after some search of the Internet found this table:

> List of mach traps in xnu-792.6.22 <br/>
> <br/>
> /* 26 */    mach_reply_port <br/>
/* 27 */    thread_self_trap <br/>
> ... <br/>
> /* 61 */    thread_switch <br/>
> from http://radare.sourcearchive.com/documentation/1.4/osx-xnu-syscall_8h-source.html

One more source of information could be XNU man pages

> The thread_switch function provides low-level access to the scheduler's context switching code. new_thread is a hint that implements hand-off scheduling. The operating system will attempt to switch directly to the new thread (bypassing the normal logic that selects the next thread to run) if possible. Since this is a hint, it may be incorrect; it is ignored if it doesn't specify a thread on the same host as the current thread or if the scheduler cannot switch to that thread (i.e., not runable or already running on another processor). In this case, the normal logic to select the next thread to run is used; the current thread may continue running if there is no other appropriate thread to run.

So here we are. Hand-off implementation of the lock is constantly trying to switch thread inside a loop while there is no way to perform lock. Switching threads isn't trivial task, so here it's implemented via syscalls and by the definition it can not be fast comparing to basic instructions. So that's why in general case it's better to try lock once again before. Off cource this proof isn't too much accurate, but it seems that it fits into existing picture.

**Summary** 

Based on some explicit and implicit methods and approaches we've found differences between `tryLock` and `lock` method, which make reasonable to use `tryLock` first. However, it's rare technique and I tends to think that `lock` implementation could be improved or it's not so useful nowadays. Actually, it seems that one of these guesses are true. I've checked the most recent source code of objc-706 and at the moment you will not find tryLock in the `retain` implementation, simple lock-unlock pair is  was used there. However, `__os_lock_handoff_` isn't used anymore. Currently it's replaced by `os_unfair_lock` new lock available since macOS 10.12.

**References:**

- http://x86.renejeschke.de/html/file_module_x86_id_41.html
- https://opensource.apple.com/source/xnu/xnu-1504.9.26/osfmk/kern/syscall_subr.c
- http://web.mit.edu/darwin/src/modules/xnu/osfmk/man/thread_switch.html
- https://lists.swift.org/pipermail/swift-dev/Week-of-Mon-20151214/000389.html
- https://www3.nd.edu/~dthain/courses/cse40243/fall2015/intel-intro.html
- https://github.com/hjl-tools/x86-psABI/wiki/X86-psABI
- http://csiflabs.cs.ucdavis.edu/~ssdavis/50/att-syntax.htm
