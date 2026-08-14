# solder-pico2

**[SOLDER](https://github.com/sysl-lang/solder) on a Raspberry Pi Pico 2 W**, over the USB serial
port. Plug the board in, open the port, and type at the language.

```
SOLDER on a Raspberry Pi Pico 2 W. Type a line, or BYE to leave.

solder> 5 3 + PR
8
solder> DEF SQUARE DUP * END
solder> 7 SQUARE PR
49
solder> 5 0 DO I PR SPACE LOOP
0 1 2 3 4
```

## What is here

**`solder/main.sysl` is the desktop console with its first few lines swapped, and nothing else.** Both
build a `sysl.term.edit.Editor` and hand it to `sh.sysl.solder.session`; this one reads the USB port,
the desktop's reads standard input after putting the terminal into raw mode. There is no mode to
change here, because a USB CDC port has no line discipline to get out of the way of — which is the
reason a line editor had to exist at all.

**There is no C in this project.** sysl exports `main`, the SDK's `crt0` branches straight into it,
and `libsolder.a` in the executable's source list is what satisfies CMake's "a target must have
sources" rule with no translation unit to infer a language from.

## Building it

```
export PICO_SDK_PATH=/path/to/pico-sdk
cmake -B build .
cmake --build build -j8
```

`build/sysl_solder.uf2` is the image. Hold BOOTSEL, plug the board in, and copy it to the volume that
appears.

The RP2350 has two personalities and this project builds for either — a pair of Cortex-M33s or a pair
of Hazard3 RISC-V cores — which is a build-time choice rather than anything the board does:

```
cmake -B build-riscv -DPICO_PLATFORM=rp2350-riscv -DPICO_TOOLCHAIN_PATH=/path/to/riscv-toolchain-15
```

## What it costs

**400 KB of flash and 5.3 KB of static RAM**, against the Pico 2 W's 4 MB and 520 KB. Everything else
the language needs — the dictionary, the stacks, the strings and arrays a program builds — comes off
the heap as it is asked for, which is what `requires { heap = true }` in `solder/package.hocon` is
declaring.

For scale, [ogol-pico2](https://github.com/sysl-lang/ogol-pico2) is 189 KB of flash on the same board.
The difference is mostly floating point: `SIN` and its neighbours reach newlib's libm, `FORMAT` and
`PR` reach its float rendering, and a language with no floats pays for none of it.

## Licence

ISC — see `LICENSE`.
