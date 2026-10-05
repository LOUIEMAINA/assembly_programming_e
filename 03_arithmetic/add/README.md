# Addition: Flags

## Program 1: 0xFFFFFFFF + 1 = 0x00000000
- Carry Flag: set. The unsigned result does not fit in 32 bits, so a carry came out of the top bit.
- Zero Flag: set. The 32-bit result is exactly zero.
- Sign Flag: cleared. The most significant bit of the result is 0.
- Overflow Flag: cleared. Signed: -1 + 1 = 0, which is correct. Operands have different signs, so signed overflow is impossible.
- Parity Flag: set. The lowest byte is 0x00, which has zero one-bits (an even count).
- Auxiliary Carry Flag: set. 0xF + 0x1 carries out of bit 3 into bit 4.

## Program 2: 0x7FFFFFFF + 1 = 0x80000000
- Carry Flag: cleared. The unsigned result (2,147,483,648) fits in 32 bits.
- Zero Flag: cleared. The result is nonzero.
- Sign Flag: set. The most significant bit of the result is 1.
- Overflow Flag: set. Two positive numbers produced a negative result, so the signed result is wrong.
- Parity Flag: set. The lowest byte is 0x00 (even count of one-bits).
- Auxiliary Carry Flag: set. 0xF + 0x1 carries out of bit 3.