# 01.asm: Basic push and pop (64-bit)

## What the program does
Loads 100 into `rax` and 200 into `rbx`, pushes both onto the stack, pops them into `rcx` and `rdx`, then exits with the Linux exit system call (number 60).

## Build and run
```bash
nasm -f elf64 01.asm -o 01.o
ld 01.o -o 01
./01
echo $?        # exit status 0
```

## Stack flow, step by step
Let `S` be the value of the stack pointer (`rsp`) when `_start` begins. The stack grows toward lower addresses.

| Step | Instruction | `rsp` after | Stack contents (lowest address first) |
|------|-------------|-------------|----------------------------------------|
| 1 | `mov rax, 100` / `mov rbx, 200` | `S` | nothing pushed yet |
| 2 | `push rax` | `S - 8` | `[S-8] = 100` |
| 3 | `push rbx` | `S - 16` | `[S-16] = 200`, `[S-8] = 100` |
| 4 | `nop` (breakpoint spot) | `S - 16` | same as step 3 |
| 5 | `pop rcx` | `S - 8` | `rcx = 200`; `[S-8] = 100` still on the stack |
| 6 | `pop rdx` | `S` | `rdx = 100`; stack back to its starting state |
| 7 | `mov rax, 60`, `xor rdi, rdi`, `syscall` | `S` | exit with status 0 |

## Explanation
Each 64-bit `push` first subtracts 8 from `rsp`, then writes the value at the new `rsp`. Each `pop` reads the value at `rsp`, then adds 8. The stack is last in, first out, so the last value pushed (200, from `rbx`) is the first one popped (into `rcx`), and 100 ends up in `rdx`. The values are reversed relative to the registers they came from. After the two pops, `rsp` equals `S` again, so the pushes and pops are balanced.

## Observations from running
Assembled with `nasm -f elf64`, linked with `ld`, and stepped through in the GNU Debugger (GDB) using `stepi`, `info registers` and `x/2gx $rsp`.

- Program output: none; the program exits normally with status 0.
- `rsp` at `_start`: `0x7fffffffdae0`
- After `push rax`: `rsp = 0x7fffffffdad8` (8 lower)
- After `push rbx`: `rsp = 0x7fffffffdad0` (16 lower than the start)
- `x/2gx $rsp` at the `nop`: `0x7fffffffdad0: 0x00000000000000c8  0x0000000000000064`. So 200 is on top at the lower address, and 100 is below it at the higher address, because the stack grows downward.
- At the `nop`: `rax = 100`, `rbx = 200`. A push copies the value and does not clear the register.
- After `pop rcx`: `rcx = 200`, `rsp = 0x7fffffffdad8`.
- After `pop rdx`: `rdx = 100`, `rsp = 0x7fffffffdae0`, the same as the starting value, so the two pushes and two pops are balanced.
- The values came out in reverse order of how they were pushed (last in, first out).
- The values 200 and 100 still remain in memory below `rsp` after the pops, but they are no longer part of the stack. `pop` only moves `rsp`; it does not erase the data.

## Conclusion
The program pushes 100, then 200. Because the stack is last in, first out, 200 is popped first into `rcx`, then 100 into `rdx`. The stack pointer (`rsp`) falls by 8 on each push and rises by 8 on each pop, and returns to its starting value before the program exits.
