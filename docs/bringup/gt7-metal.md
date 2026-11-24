# Prospero · GT7 on native Metal

Prospero is a KyTx fork for Apple Silicon. This branch builds on KyTx commit
`d5a3160`. See [macOS ARM64 build instructions](macos-arm64.md) for the
dependencies and commands used by the existing checkout.

GT7 1.71 reaches the World Map on the M4. The PS5 reference shows the coastline,
terrain, water, buildings, and readable menu labels. The Metal capture preserves
the shape of the coast and some icons, but the map is red and the labels are
absent. The World Map remains an open compatibility target.

The Metal device reports 32,768 bytes of threadgroup memory. The first GT7 mesh
pipeline originally requested 34,000 bytes. KyTx now packs the three ten-bit
primitive indices into one word for staging and preserves the guest LDS size.
Metal pipeline errors report the device limit.

Run with `KYTY_LOG_UNBOUND=1` to record the guest shader hash, Metal function
key, resource slot, descriptor source and first use PC. Missing snapshots,
null descriptors and non-null descriptors missing a Metal texture can be
distinguished in the trace. The two screenshots are final frames. Intermediate
PS5 render targets are needed for comparisons of individual passes.

Run the focused tests in the existing build directory:

```sh
cmake --build "$build" --target resource_tracking_tests resource_materialization_tests
ctest --test-dir "$build" --output-on-failure -R '^resource_(tracking|materialization)$'
```
