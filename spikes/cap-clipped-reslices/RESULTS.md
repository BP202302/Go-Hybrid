# Checker-planned capacity clipping — implementation of #1927

This implements the existing proposal by the author of
https://github.com/limitlessprojects/Go-Hybrid/issues/1927. It is not a new
finding, a new Wildcard proposal, or a claim to that proposal's award.

## Pre-registered protocol (before implementation measurements)

Baseline: upstream main `1f0ff978f4deff78c77bca96db0e3e46f32de71b`.
Hypothesis: wrapping a checked two-index slice in an inlined generic helper
can enforce `cap == len`, with no duplicate evaluation, while keeping the
slice workload within 5% of main. Arrays must remain views of the original
storage, not copies. Strings and unresolved bases must remain two-index.
Explicit three-index slices retain their requested capacity.

Functional gates: failing-before/passing-after CLI regressions; checker
plan coverage for slice/array/pointer-to-array, aliases, fields, nested
slices and closures; string/unknown controls; parser and negative checks
for max bounds; evaluation order including `?`; workspace tests, registered
checker corpus, and full differential loop. Do not weaken existing tests.

Performance protocol, fixed before running either variant:

- Build the same `workload.hy` using the main and patched compilers in
  package mode; compile each generated package with the same Go 1.26.9.
- Run `BenchmarkSlice` in `slice_test.go`: dynamic two-index, omitted low,
  omitted high, omitted both, array, pointer-to-array and nested views.
  Each iteration consumes an element and publishes the resulting header,
  so the slice operation cannot be discarded. No append or allocation in
  the measured loop. The array benchmark uses an addressable array field.
- Use native ARM64 toolchains on this host; record actual versions, CPU,
  platform, generated Go, compiler SHAs, commands and unedited outputs.
  No cross-architecture binaries or toolchain-drift override.
- One warmup per variant (excluded). Eleven paired samples, alternating
  main/patched order, `GOMAXPROCS=1`, `-cpu=1`, `-benchtime=500ms`,
  `-count=1`, `-benchmem`. Only one heavy job at a time.
- For each named workload, divide the median patched ns/op by the median
  main ns/op. Every ratio must be <= 1.05; report all samples, medians,
  ratios and allocations. Any failure is a failed gate, not rounded down.
  If an implementation is optimized after a failure, retain failed results
  and rerun the entire protocol as a separately named attempt.
- Timing covers emitted slice workloads, not compiler throughput, appends,
  the FFI bridge, a universal safety proof, or every hardware platform.

Reproduction (from repository root, pinned tools on PATH):

```
python3 spikes/cap-clipped-reslices/bench.py \
  --baseline /path/to/main/hyb --patched /path/to/patched/hyb \
  --out /tmp/cap-clip-bench
```

## Results

Pending. This section will be replaced only with executed measurements.
