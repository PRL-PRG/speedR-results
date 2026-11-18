# speedR-results

Benchmark results for GNU R, produced by
[speedR](https://github.com/PRL-PRG/speedR) and shown on its dashboard.

> **Status: empty.** No measurements have been published yet.

Everything here is written by speedR's pipeline (the scheduler's ingest job);
please do not edit it by hand. Questions and problems go to the
[speedR issue tracker](https://github.com/PRL-PRG/speedR/issues).

## Licence

The results are dedicated to the public domain under
[CC0 1.0](LICENSE): use them for anything, no permission needed. Citing speedR
is appreciated but not required. The benchmark programs are **not** part of
this repository; they keep their own licences in
[r-benchmarks](https://github.com/PRL-PRG/r-benchmarks).

## Layout

```
runs/<machine>/<impl>/<version>/<config>/<run_id>/
    meta.json            what was run, how, and where (schema below)
    samples.csv.gz       one row per benchmark process
    profile.csv.gz       category shares per benchmark (when profiled)
    symbols.csv.gz       top innermost-classified and unclassified symbols
    sentinel.json        the session's sentinel benchmark result
index/                   tables derived from runs/, read by the dashboard
```

- `machine` is a concrete box (e.g. `prl-bench-x64-1`). **Times are only
  comparable within one machine.**
- `impl` is the implementation (`gnur`), `version` a release (`4.5.2`) or
  `devel`, `config` a build configuration (`default`, `no-bytecode`, ...).
- `run_id` is the UTC start time plus a nonce, e.g. `20261111T093000Z-3f9a1c`.

`meta.json` follows the JSON Schema in speedR,
[`speedr/schema/meta.schema.json`](https://github.com/PRL-PRG/speedR/blob/main/speedr/schema/meta.schema.json).
It records the build (source, flags, toolchain image digest), the suite and
`bench` commits, the run design, the host (kernel, command line, boot id, wake
method, doctor report) and the sentinel result.

## Index tables

| table | one row per | columns |
|---|---|---|
| `runs.csv` | run | run, machine, impl, version, kind (`release`/`devel`), config, date, svn_rev, suspect, sentinel_drift, n_ok, n_total |
| `machines.csv` | machine | machine, os, arch, cpu, kernel, toolchain, profiler, profiles (`validated`/`experimental`), wake, first_version |
| `benchmarks.csv` | benchmark | id (`suite/name`), suite, name, min_version |
| `events.csv` | machine change | machine, date, event (kernel, firmware, toolchain updates) |
| `unsupported.csv` | benchmark that cannot run | machine, impl, version, config, benchmark, reason |
| `samples.csv.gz` | benchmark process | run, benchmark, process, time_ms (median iteration), gc_ms, max_rss_mb |
| `profiles.csv.gz` | benchmark × category | run, benchmark, category, share (of perf samples in `execute()`) |

A run marked `suspect` failed its sentinel check (the machine was more than 3%
off its recent median); its results are published but excluded from change
detection.

Check a checkout with speedR: `uv run speedr check-results path/to/speedR-results`.
