# `01.asm` — Basic 64-bit push and pop

## What happens to the stack

Let the value of `RSP` on entry be **S**. The program first loads `RAX = 100` and `RBX = 200`; these `mov` instructions do not use the stack.

`push rax` decreases `RSP` by 8 bytes to **S - 8** and stores the 64-bit value `100` there. `push rbx` decreases `RSP` by another 8 bytes to **S - 16** and stores `200` at the new top of the stack. At this point the most recently pushed value, `200`, is at the top.

`pop rcx` reads `200` from **S - 16** into `RCX` and increases `RSP` to **S - 8**. `pop rdx` then reads `100` into `RDX` and restores `RSP` to **S**. This is LIFO order: the last pushed value is the first value popped.

## End of execution and test observation

The `nop` instructions do not change the stack. The final Linux `exit` system call terminates the program with status 0; it does not need to pop the values because the stack was already restored. The assembled program ran successfully, printed no output, and exited with code 0. To inspect the result in a debugger just before the exit call, `RCX` should contain `200` and `RDX` should contain `100`.
