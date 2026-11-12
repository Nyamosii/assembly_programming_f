# `06.asm` — 32-bit LIFO order with register values

## What happens to the stack

This 32-bit example uses 4-byte stack entries. It sets `EAX = 0x11223344` and `EBX = 0x1234` without changing `ESP`. Let entry `ESP` be **S**. `push eax` stores `0x11223344` at **S - 4** and changes `ESP` to **S - 4**. `push ebx` stores `0x1234` at **S - 8** and changes `ESP` to **S - 8**.

`pop ebx` retrieves the most recently pushed value (`0x1234`) and changes `ESP` to **S - 4**. `pop eax` retrieves `0x11223344` and restores `ESP` to **S**. The values are therefore recovered in LIFO order, and the stack is balanced before the program exits.

## End of execution and test observation

The `mov eax, 1` after the stack test replaces the recovered value in `EAX` with the i386 exit syscall number; `xor ebx, ebx` supplies exit status 0. The program assembled and ran successfully as a 32-bit ELF executable, produced no standard output, and exited with code 0.
