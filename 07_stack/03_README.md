# `03.asm` — Call and return using the stack

## What happens to the stack

Let entry `RSP` be **S**. `_start` loads `RAX = 10`, then `call calculate` pushes the address of `after_call` onto the stack. `RSP` becomes **S - 8**, and that one 8-byte item is the only stack value used by this program.

`calculate` does not create a frame or push any data. It adds 5 to `RAX`, so `RAX` becomes `15`. `ret` pops the saved return address into the instruction pointer, transfers execution to `after_call`, and restores `RSP` to **S**.

## End of execution and test observation

At `after_call`, the comment is correct: `RAX` is `15` until `_start` overwrites it with the exit syscall number. The final `exit(0)` does not alter the program's own push/pop balance. The assembled program ran with no output and exit code 0. This example demonstrates that `call` and `ret` use the stack even though no explicit `push` or `pop` appears in the source.
