# `04.asm` — 32-bit function stack frame and local variables

## What happens to the stack

This is a 32-bit program, so each pushed address or register value changes `ESP` by 4 bytes. Let entry `ESP` be **S**. `call work` pushes a 4-byte return address and sets `ESP` to **S - 4**.

Inside `work`, `push ebp` saves the caller's frame pointer at **S - 8**. `mov ebp, esp` establishes that location as the frame base. `sub esp, 8` reserves two 4-byte local-variable slots, setting `ESP` to **S - 16**. The stores set `[EBP - 4] = 100` and `[EBP - 8] = 200`; they fill the reserved slots without changing `ESP`. Loading those slots gives `EAX = 100` and `EBX = 200`.

The epilogue exactly unwinds the frame: `mov esp, ebp` removes the local area, so `ESP` becomes **S - 8**. `pop ebp` restores the previous frame pointer and changes `ESP` to **S - 4**. Finally, `ret` pops the return address, resumes `_start`, and restores `ESP` to **S**.

## End of execution and test observation

`_start` then executes the i386 Linux `exit(0)` syscall. The program assembled and ran successfully as a 32-bit ELF executable, printed no output, and exited with code 0. Its stack flow is the 32-bit version of `02.asm`: the key difference is 4-byte stack entries rather than 8-byte ones.
