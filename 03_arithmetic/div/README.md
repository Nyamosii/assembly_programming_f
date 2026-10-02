# Arithmetic Operations: div

I assembled each program with `nasm -f elf32`, linked it with `ld -m elf_i386`, and stepped through it in GDB (`starti`, then `stepi`). After the `div` instruction ran, I read the registers and flags with `info registers eax edx eflags`.

`DIV` is an **unsigned** divide, and the dividend is twice as wide as the divisor:
* 8-bit divisor: `AX` ÷ divisor gives the quotient in `AL` and the remainder in `AH`.
* 16-bit divisor: `DX:AX` ÷ divisor gives the quotient in `AX` and the remainder in `DX`.
* 32-bit divisor: `EDX:EAX` ÷ divisor gives the quotient in `EAX` and the remainder in `EDX`.

After `DIV`, all six arithmetic flags (CF, OF, SF, ZF, AF, PF) are undefined. Unlike `ADD` and `SUB`, they do not describe the result, so ZF does not tell you whether the quotient or remainder is zero. In all three programs GDB showed `eflags = 0x202 [ IF ]` both before and after the `div`, so the flags were simply left unchanged on my machine. Another CPU could leave different values, which is why they should never be relied on.

`DIV` reports problems differently. If the divisor is 0, or the quotient is too big for its register, the CPU raises a divide error (`#DE`) and the program crashes with SIGFPE on Linux. That is the real "error signal", not a flag.

---

## div1.asm
**Operation:** Divides 100 by 7 using `div bl`. The dividend is `AX = 100`, and the result is quotient 14 in `AL` and remainder 2 in `AH`.

**GDB:** `eflags` stayed at `0x202 [ IF ]`, and `AX = 0x020e`, which means `AL = 0x0e` (14) and `AH = 0x02` (2).

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 (undefined) - `DIV` does not define it. It is 0 only because it was already 0 before the division.
* **CF (Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **SF (Sign Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **OF (Overflow Flag):** 0 (undefined) - not defined by `DIV`, left unchanged. Overflow in a division is reported by the `#DE` exception, not by OF.
* **PF (Parity Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.

**Takeaway:** the quotient 14 fits in `AL`, so there is no divide error. Check: 14 × 7 + 2 = 100.

---

## div2.asm
**Operation:** Divides 50000 by 300 using `div bx`. The dividend is `DX:AX` (with `DX = 0` and `AX = 50000`), and the result is quotient 166 in `AX` and remainder 200 in `DX`.

**GDB:** `eflags` stayed at `0x202 [ IF ]`, with `AX = 0x00a6` (166) and `DX = 0x00c8` (200).

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 (undefined) - not defined by `DIV`, so it tells us nothing about the quotient.
* **CF (Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **SF (Sign Flag):** 0 (undefined) - not defined by `DIV`. The quotient `0xA6` has bit 7 set, yet SF is 0, which shows SF is not updated from the result.
* **OF (Overflow Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **PF (Parity Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.

**Takeaway:** the program clears `DX` before dividing because `DIV bx` uses `DX:AX` as the dividend. If `DX` held a leftover non-zero value, the quotient would not fit in `AX` and the program would crash with `#DE`. Check: 166 × 300 + 200 = 50000.

---

## div3.asm
**Operation:** Divides 300,000,000 by 1000 using `div ebx`. The dividend is `EDX:EAX` (with `EDX = 0`), and the result is quotient 300000 in `EAX` and remainder 0 in `EDX`.

**GDB:** `eflags` stayed at `0x202 [ IF ]`, with `EAX = 0x000493e0` (300000) and `EDX = 0x00000000`.

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 (undefined) - not defined by `DIV`. The remainder is exactly 0, yet ZF is not set, which proves ZF does not reflect the result of a division.
* **CF (Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **SF (Sign Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **OF (Overflow Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **PF (Parity Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `DIV`, left unchanged.

**Takeaway:** this is the clearest example. The remainder is 0, which looks like what ZF would show, but ZF stays clear because `DIV` does not touch it. To check for an exact division, test `EDX` yourself (for example `test edx, edx`), which does set ZF.
