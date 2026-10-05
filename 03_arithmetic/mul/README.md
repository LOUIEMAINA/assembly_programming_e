# Multiplication: Flags

Only the Carry Flag and Overflow Flag are defined after `mul`. Zero, Sign, Parity, and Auxiliary Carry Flags are undefined (any value GDB shows is meaningless).

## Program 1: 5 * 3 = 15 (EDX:EAX = 0:15)
- Carry Flag: cleared. The upper half (EDX) is zero, so the result fits in the lower half.
- Overflow Flag: cleared. Same condition as the Carry Flag for `mul`.

## Program 2: 0x10000 * 0x10000 = 0x1_00000000 (EDX = 1, EAX = 0)
- Carry Flag: set. The upper half (EDX) is nonzero, so the result does not fit in 32 bits.
- Overflow Flag: set. Same reason: the upper half holds significant bits.