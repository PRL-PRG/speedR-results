# speedR-results

Benchmark results for GNU R, produced by
[speedR](https://github.com/PRL-PRG/speedR) and shown on its dashboard.

The results are on the [`results`](https://github.com/PRL-PRG/speedR-results/tree/results)
branch; `main` holds only this description. Every push to `results` renders the
dashboard, <https://prl-prg.github.io/speedR-results/>.

Everything there is written by speedR's pipeline (`speedr-publish` on the
cluster's controller); please do not edit it by hand. Questions and problems go
to the [speedR issue tracker](https://github.com/PRL-PRG/speedR/issues).

## Licence

The results are dedicated to the public domain under
[CC0 1.0](LICENSE): use them for anything, no permission needed. Citing speedR
is appreciated but not required. The benchmark programs are **not** part of
this repository; they keep their own licences in
[r-benchmarks](https://github.com/PRL-PRG/r-benchmarks).

## Layout

```
<methodology>/                   one tree per methodology version (e.g. 0.2)
    <impl>/<machine>/<version>/<config>/<run_id>/
        meta.json                what was run, how, and where (schema below)
        samples.csv              one row per benchmark process
        iterations.csv           one row per iteration of each process
    index/                       tables derived from the runs, read by the dashboard
```

Results measured under different methodology versions are never compared, so
each version has its own tree and index.

Files are stored uncompressed: git compresses them itself, and storing them
compressed would stop it from storing only the differences between versions.

- `impl` is the R implementation: `gnur` (GNU R) now, others later.
- `machine` is a set of identical boxes (e.g. `kabylake`, the Intel Core
  i7-7700K ones); `meta.json` names the box that ran it. **Times are only
  comparable within one machine.**
- `version` is a release (`4.5.2`) or `devel`, `config` a build configuration
  (`default`, `no-bytecode`, ...).
- `run_id` is the UTC start time plus a nonce, e.g. `20261111T093000Z-3f9a1c`.

`meta.json` follows the JSON Schema in speedR,
[`speedr/schema/meta.schema.json`](https://github.com/PRL-PRG/speedR/blob/main/speedr/schema/meta.schema.json).
It records the build (source, settings, toolchain image digest), the suite,
`bench` and speedR commits, the run design, the host (box, kernel, command
line, boot id, the box's native probe before the job) and the sentinel result.

## Index tables

| table | one row per | columns |
|---|---|---|
| `runs.csv` | run | run, methodology, machine, impl, version, kind (`release`/`devel`), config, date (of the R source), svn_rev, suspect, sentinel_drift, n_ok, n_total, node, started, sentinel_status, probe_status, thermal_suspect |
| `machines.csv` | machine | machine, os, arch, cpu, kernel, toolchain, profiler, profiles (`validated`/`experimental`), wake, first_version |
| `benchmarks.csv` | benchmark | id (`suite/name`), suite, name, min_version |
| `events.csv` | machine change | machine, date, event (kernel, firmware, toolchain updates) |
| `unsupported.csv` | benchmark that cannot run | machine, impl, version, config, benchmark, reason |
| `samples.csv` | benchmark process | run, benchmark, process, time_ms (median measured iteration), gc_ms, max_rss_mb, cpu_temp_max, cpu_throttle, suspect (the CPU throttled or ran near its limit) |
| `profiles.csv` | benchmark × category | run, benchmark, category, share (of perf samples in `execute()`) |

A run is marked `suspect` when its box performed off its machine's reference
(the sentinel, or the node's probe, beyond the suspect threshold) or the CPU
ran hot or throttled during one of its processes. Its results are published
but excluded from change detection. A box that is far off does not run at all.

Check a checkout with speedR: `uv run speedr check-results path/to/speedR-results`.
