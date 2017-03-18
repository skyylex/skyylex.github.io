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

We found `_os_lock_lock` and it's located in the library *libSystem*. Each binary contain list of the required dependencies which we can find using `otool`:

```bash
otool -L Foundation | grep "libSystem"
```

> /usr/lib/libSystem.dylib (compatibility version 1.0.0, current version 1238.0.0)

Exactly what we need, then let's try to find this symbol in the discovered dylib using provided path.

```bash
nm /usr/lib/libSystem.dylib -m | grep "os_lock_lock"
``` 

and we get empty result. `libSystem` refers a lot of libraries inside, and most probably provide simplified access for importing core functionality.

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

Using simple enumeration we can find out that `/usr/lib/system/libsystem_platform.dylib` is our next stop.

```bash
nm /usr/lib/system/libsystem_platform.dylib -m | grep "os_lock_lock"
```

> 00000000000022c4 (__TEXT,__text) external _os_lock_lock

That means that `libsystem_platform.dylib` contains `_os_lock_lock` string in the symbol table. So we can try to disassemble it.

```bash
objdump -macho -disassemble libsystem_platform.dylib > libsystem_platform_disasm.out
```

I stored whole output into the file hoping to work closely with the other symbol strings. But the result which I saw was a surprise for me. Output file was about 8k lines and I used search to find `_os_lock_lock`. And found this:

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

> 0000000000009210 (__DATA,__const) external __os_lock_type_handoff

but still these stored values have no meaning for our investigation. 

> 0000000000009210	73 79 00 00 00 00 00 00 9d 25 00 00 00 00 00 00 
> 0000000000009220	09 29 00 00 00 00 00 00 b4 25 00 00 00 00 00 00

Potentially, it could be the end of the post without any result. But there was one mistake I made at the beginnning.


- ; in /usr/lib/libSystem.dylib

/usr/lib/system/libsystem_platform.dylib

__os_lock_handoff_trylock:

0000000000002909         mov        ecx, dword [gs:0x18]
0000000000002911         xor        eax, eax
0000000000002913         lock cmpxchg dword [rdi+8], ecx
0000000000002918         sete       al
000000000000291b         ret
                        ; endp

compare + exchange + lock (atomic operation)

==================================================

```
int __os_lock_handoff_lock_slow(int arg0) {
    r15 = arg0 + 0x8;
    rbx = 0x4;
    r12 = 0x1;
    r13 = 0xffffff9c;
    rax = *(int32_t *)r15;
    if (rax != 0x0) goto loc_2983;

loc_297a:
    r14 = *(int32_t *)0x18;
    rax = 0x0;
    *(int32_t *)r15 = lock intrinsic_cmpxchg(*(int32_t *)r15, r14);
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
}
```

**thread_switch**


in /usr/lib/system/libsystem_kernel.dylib

_syscall_thread_switch:
0000000000012488         mov        r10, rcx                                    ; CODE XREF=_thread_switch
000000000001248b         mov        eax, 0x100003d
0000000000012490         syscall
0000000000012492         ret

61
thread_switch
osfmk/kern/syscall_subr.c
Force context switch of current thread. Allows for handoff (specifying the next thread hint)

0x100003d - seems like list of syscals is defined from 0x1000000
Anyway, "3D" in hex is equal 61 

thread_switch usage

**References:**

- https://skyylex.github.io/NSObject_internals_-retain_and_release
- http://x86.renejeschke.de/html/file_module_x86_id_41.html
- https://opensource.apple.com/source/xnu/xnu-1504.9.26/osfmk/kern/syscall_subr.c
- http://web.mit.edu/darwin/src/modules/xnu/osfmk/man/thread_switch.html
- https://lists.swift.org/pipermail/swift-dev/Week-of-Mon-20151214/000389.html
- https://www3.nd.edu/~dthain/courses/cse40243/fall2015/intel-intro.html
- https://github.com/hjl-tools/x86-psABI/wiki/X86-psABI
- http://csiflabs.cs.ucdavis.edu/~ssdavis/50/att-syntax.htm
