# xmlbench — Rust XML parser, evaluation build

`xmlbench` is a standalone CLI that parses XML into a real tree and benchmarks
the parse hot path against your own files. It is the evaluation build of a Rust
XML core that outperforms JavaScript XML parsers by an order of magnitude on
real documents (measured up to **32–41×** vs fast-xml-parser in max modes).

This folder ships prebuilt binaries, so nothing to
set up.

## Quick start (macOS Apple Silicon)

```bash
git clone https://github.com/tawfeks/next-gen-xml-parser.git && cd next-gen-xml-parser
curl -O https://raw.githubusercontent.com/NaturalIntelligence/fast-xml-parser/bb2ec6ce4bfc3f62d621df6e6bf2b3152965175a/spec/assets/large.xml \
     -O https://raw.githubusercontent.com/NaturalIntelligence/fast-xml-parser/bb2ec6ce4bfc3f62d621df6e6bf2b3152965175a/spec/assets/midsize.xml
./xmlbench-aarch64-apple-darwin bench large.xml
```

Downloads the two public benchmark fixtures (from the fast-xml-parser repo)
next to the binary, then benchmarks `large.xml` (~97 MiB).

- macOS Intel: use `xmlbench-x86_64-apple-darwin`
- Windows x64: `.\xmlbench-x86_64-pc-windows-gnu.exe bench large.xml`

## Commands

```bash
xmlbench demo                 # parse a built-in sample; see what a parse produces
xmlbench parse <file.xml>     # parse your file; stats + tree preview
xmlbench bench <file.xml>     # benchmark the parse hot path on your file
```

## Options

| Option | Applies to | Values (default first) |
|---|---|---|
| `--engine <name>` | `bench` | `fast` \| `core` \| `par` \| `fxp5-shape` |
| `--iterations <n>` | `bench` | timed runs (10) |
| `--warmup <n>` | `bench` | untimed warmups (3) |
| `--threads <n>` | `bench --engine par` | worker threads (auto = all cores) |
| `--json <out.json>` | `parse` | write full parsed tree as JSON |
| `--preview <n\|full>` | `parse` | tree-preview line cap (50; `0` = stats only) |
| `--max-depth <n>` | `parse` | preview depth limit (4) |

`--engine par` splits the file across all cores (throughput ceiling probe, not
apples-to-apples with single-threaded JS parsers). `--engine fxp5-shape` emits
the default fast-xml-parser v5 object shape as JSON.

## Measured results (MacBook Pro, M3 Pro, 12 cores)

`xmlbench bench large.xml` (~97 MiB) — engine `fast`, 10 iterations, 3 warmups:

| Metric | Value |
|---|---|
| Throughput | **1226.4 MiB/s** |
| Req/s (median) | 12.6 |
| Parse time (median) | 79.2 ms |
| Fastest / slowest | 78.5 / 82.7 ms |
| Mean | 79.7 ms |
| Cold parse | 92.1 ms |
| Load (args + file read) | 18.0 ms |
| Peak RSS | 1217 MiB |

Structure (independently verified): 2,764,801 elements · 483,840 attributes ·
44,858,872 text bytes · max depth 5.

For context, on the same machine and file, fast-xml-parser v5 parses at
~25 MiB/s and v6 at ~40 MiB/s.

## Intended use

This build exists for **evaluation only**: run `demo`, `parse` your own XML,
and `bench` it on your hardware. Redistribution, reverse engineering, and
production use are not permitted. See [LICENSE.md](LICENSE.md). For full
access, integration (streaming, full spec coverage), or
commercial licensing, contact the author.