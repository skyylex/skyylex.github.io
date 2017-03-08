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

http://web.mit.edu/darwin/src/modules/xnu/osfmk/man/thread_switch.html

0x100003d - seems like list of syscals is defined from 0x1000000
Anyway, "3D" in hex is equal 61 

thread_switch usage

https://opensource.apple.com/source/xnu/xnu-1504.9.26/osfmk/kern/syscall_subr.c
