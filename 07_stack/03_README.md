# 03.asm: call and ret only (64-bit)

## What the program does
Puts 10 in `rax`, calls `calculate`, which adds 5 and returns. After the call, `rax` holds 15. Then the program exits.

## Build and run
```bash
nasm -f elf64 03.asm -o 03.o
ld 03.o -o 03
./03
echo $?
```

## Stack flow, step by step
Let `S` be the value of `rsp` when `_start` begins.

| Step | Instruction | `rsp` after | Stack contents |
|------|-------------|-------------|----------------|
| 1 | `mov rax, 10` | `S` | empty |
| 2 | `call calculate` | `S - 8` | `[S-8]` = address of `after_call` |
| 3 | `add rax, 5` | `S - 8` | unchanged; `rax = 15` |
| 4 | `ret` | `S` | return address popped into the instruction pointer |
| 5 | `after_call: nop` | `S` | execution continues here |
| 6 | exit system call | `S` | exit with status 0 |

## Explanation
This program has no `push` or `pop`. The stack is used only by `call` and `ret`. `call` pushes the address of the instruction after it (`after_call`) and decreases `rsp` by 8, then jumps to `calculate`. The function does not touch `rsp`, so when `ret` runs, the top of the stack is still the return address. `ret` pops it into the instruction pointer and increases `rsp` by 8, and execution continues at `after_call`. If the function had changed `rsp` without restoring it, `ret` would jump to a wrong address and crash.

## Observations from running
Assembled with `nasm -f elf64`, linked with `ld`, and stepped through in the GNU Debugger (GDB) with `stepi`.

- Program output: none; the program exits with status 0.
- Before `call calculate`: `rsp = 0x7fffffffdc60`, `rax = 10`.
- Inside `calculate` (right after `call`): `rsp = 0x7fffffffdc58` (8 lower). `x/gx $rsp` shows `0x7fffffffdc58: 0x000000000040100a`, the address of `after_call`.
- After `add rax, 5`: `rax = 15`; `rsp` did not change.
- After `ret`: `rsp = 0x7fffffffdc60` (the starting value), `rax = 15`, and execution continued at the `nop` under the `after_call` label.
- No `push` or `pop` appears in this program, but the stack was still used: `call` pushed the return address and `ret` popped it.

## Conclusion
`call` saves where to come back to on the stack and `ret` uses it. Because `calculate` left `rsp` untouched, the return address was still on top when `ret` ran.
