# Division: Flags

After `div`, all arithmetic flags (Carry, Zero, Sign, Overflow, Parity, Auxiliary Carry) are undefined. The processor does not report the result through flags; it returns quotient in EAX and remainder in EDX. Flag values in GDB are leftovers from earlier instructions and must not be interpreted.

## Program 1: 10 / 3
- Quotient EAX = 3, remainder EDX = 1.
- Flags: undefined. A zero remainder or quotient does not set the Zero Flag.

## Program 2: 100 / 10
- Quotient EAX = 10, remainder EDX = 0.
- Flags: undefined. The remainder being zero does not set the Zero Flag.

Note: dividing by zero or a quotient too large for EAX raises a divide error exception (SIGFPE in GDB) rather than setting a flag.