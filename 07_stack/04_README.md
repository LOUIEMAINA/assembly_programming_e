# 04.asm: Stack frame with local variables (32-bit)

## What the program does
Same idea as `02.asm`, but written for 32-bit mode (`BITS 32`). `_start` calls `work`, which builds a frame with the 32-bit base pointer (`ebp`), reserves 8 bytes for two locals (100 and 200), loads them into `eax` and `ebx`, tears the frame down, and returns. The exit uses the 32-bit system call `int 0x80` with `eax = 1`.

## Build and run
```bash
nasm -f elf32 04.asm -o 04.o
ld -m elf_i386 04.o -o 04
./04
echo $?
```

## Stack flow, step by step
Let `S` be the value of the 32-bit stack pointer (`esp`) when `_start` begins. Every stack slot is 4 bytes.

| Step | Instruction | `esp` after | `ebp` after | What changes on the stack |
|------|-------------|-------------|-------------|----------------------------|
| 1 | `call work` | `S - 4` | old | Return address pushed at `[S-4]` |
| 2 | `push ebp` | `S - 8` | old | Caller's `ebp` saved at `[S-8]` |
| 3 | `mov ebp, esp` | `S - 8` | `S - 8` | None |
| 4 | `sub esp, 8` | `S - 16` | `S - 8` | 8 bytes reserved for locals |
| 5 | `mov dword [ebp-4], 100` | `S - 16` | `S - 8` | `[S-12] = 100` |
| 6 | `mov dword [ebp-8], 200` | `S - 16` | `S - 8` | `[S-16] = 200` |
| 7 | `mov eax, [ebp-4]` / `mov ebx, [ebp-8]` | `S - 16` | `S - 8` | `eax = 100`, `ebx = 200` |
| 8 | `mov esp, ebp` | `S - 8` | `S - 8` | Locals released |
| 9 | `pop ebp` | `S - 4` | restored | Caller's `ebp` restored |
| 10 | `ret` | `S` | restored | Return address popped |

## Explanation
The logic matches `02.asm`, but each slot is 4 bytes instead of 8, so `call`, `push` and `pop` move `esp` by 4. The locals are reserved with `sub esp, 8` (two 4-byte values) and addressed relative to `ebp` at `[ebp-4]` and `[ebp-8]`. Storing them with `mov` does not change `esp`. Leaving the function reverses the setup in the opposite order: restore `esp` from `ebp`, pop the saved `ebp`, then `ret` pops the return address. `esp` ends at `S`.

## Observations from running
Assembled with `nasm -f elf32`, linked with `ld -m elf_i386`, and stepped through in the GNU Debugger (GDB) with `stepi`. In 32-bit mode every slot is 4 bytes.

- Program output: none; the program exits with status 0.
- At `_start`: `esp = 0xffffcda0`, `ebp = 0x0`.
- After `call work`: `esp = 0xffffcd9c` (4 lower). `x/wx $esp` shows `0xffffcd9c: 0x08049005`, the return address.
- After `push ebp`: `esp = 0xffffcd98`, saved `ebp` is 0.
- After `mov ebp, esp`: `ebp = 0xffffcd98`.
- After `sub esp, 8`: `esp = 0xffffcd90`.
- At the `nop`, `x/4wx $esp` shows: `0xffffcd90: 0x000000c8  0x00000064  0x00000000  0x08049005`, which is 200, 100, the saved `ebp`, and the return address.
- At the `nop`: `eax = 100`, `ebx = 200`.
- After `mov esp, ebp`: `esp = 0xffffcd98`; after `pop ebp`: `esp = 0xffffcd9c` and `ebp = 0x0`; after `ret`: `esp = 0xffffcda0`, the starting value.
- Compared with `02.asm`, every step moves the pointer by 4 instead of 8, and the locals take 8 bytes instead of 16.

## Conclusion
The frame is built and removed exactly as in the 64-bit version. The stack pointer returns to `0xffffcda0` before the exit.
