# Arithmetic Operations: sub

I assembled each program with `nasm -f elf32`, linked it with `ld -m elf_i386`, and stepped through it in GDB (`starti`, then `stepi`). After the `sub`/`sbb` instruction ran, I read the flags with `info registers eflags`.

For subtraction, CF means borrow, not carry. It is set when the first number is smaller than the second as unsigned numbers. **OF** is set when the signed result is wrong, which can only happen when the two operands have different signs.

---

## sub1.asm
**Operation:** Subtracts 80 (`0x50`) from 50 (`0x32`) in the 8-bit register `AL`. The answer is -30, which in 8-bit two's complement is `0xE2` (`1110 0010`), or 226 if read as unsigned.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0x287 [ CF PF SF IF ]`, and `AL = 0xe2`.

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 - the result `0xE2` is not zero, because the two numbers were not equal.
* **CF (Carry Flag):** 1 - as unsigned numbers, 50 is smaller than 80, so a borrow was needed.
* **SF (Sign Flag):** 1 - bit 7 of `1110 0010` is 1, so the result is negative (-30).
* **OF (Overflow Flag):** 0 - -30 fits comfortably in -128 to +127. Both operands are positive, and positive minus positive can never overflow.
* **PF (Parity Flag):** 1 - the low byte `1110 0010` has four 1 bits, which is even.
* **AF (Auxiliary Carry Flag):** 0 - the low nibbles were 2 - 0, so no borrow from bit 4 was needed.

**Takeaway:** CF=1 says "unsigned borrow" and SF=1 says "looks negative". Here they agree, and the signed answer is valid (OF=0).

---

## sub2.asm
**Operation:** Subtracts 2000 (`0x07D0`) from 1000 (`0x03E8`) in the 16-bit register `AX`. The answer is -1000, which is `0xFC18`.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0x287 [ CF PF SF IF ]`, and `AX = 0xfc18`.

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 - the result is not zero.
* **CF (Carry Flag):** 1 - as unsigned numbers, 1000 is smaller than 2000, so a borrow was needed.
* **SF (Sign Flag):** 1 - bit 15 of `0xFC18` is 1, so the result is negative.
* **OF (Overflow Flag):** 0 - -1000 fits easily in signed 16-bit, and both operands are positive, so overflow is impossible.
* **PF (Parity Flag):** 1 - the low byte `0x18` (`0001 1000`) has two 1 bits, which is even.
* **AF (Auxiliary Carry Flag):** 0 - the low nibbles were 8 - 0, so no borrow from bit 4.

**Takeaway:** this is the same pattern as `sub1.asm`, just 16-bit. SF now looks at bit 15 instead of bit 7, because the flags depend on the size of the operation.

---

## sub3.asm
**Operation:** Subtracts 1 from 0 in `AX`, which wraps around to `0xFFFF`, and then runs `sbb ax, 0` to subtract the borrow as well.

**GDB:**
* After `sub ax, [num2]`: `eflags = 0x297 [ CF PF AF SF IF ]`, `AX = 0xffff`
* After `sbb ax, 0`: `eflags = 0x282 [ SF IF ]`, `AX = 0xfffe`

**Flags Status & Explanation (after the `sub`):**
* **ZF (Zero Flag):** 0 - the result `0xFFFF` is not zero.
* **CF (Carry Flag):** 1 - 0 is smaller than 1 as unsigned numbers, so a borrow was needed. That borrow is why the result wrapped around to `0xFFFF`.
* **SF (Sign Flag):** 1 - bit 15 of `0xFFFF` is 1 (it means -1 as signed).
* **OF (Overflow Flag):** 0 - signed 0 - 1 = -1 is a valid 16-bit value, so there is no signed overflow.
* **PF (Parity Flag):** 1 - the low byte `0xFF` has eight 1 bits, which is even.
* **AF (Auxiliary Carry Flag):** 1 - the low nibbles were 0 - 1, which needs a borrow from bit 4.

**Flags Status & Explanation (after the `sbb ax, 0`):**
* **ZF (Zero Flag):** 0 - the result `0xFFFE` is not zero.
* **CF (Carry Flag):** 0 - `sbb` used up the borrow (0xFFFF - 0 - 1 = 0xFFFE), and `0xFFFF` is large enough that no new borrow is needed.
* **SF (Sign Flag):** 1 - bit 15 of `0xFFFE` is still 1.
* **OF (Overflow Flag):** 0 - -1 - 0 - 1 = -2, which fits in signed 16-bit.
* **PF (Parity Flag):** 0 - the low byte `0xFE` (`1111 1110`) has seven 1 bits, which is odd.
* **AF (Auxiliary Carry Flag):** 0 - the low nibbles were 0xF - 0 - 1 = 0xE, so no borrow from bit 4.

**Takeaway:** the flags are recalculated by every instruction. After `sbb`, CF, AF and PF changed because the new result is different, while SF stayed 1. `sbb` is how a borrow is passed on in multi-word subtraction.
