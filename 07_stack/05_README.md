# 05.asm: Byte layout of pushed values (64-bit)

## What the program does
Loads a full 64-bit value `0x1122334455667788` into `rax` and `0x1234` into `rbx`, pushes both, then pops them back in reverse order into the same registers. Then it exits. The point of this program is to see how values are stored in memory.

## Build and run
```bash
nasm -f elf64 05.asm -o 05.o
ld 05.o -o 05
./05
echo $?
```

## Stack flow, step by step
Let `S` be the value of `rsp` when `_start` begins.

| Step | Instruction | `rsp` after | Stack contents (lowest address first) |
|------|-------------|-------------|----------------------------------------|
| 1 | `mov rax, ...` / `mov rbx, 0x1234` | `S` | empty |
| 2 | `push rax` | `S - 8` | `88 77 66 55 44 33 22 11` |
| 3 | `push rbx` | `S - 16` | `34 12 00 00 00 00 00 00` then `88 77 66 55 44 33 22 11` |
| 4 | `nop` | `S - 16` | same as step 3 |
| 5 | `pop rbx` | `S - 8` | `rbx = 0x1234` |
| 6 | `pop rax` | `S` | `rax = 0x1122334455667788` |
| 7 | exit system call | `S` | exit with status 0 |

## Explanation
A 64-bit `push` always writes 8 bytes, so even the small value `0x1234` takes a full 8-byte slot, padded with zero bytes. The processor stores numbers in little-endian order, meaning the least significant byte goes at the lowest address. That is why `0x...7788` appears in memory as `88 77 ...` and `0x1234` appears as `34 12 ...`. Because the pops are in the reverse order of the pushes and into the same registers, each register gets its original value back, and `rsp` returns to `S`.

## Observations from running
Assembled with `nasm -f elf64`, linked with `ld`, and stepped through in the GNU Debugger (GDB) with `stepi` and `x/16xb`.

- Program output: none; the program exits with status 0.
- At the first `push`: `rsp = 0x7fffffffdc60`.
- After `push rax`: `rsp = 0x7fffffffdc58`; after `push rbx`: `rsp = 0x7fffffffdc50`.
- `x/16xb $rsp` at the `nop`:
  - `0x7fffffffdc50: 34 12 00 00 00 00 00 00` (from `rbx`)
  - `0x7fffffffdc58: 88 77 66 55 44 33 22 11` (from `rax`)
- `x/2gx $rsp` shows `0x0000000000001234` and `0x1122334455667788`, because GDB reads 8 bytes as one number.
- The least significant byte is stored at the lowest address (little-endian), so `0x...7788` appears as `88 77 ...`. The small value `0x1234` still takes a full 8-byte slot, padded with zero bytes.
- After `pop rbx`: `rsp = 0x7fffffffdc58`, `rbx = 0x1234`. After `pop rax`: `rsp = 0x7fffffffdc60`, `rax = 0x1122334455667788`.

## Conclusion
Popping in the reverse order of the pushes restored both registers, and `rsp` returned to its starting value. The memory view shows how the processor lays out numbers in bytes.
