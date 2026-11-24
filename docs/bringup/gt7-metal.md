# Prospero · GT7 on native Metal

Gran Turismo 7 1.71 reaches the World Map on a Mac mini M4. The PS5 reference
shows the coastline, terrain, water, buildings and readable menu labels. The
Metal capture preserves the coastline and some icons. The red overlay is gone.
The landscape is still mostly black, and menu labels are missing.

## What changed since KyTx

| Commit | Change |
| --- | --- |
| `859671c` | Adapted the Darwin FEX integration and Unity startup. |
| `2964105`, `d5a3160` | Read embedded vertex attributes and specialize vertex pulling by layout. |
| `44e1f49`, `a288f2a` | Pinned FEXCore and documented and scripted the ARM64 build. |
| `52d074d` | Corrected the Metal presenter's sRGB conversion. |
| `188c68b` | Added an archive adapter for streaming game packages. |
| `5f07318` | Fixed GT7 startup on ARM64. |
| `efc1923` | Added GT7 mesh translation and descriptor tracing. Added support for compute cubemap sampling. |
| `48b42f6`, `bd29a0b` | Preserved guest compute LDS bounds and backed oversized workgroups with private GPU memory. |
| `0ccde5e` | Added `.pkg` and `.fpkg` import to the Prospero library, using the existing streaming archive adapter and a local `read-fpkg` helper. |
| `52fa466` | Applied guest component selectors to Metal storage-image writes and fixed sparse store masks. |

KyTx is based on KytyPS5, a fork of Kyty. The
[macOS ARM64 build guide](macos-arm64.md) identifies the fork's FEX dependency
and the referenced toolchain.

## What the World Map trace shows

- With `KYTY_LOG_UNBOUND=1`, a baseline GT7 run reported 26 unique non-null
  image failures. They pointed at the same 1×1 cubemap. All four component
  selectors request zero. The native Metal bindings now use a compatible
  six-slice texture with the guest selectors, with no instances of those 26
  failures in the repeated scene.
- The device reports 32,768 bytes of threadgroup memory. For GT7's first mesh
  pipeline, staging three indices in one word reduced Metal's original
  34,000-byte requirement without reducing the guest allocation. The native
  compute path now runs two kernels with their full 65,536 bytes of LDS per
  group. It can allocate up to 64 MiB for an oversized dispatch, and reports
  allocation failures with the shader hash.
- The display pass with Metal function key `4f4e29ec510fee3b` reads an
  HDR image containing the coastline and a nearly black landscape, then
  samples a 3D color table before drawing the interface. The table's ABGR
  descriptor exposed the alpha written into the physical alpha channel as red.
  The corrected Metal store puts alpha in the physical red channel. The table
  now reads as black at the origin and has independent RGB axes. In final map
  captures, a sampled region went from 44,720 of 73,080 pixels with R ≥ 240 and
  G/B ≤ 16 to zero. The dark HDR input still needs investigation.
- Capturing the same scene before and after `4aa7c29794420db2` shows the pink
  sky and the mountain silhouette in its input at `0x920ec00000`, with most of
  the terrain already black. Its output at `0x9223690000` (color attachment 1)
  is darker still. `a6c1f1e9298a6dcd` then writes that same attachment. The trace
  also shows both passes using the scene depth surface at `0x9210c10000`.
- The dispatch for `ed3ba044f1410a06` writes to a 128×128, six-layer descriptor
  at `0x9296cd0000`, but the old storage allocation was a single 2D texture.
  Several World Map pixel shaders sampled a six-face cube decoded from guest
  backing at the same address. That copy had zero nonzero bytes. The experimental
  `KYTY_METAL_GPU_ARRAYS=1` path retains all six layers and the mip chain
  for the writer and reuses the texture for compatible array and cube readers.
  An M4 test writes the sixth layer through a view of mip level 1 and verifies
  the stored channels. This path still needs a completed World Map run.
- Follow-up runs intermittently stalled in Metal before reaching the cubemap
  writer. The watchdog snapshot repeatedly showed the last prepared indirect
  dispatch as `e9a66de293bc4b3d` with a `(0,1,1)` grid. An independent Metal
  check ran 16 command buffers with the same empty grid followed by a two-group
  dispatch. Each completed with the expected 128 threads across the two dispatches.
  The stalled command-buffer operation remains unidentified.
- The PS5 reference is a final frame. It identifies the matching shoreline and
  shows which map elements are missing. Comparing intermediate outputs against
  PS5 requires a capture of the same render target on that platform.

## Checks in the existing build directory

```sh
build=/path/to/existing/build/kytx
cmake --build "$build" --target resource_tracking_tests resource_materialization_tests \
  scalar_provenance_tests shader_vertex_metadata_tests
ctest --test-dir "$build" --output-on-failure \
  -R '^(resource_tracking|resource_materialization|scalar_provenance|shader_vertex_metadata)$'
bash tools/test_metal_msl.sh "$build"
```

The MSL tests include cubemap sampling, full and sparse storage-image writes,
packed D16 writes, local LDS counters, and a dispatch over eight groups that
verifies the final dword of an oversized LDS
allocation, synchronization across SIMD groups and isolation between
workgroups. Replay GT7 and inspect the terrain, water, buildings and menu
text independently to verify the World Map.

For a bounded render-target trace, set `KYTY_LOG_IMAGE_PRODUCER` to the guest
image address and `KYTY_DUMP_RT_MIN_FLIP` to the start of the 12-frame window.
`KYTY_DUMP_RT_DRAW=address,pixel_shader,0,1` dumps the target before the matching
draw and before the next draw to that target. Set `KYTY_DUMP_RT_DIR` to the
existing scratch directory for the capture. The draw trace reports the color
attachment index alongside its shader hash and write masks.
