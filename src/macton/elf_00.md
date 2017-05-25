Following up my look into the [x64 PE file format on Windows](pe_00.html), I also wanted to have a deeper look at building an ELF formated executable on Linux. Turns out that's pretty straightforward.

As before, the files are all written in [fasm](https://flatassembler.net/). 

| File                            | Description                                                         |
| --------------------------------| ------------------------------------------------------------------- |
| [01_elf_return.asm](https://github.com/macton/x64-fasm-examples/blob/master/Linux/00_BasicOS/01_elf_return.asm) | Simple example assembling ELF executable, return 42.            |
| [02_elf_bss.asm](https://github.com/macton/x64-fasm-examples/blob/master/Linux/00_BasicOS/02_elf_bss.asm)       | Add ".bss" section by increasing in-memory size of the section. |
| [03_elf_libc.asm](https://github.com/macton/x64-fasm-examples/blob/master/Linux/00_BasicOS/03_elf_libc.asm)     | Dynamically link libc.so.6, call printf as example.             |

