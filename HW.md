# Solutions for xv6 homeworks in 6.828

## [syscall]

For part two (adding a syscall), you should edit

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

[syscall]: https://pdos.csail.mit.edu/6.828/2017/homework/xv6-syscall.html
