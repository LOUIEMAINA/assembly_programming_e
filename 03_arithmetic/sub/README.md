# Subtraction: Flags

## Program 1: 5 - 5 = 0
- Carry Flag: cleared. Carry Flag means borrow here; no borrow was needed because 5 >= 5 (unsigned).
- Zero Flag: set. The result is zero.
- Sign Flag: cleared. The result's most significant bit is 0.
- Overflow Flag: cleared. Same-sign operands subtracted cannot overflow.
- Parity Flag: set. Lowest byte 0x00 has an even count of one-bits.
- Auxiliary Carry Flag: cleared. The low nibble 5 - 5 needs no borrow.

## Program 2: 3 - 5 = 0xFFFFFFFE (-2)
- Carry Flag: set. 3 < 5 unsigned, so a borrow was needed.
- Zero Flag: cleared. The result is nonzero.
- Sign Flag: set. The most significant bit of 0xFFFFFFFE is 1.
- Overflow Flag: cleared. Signed 3 - 5 = -2 fits in 32 bits.
- Parity Flag: cleared. The lowest byte 0xFE has seven one-bits (odd count).
- Auxiliary Carry Flag: set. Low nibble 3 < 5, so a borrow came from bit 4.