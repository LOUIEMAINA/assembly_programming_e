# 06.asm: Byte layout of pushed values (32-bit)

## What the program does
Same idea as `05.asm` in 32-bit mode (`BITS 32`). Loads `0x11223344` into `eax` and `0x1234` into `ebx`, pushes both, pops them back in reverse order, and exits with `int 0x80`.

## Build and run
```bash
nasm -f elf32 06.asm -o 06.o
ld -m elf_i386 06.o -o 06
./06
echo $?
```

## Stack flow, step by step
Let `S` be the value of the 32-bit stack pointer (`esp`) when `_start` begins.

| Step | Instruction | `esp` after | Stack contents (lowest address first) |
|------|-------------|-------------|----------------------------------------|
| 1 | `mov eax, ...` / `mov ebx, 0x1234` | `S` | empty |
| 2 | `push eax` | `S - 4` | `44 33 22 11` |
| 3 | `push ebx` | `S - 8` | `34 12 00 00` then `44 33 22 11` |
| 4 | `nop` | `S - 8` | same as step 3 |
| 5 | `pop ebx` | `S - 4` | `ebx = 0x1234` |
| 6 | `pop eax` | `S` | `eax = 0x11223344` |
| 7 | `mov eax, 1` / `xor ebx, ebx` / `int 0x80` | `S` | exit with status 0 |

## Explanation
In 32-bit mode each `push` writes 4 bytes and lowers `esp` by 4 (compared with 8 in `05.asm`). The values are stored in little-endian order, so `0x11223344` appears as `44 33 22 11` and `0x1234` as `34 12 00 00`. The last value pushed (`ebx`) is on top, so it is popped first, and popping in reverse order restores both registers. `esp` returns to `S`.

## Observations from running
Assembled with `nasm -f elf32`, linked with `ld -m elf_i386`, and stepped through in the GNU Debugger (GDB) with `stepi` and `x/8xb`.

- Program output: none; the program exits with status 0.
- At the first `push`: `esp = 0xffffcda0`.
- After `push eax`: `esp = 0xffffcd9c` (4 lower); after `push ebx`: `esp = 0xffffcd98` (8 lower).
- `x/8xb $esp` at the `nop`: `0xffffcd98: 34 12 00 00 44 33 22 11`. The first four bytes are `ebx` (`0x1234`), and the next four are `eax` (`0x11223344`).
- `x/2wx $esp` shows `0x00001234` and `0x11223344`.
- Little-endian order: `0x11223344` is stored as `44 33 22 11`, with the least significant byte at the lowest address.
- After `pop ebx`: `esp = 0xffffcd9c`, `ebx = 0x1234`. After `pop eax`: `esp = 0xffffcda0`, `eax = 0x11223344`.
- Compared with `05.asm`, each push moves the pointer by 4 here and by 8 there.

## Conclusion
Each 32-bit push writes 4 bytes and lowers `esp` by 4. Popping in reverse order restored both registers, and `esp` returned to its starting value.
