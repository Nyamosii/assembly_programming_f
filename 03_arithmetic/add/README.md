# Arithmetic Operations: add

I assembled each program with `nasm -f elf32`, linked it with `ld -m elf_i386`, and stepped through it in GDB (`starti`, then `stepi`). After the `add`/`adc` instruction ran, I read the flags with `info registers eflags`.

flag checks:
* CF looks at the result as an unsigned number (did it carry out of the top bit?).
* OF looks at the result as a signed number (is the sign wrong for the operands?).
* ZF = result is zero, SF = top bit of the result is 1, PF = low byte has an even number of 1 bits, AF = carry from bit 3 into bit 4.

---

## add1.asm
**Operation:** Adds 120 (`0x78`, `0111 1000`) and 10 (`0x0A`, `0000 1010`) in the 8-bit register `AL`. The result is `0x82` (`1000 0010`), which is 130 if read as unsigned but -126 if read as signed.

**GDB:** `eflags` went from `0x202 [ IF ]` to `0xa96 [ PF AF SF IF OF ]`, and `AL = 0x82`.

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 - the result `0x82` is not zero.
* **CF (Carry Flag):** 0 - 130 still fits in 8 bits unsigned (max 255), so nothing carried out of bit 7.
* **SF (Sign Flag):** 1 - the most significant bit (bit 7) of `1000 0010` is 1.
* **OF (Overflow Flag):** 1 - two positive signed numbers (+120 and +10) gave a result whose sign bit is 1 (-126). The real answer, +130, does not fit in -128 to +127, so signed overflow happened.
* **PF (Parity Flag):** 1 - the low byte `1000 0010` has two 1 bits, which is an even count.
* **AF (Auxiliary Carry Flag):** 1 - the low nibbles added up to 8 + 10 = 18, which is more than 15, so a carry went from bit 3 into bit 4.

**Takeaway:** the same bits are fine as unsigned (CF=0) but wrong as signed (OF=1). CF and OF judge the same addition in two different ways.

---

## add2.asm
**Operation:** Adds 32000 (`0x7D00`) and 500 (`0x01F4`) in the 16-bit register `AX`. The result is 32500 (`0x7EF4`).

**GDB:** `eflags` stayed at `0x202 [ IF ]` after the add, and `AX = 0x7ef4`. No arithmetic flag is set.

**Flags Status & Explanation:**
* **ZF (Zero Flag):** 0 - the result is not zero.
* **CF (Carry Flag):** 0 - 32500 is below 65535, so there is no carry out of bit 15.
* **SF (Sign Flag):** 0 - bit 15 of `0x7EF4` is 0.
* **OF (Overflow Flag):** 0 - 32500 is not above 32767 (the biggest signed 16-bit number), and positive + positive stayed positive.
* **PF (Parity Flag):** 0 - the low byte `0xF4` (`1111 0100`) has five 1 bits, which is odd.
* **AF (Auxiliary Carry Flag):** 0 - the low nibbles added up to 0 + 4 = 4, so there was no carry into bit 4.

**Takeaway:** this is a clean addition where everything fits, so every flag is clear. It is a good baseline to compare the other programs with.

---

## add3.asm
**Operation:** Adds `0xFFFF` (65535) and 1 in `AX`, then runs `adc ax, 0` to add the carry on top. The true sum is `0x10000`, which needs 17 bits, so `AX` keeps only `0x0000` and the extra bit goes into CF.

**GDB:**
* After `add ax, [num2]`: `eflags = 0x257 [ CF PF AF ZF IF ]`, `AX = 0x0000`
* After `adc ax, 0`: `eflags = 0x202 [ IF ]`, `AX = 0x0001`

**Flags Status & Explanation (after the `add`):**
* **ZF (Zero Flag):** 1 - the 16 bits that were kept (`0x0000`) are all zero.
* **CF (Carry Flag):** 1 - the carry out of bit 15 was lost from `AX` and landed in CF, an unsigned overflow.
* **SF (Sign Flag):** 0 - bit 15 of `0x0000` is 0.
* **OF (Overflow Flag):** 0 - as signed numbers this is (-1) + (+1) = 0, which is correct. Adding a negative and a positive can never overflow.
* **PF (Parity Flag):** 1 - the low byte `0x00` has zero 1 bits, which counts as even.
* **AF (Auxiliary Carry Flag):** 1 - the low nibbles added up to 0xF + 1 = 0x10, so a carry went from bit 3 into bit 4.

**Flags Status & Explanation (after the `adc ax, 0`):**
* **ZF (Zero Flag):** 0 - `AX` is now 1 (0 + 0 + CF), so it is not zero. ZF was recalculated.
* **CF (Carry Flag):** 0 - `adc` used up the old carry, and the new result produces no carry out.
* **SF (Sign Flag):** 0 - bit 15 of `0x0001` is 0.
* **OF (Overflow Flag):** 0 - 0 + 0 + 1 = 1, no signed overflow.
* **PF (Parity Flag):** 0 - `0x01` has one 1 bit, which is odd.
* **AF (Auxiliary Carry Flag):** 0 - no carry out of bit 3 this time.

**Takeaway:** every arithmetic instruction recalculates the flags from its own result, which is why CF, ZF, PF and AF all dropped back to 0 after `adc`. `adc` is also how a carry gets passed on in multi-word addition.
