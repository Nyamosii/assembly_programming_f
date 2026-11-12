# `02.asm` — 64-bit function stack frame and local variables

## What happens to the stack

Let entry `RSP` be **S**. `call work` pushes the return address (the instruction after the call) and changes `RSP` to **S - 8** before transferring control to `work`.

The function prologue builds a frame. `push rbp` saves the caller's frame pointer at **S - 16**; `mov rbp, rsp` records that address as the base of this frame. `sub rsp, 16` reserves 16 bytes for two 8-byte local variables, so `RSP` becomes **S - 32**. The program writes `100` to `[RBP - 8]` and `200` to `[RBP - 16]`. These stores use already-reserved stack slots and therefore do not change `RSP`.

`mov rax, [rbp - 8]` and `mov rbx, [rbp - 16]` retrieve the local values, giving `RAX = 100` and `RBX = 200` while the frame is active. The epilogue reverses the setup: `mov rsp, rbp` discards both local slots and returns `RSP` to **S - 16**; `pop rbp` restores the saved frame pointer and moves `RSP` to **S - 8**; `ret` pops the return address and restores `RSP` to **S**.

## End of execution and test observation

Back in `_start`, the program makes the 64-bit `exit(0)` system call. The program ran successfully with no standard output and exit code 0. The useful observation is that the function's local values are gone after the epilogue: their stack space is released before the return address is popped.
