# Prospero

**Prospero** is a fork of [KyTx](https://github.com/TxCsOffc/KyTx) focused on
making native Apple Silicon PlayStation 5 emulation easier to build, test and use.
KyTx builds on [KytyPS5](https://github.com/KytyPS5/KytyPS5) and
[Kyty](https://github.com/InoriRus/Kyty).
It replaces the upstream x86-64 + Rosetta 2 + MoltenVK setup with a native ARM64 build:

- **Guest CPU:** x86-64 game code runs under [FEX](https://github.com/FEX-Emu/FEX) (FEXCore embedded in-process),
  not Rosetta 2.
- **GPU:** RDNA shaders are translated straight to Metal Shading Language and PM4 command streams are executed on a
  native Metal backend (`src/graphics/host_gpu/metal/`). The product path uses no Vulkan, MoltenVK or SPIRV-Cross; the
  inherited Vulkan renderer is kept only as a diagnostic reference.

> [!IMPORTANT]
> Prospero is not affiliated with Sony Interactive Entertainment, PlayStation or any game publisher. The project does not
> distribute games, game data or copyrighted system software. Use only game files that you have obtained legally.

## Compatibility

| Game | Observed behavior |
|---|---|
| Is This Seat Taken? | On a Mac mini M4, reaches the title menu, Barcelona map and first level. |
| Gran Turismo 7 1.71 | On a Mac mini M4, boots to the World Map. The landscape and menu labels still render incorrectly. [GT7 findings](docs/bringup/gt7-metal.md) |
| EA SPORTS FC 27 | In the original KyTx tests on an M5 Max, reached the main menu and a match with rendering errors and occasional crashes or GPU hangs. |

Prospero is an experimental macOS build for Apple Silicon with Metal 4.

## Build and test

Follow the [macOS ARM64 build guide](docs/bringup/macos-arm64.md) for the pinned
FEXCore revision, dependencies and game launch command. The
[GT7 bring-up notes](docs/bringup/gt7-metal.md) list the descriptor and shader
regression tests and explain how to reproduce the Metal captures.

## Development notes

Diagnostic switches are environment variables documented next to their code (for example `KYTY_PASS_TRACE_*`,
`KYTY_GPU_BUSY`, `KYTY_GPU_PASS_TIME` in `src/graphics/host_gpu/metal/`). The upstream KytyPS5 documentation is in
[`docs/`](docs/).

## Credits

- [Kyty](https://github.com/InoriRus/Kyty) by InoriRus and contributors.
- [KytyPS5](https://github.com/KytyPS5/KytyPS5) and its contributors.
- The KytyPS5-UFC5 fork by Reece Ketley.
- [FEX](https://github.com/FEX-Emu/FEX) (MIT).

## License

GPL-2.0, as upstream. See [`LICENSE`](LICENSE) and [`LICENSES/`](LICENSES/).
