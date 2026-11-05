---
layout: default
title: Go-Turso Benchmark — Dokumentasi
---

# Go-Turso Benchmark — Dokumentasi

Dokumentasi publik untuk [rezapace/Go-Turso](https://github.com/rezapace/Go-Turso) — CLI benchmark CRUD dan concurrent `INSERT` dengan Turso MVCC (`BEGIN CONCURRENT`).

## Daftar Dokumen

1. [Technical Plan](Docs/Technical-Plan.html) — rencana teknis, keputusan arsitektur, batasan MVCC, dan quality gate.
2. [Test Results](Docs/Test-Results.html) — hasil smoke test, race detector, build ARM64, dan benchmark.
3. [Readme Tambahan](Docs/Readme.html) — contoh program Go lengkap dan penjelasan cara kerja MVCC.

## Ringkasan Proyek

| Aspek | Keterangan |
|---|---|
| Bahasa | Go 1.23+ |
| Database | Turso lokal (LibSQL) via driver `turso-go` |
| Mode | `crud` dan `benchmark` |
| Metrik | Throughput, average latency, P50/P95/P99 |
| Dokumen teknis | 3 file Markdown di folder `Docs/` |

## Tautan Terkait

- Repository utama: <https://github.com/rezapace/Go-Turso>
- README utama: <https://github.com/rezapace/Go-Turso#readme>
- Release v1.0.0: <https://github.com/rezapace/Go-Turso/releases/tag/v1.0.0>
