# `05.asm` — 64-bit LIFO order with register values

## What happens to the stack

The program loads `RAX = 0x1122334455667788` and `RBX = 0x1234`; neither load changes `RSP`. Let entry `RSP` be **S**. `push rax` reserves 8 bytes, changes `RSP` to **S - 8**, and stores `0x1122334455667788`. `push rbx` reserves another 8 bytes, changes `RSP` to **S - 16**, and stores `0x1234` at the top.

The first `pop rbx` takes the top value (`0x1234`) and changes `RSP` back to **S - 8**. The next `pop rax` takes the older value (`0x1122334455667788`) and restores `RSP` to **S**. Thus both registers recover their original values, showing the last-in, first-out rule with values of different sizes and magnitudes; both stack entries are still 8 bytes because the program is 64-bit.

## End of execution and test observation

Immediately after the pops, `RBX` is `0x1234` and `RAX` is `0x1122334455667788`. The later `mov rax, 60` deliberately overwrites `RAX` with the Linux exit syscall number, after the stack demonstration is complete. The program ran successfully with no output and exit code 0.
