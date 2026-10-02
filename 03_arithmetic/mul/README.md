# Arithmetic Operations: mul

I assembled each program with `nasm -f elf32`, linked it with `ld -m elf_i386`, and stepped through it in GDB (`starti`, then `stepi`). After the `mul` instruction ran, I read the registers and flags with `info registers eax edx eflags`.

`MUL` is an **unsigned** multiply, and the result is twice as wide as the inputs: `AL × r/m8` goes into `AX`, `AX × r/m16` goes into `DX:AX`, and `EAX × r/m32` goes into `EDX:EAX`.

Only two flags have a defined meaning after `MUL`:
* **CF and OF** are both set to 1 if the upper half of the result (`AH`, `DX` or `EDX`) is not zero, which means the answer did not fit in the lower half. Otherwise both are 0.
* ZF, SF, PF and AF are undefined. The CPU leaves some value in them, but it does not describe the product. I list what GDB showed and mark them "undefined" instead of explaining them as if they were real results.

---

## mul1.asm
**Operation:** Multiplies 25 by 10 using `mul byte [num2]`. `AL` (25) times 10 gives 250 (`0xFA`), which is stored in `AX`.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0x286 [ PF SF IF ]`, and `AX = 0x00fa` with `AH = 0x00`.

**Flags Status & Explanation:**
* **CF (Carry Flag):** 0 - the upper half of the result (`AH`) is `0x00`, so 250 fits in 8 bits.
* **OF (Overflow Flag):** 0 - it follows the same rule as CF, and the upper half is zero.
* **ZF (Zero Flag):** 0 (undefined) - `MUL` does not define ZF. It shows 0 here, but you cannot use it to tell if the product is zero.
* **SF (Sign Flag):** 1 (undefined) - `MUL` does not define SF. It shows 1 here only because bit 7 of `0xFA` happens to be 1, and `MUL` is unsigned anyway.
* **PF (Parity Flag):** 1 (undefined) - not defined by `MUL`. It shows 1 here because `0xFA` has an even number of 1 bits.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `MUL`.

**Takeaway:** CF=OF=0 only tells us "the result fits in `AL`". It says nothing about positive or negative, because `MUL` is unsigned.

---

## mul2.asm
**Operation:** Multiplies 3000 (`0x0BB8`) by 200 using `mul word [num2]`. The product is 600000 (`0x927C0`), which does not fit in 16 bits, so it is split into `DX:AX`.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0xa07 [ CF PF IF OF ]`, with `DX = 0x0009` and `AX = 0x27c0`.

**Flags Status & Explanation:**
* **CF (Carry Flag):** 1 - 600000 is bigger than 65535, so the upper half `DX = 9` is not zero.
* **OF (Overflow Flag):** 1 - it is set together with CF for the same reason, because `DX` is not zero.
* **ZF (Zero Flag):** 0 (undefined) - not defined by `MUL`.
* **SF (Sign Flag):** 0 (undefined) - not defined by `MUL`.
* **PF (Parity Flag):** 1 (undefined) - not defined by `MUL`.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `MUL`.

**Takeaway:** CF=OF=1 is a warning that you must read `DX` as well as `AX`, otherwise you lose the top part of the answer.

---

## mul3.asm
**Operation:** Multiplies 100000 (`0x186A0`) by 300000 using `mul dword [num2]`. The product is 30,000,000,000, which is `0x6_FC23AC00` and needs 35 bits, so it is split into `EDX:EAX`.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0xa87 [ CF PF SF IF OF ]`, with `EDX = 0x00000006` and `EAX = 0xfc23ac00`.

**Flags Status & Explanation:**
* **CF (Carry Flag):** 1 - 3 × 10^10 is bigger than the largest 32-bit number (about 4.29 × 10^9), so the upper half `EDX = 6` is not zero.
* **OF (Overflow Flag):** 1 - it is set together with CF, because `EDX` is not zero.
* **ZF (Zero Flag):** 0 (undefined) - not defined by `MUL`.
* **SF (Sign Flag):** 1 (undefined) - not defined by `MUL`. It happens to match bit 31 of `EAX` here, but that is not guaranteed.
* **PF (Parity Flag):** 1 (undefined) - not defined by `MUL`.
* **AF (Auxiliary Carry Flag):** 0 (undefined) - not defined by `MUL`.

**Takeaway:** the rule is the same at every size (8, 16 and 32 bits): CF=OF=1 exactly when the upper half (`AH`, `DX` or `EDX`) is not zero.
