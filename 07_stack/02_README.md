# 02.asm: Stack frame with local variables (64-bit)

## What the program does
`_start` calls the function `work`. `work` builds a stack frame with the base pointer (`rbp`), reserves 16 bytes for two local variables (100 and 200), loads them into `rax` and `rbx`, tears the frame down, and returns. Then the program exits.

## Build and run
```bash
nasm -f elf64 02.asm -o 02.o
ld 02.o -o 02
./02
echo $?
```

## Stack flow, step by step
Let `S` be the value of `rsp` when `_start` begins.

| Step | Instruction | `rsp` after | `rbp` after | What changes on the stack |
|------|-------------|-------------|-------------|----------------------------|
| 1 | `call work` | `S - 8` | old | Return address (the `mov rax, 60` line) pushed at `[S-8]` |
| 2 | `push rbp` | `S - 16` | old | Caller's `rbp` saved at `[S-16]` |
| 3 | `mov rbp, rsp` | `S - 16` | `S - 16` | None; `rbp` now marks the frame |
| 4 | `sub rsp, 16` | `S - 32` | `S - 16` | 16 bytes reserved for locals |
| 5 | `mov qword [rbp-8], 100` | `S - 32` | `S - 16` | `[S-24] = 100` |
| 6 | `mov qword [rbp-16], 200` | `S - 32` | `S - 16` | `[S-32] = 200` |
| 7 | `mov rax, [rbp-8]` / `mov rbx, [rbp-16]` | `S - 32` | `S - 16` | `rax = 100`, `rbx = 200`; nothing pushed or popped |
| 8 | `mov rsp, rbp` | `S - 16` | `S - 16` | Locals released |
| 9 | `pop rbp` | `S - 8` | restored | Caller's `rbp` restored |
| 10 | `ret` | `S` | restored | Return address popped; execution resumes after `call` |

## Explanation
`call` pushes the address of the next instruction and jumps to `work`. The function then follows the standard frame pattern: save the old `rbp`, set `rbp` to the current `rsp`, and subtract from `rsp` to make room for locals. Locals are accessed at fixed negative offsets from `rbp`, which stays constant while `rsp` could move. The locals are written with `mov`, not `push`, so `rsp` does not change when they are stored. To leave, `mov rsp, rbp` discards the locals in one step, `pop rbp` restores the caller's base pointer, and `ret` pops the return address into the instruction pointer. `rsp` ends at `S`, exactly where it started.

## Observations from running
Assembled with `nasm -f elf64`, linked with `ld`, and stepped through in the GNU Debugger (GDB) with `stepi`, printing `rsp` and `rbp` after each instruction. The exact addresses change from run to run, but the differences stay the same.

- Program output: none; the program exits with status 0.
- At `_start`: `rsp = 0x7fffffffdc60`, `rbp = 0x0`.
- After `call work`: `rsp = 0x7fffffffdc58` (8 lower). `x/gx $rsp` shows `0x7fffffffdc58: 0x0000000000401005`, the return address (the `mov rax, 60` line).
- After `push rbp`: `rsp = 0x7fffffffdc50`; the saved value is the caller's `rbp`, which was 0.
- After `mov rbp, rsp`: `rbp = 0x7fffffffdc50`; `rsp` did not change.
- After `sub rsp, 16`: `rsp = 0x7fffffffdc40`; `rbp` stayed at `0x7fffffffdc50`.
- At the `nop`, `x/4gx $rsp` shows:
  - `0x7fffffffdc40: 0xc8` (200, local at `[rbp-16]`) and `0x7fffffffdc48: 0x64` (100, local at `[rbp-8]`)
  - `0x7fffffffdc50: 0x0` (the saved `rbp`) and `0x7fffffffdc58: 0x401005` (the return address)
- At the `nop`: `rax = 100`, `rbx = 200`. Storing the locals with `mov` did not move `rsp`; only `call`, `push` and `sub` did.
- After `mov rsp, rbp`: `rsp = 0x7fffffffdc50`, so the 16 bytes of locals were released at once.
- After `pop rbp`: `rsp = 0x7fffffffdc58` and `rbp = 0x0`, the caller's value restored.
- After `ret`: `rsp = 0x7fffffffdc60`, the starting value, and execution continued at `mov rax, 60` in `_start`.

## Conclusion
The function creates a frame (`call`, `push rbp`, `mov rbp, rsp`, `sub rsp, 16`) and removes it in the opposite order (`mov rsp, rbp`, `pop rbp`, `ret`). The stack pointer ends exactly where it began, which is what lets `ret` find the correct return address.
