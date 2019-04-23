# Solutions for xv6 homeworks in 6.828

## [syscall]

To add a syscall date, I edited:

- syscall.h - defines `SYS_date` value.
- user.h    - declares `sys_date`.
- usys.S    - defines `date` as the syscall (int 0x80) setting `SYS_date` to eax.
- sysproc.c - implements `sys_date`
  - `sys_date` uses argptr to get the passed struct's address in a safe way.
- syscall.c - associates `SYS_date` syscall number to the `sys_date` function.

To add a shell function date, I edited

- date.c    - implementation of date.
- Makefile  - added `_date` to UPROGS. This target is defined as `_%:` around L142.

## [lazy page allocation]

sbrk is the system call to increase the process' ocuppying memory size.

To allow lazy page allocation, modified `sys_sbrk` to change only the process size without page allocation.
`echo hi` panics with the following message.

`
pid 3 sh: trap 14 err 6 on cpu 0 eip 0x114c addr 0x4004--kill proc
`

Why it happens? Well, 
on executing a command, 
shell `malloc`s a command struct before executing the command.
`malloc` internally calls `sbrk`.

[syscall]: https://pdos.csail.mit.edu/6.828/2017/homework/xv6-syscall.html
[lazy page allocation]: https://pdos.csail.mit.edu/6.828/2017/homework/xv6-zero-fill.html
