# Hasil Pengujian Implementasi

**Waktu pengujian (UTC):** 2026-11-05T03:52:14Z  
**Environment:** Linux ARM64 (`uname -m=aarch64`), Go `1.23.12`  
**Target build:** `GOOS=linux GOARCH=arm64`

## Ringkasan

| Pemeriksaan | Perintah | Status | Hasil |
|---|---|---|---|
| Unit test | `make test` | Lulus | Seluruh test berhasil. |
| Static analysis | `make vet` | Lulus | Tidak ada temuan. |
| Build native ARM64 | `make build` | Lulus | Binary `Production/turso-benchmark` dibuat sebagai ARM64. |
| CRUD smoke test | `./Production/turso-benchmark -mode crud ...` | Lulus | Create, read, update, dan delete verification selesai. |
| Race detector | `make race` | Lulus | Seluruh test lulus dengan race detector. |
| Benchmark MVCC | command benchmark 2×10 | Gagal tervalidasi | Library Turso yang dipin tidak mengenali journal mode `mvcc`. |

## Perbaikan Arsitektur Target

Kegagalan awal bukan karena Turso tidak mendukung ARM64. Host adalah ARM64, tetapi Go sebelumnya membangun binary target `amd64`; library Turso x86-64 lalu dijalankan lewat QEMU dan memerlukan dynamic loader x86-64 yang tidak ada.

`Makefile` sekarang memetakan arsitektur host:

```make
GOARCH ?= $(shell uname -m | sed -e 's/^x86_64$$/amd64/' -e 's/^aarch64$$/arm64/')
```

Pada host ini, seluruh command Make berjalan sebagai `GOOS=linux GOARCH=arm64`. Driver `github.com/tursodatabase/turso-go v0.2.0` kemudian memuat `linux_arm64/libturso_go.so` yang embedded di module.

## Detail Hasil

### Unit test, vet, build, dan race detector

```text
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go test ./...
ok   github.com/rezasaputra/go-turso-benchmark/internal/benchmark
ok   github.com/rezasaputra/go-turso-benchmark/internal/config

GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go vet ./...

GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build -o Production/turso-benchmark ./cmd/turso-benchmark

GOOS=linux GOARCH=arm64 CGO_ENABLED=1 go test -race ./...
ok   github.com/rezasaputra/go-turso-benchmark/internal/benchmark
ok   github.com/rezasaputra/go-turso-benchmark/internal/config
```

GCC native terdeteksi sebagai `aarch64-linux-gnu`; karena target Go juga ARM64, `make race` sekarang lulus.

### CRUD smoke test — lulus

```text
Database: data/smoke-crud-make.db
CREATE: ID=1
READ: ID=1 | Status=PASSED
UPDATE: ID=1 | Status=FAILED
VERIFY DELETE: ID=1 tidak ditemukan
```

### Benchmark MVCC — belum didukung oleh library yang dipin

Perintah:

```bash
./Production/turso-benchmark \
  -mode benchmark \
  -db data/smoke-benchmark-make.db \
  -workers 2 \
  -operations 10 \
  -retries 3 \
  -timeout 30s
```

Output:

```text
mengaktifkan MVCC: Parse error: Unknown journal mode: mvcc
```

Jadi masalah tersisa bukan arsitektur ARM maupun compiler. Native library yang dibawa `github.com/tursodatabase/turso-go v0.2.0` tidak menyediakan `PRAGMA journal_mode = 'mvcc'`. Karena setup MVCC gagal sebelum worker dijalankan, tidak ada hasil throughput atau latency yang dilaporkan.

## Tindak Lanjut MVCC

Pilih salah satu jalur berikut:

1. **Benchmark MVCC:** gunakan dan pin binding/native Turso yang secara eksplisit mendukung `PRAGMA journal_mode=mvcc` serta `BEGIN CONCURRENT`; lalu jalankan kembali smoke benchmark ini.
2. **Benchmark SQLite/Turso tanpa MVCC:** ubah aplikasi agar memakai journal mode yang didukung (misalnya WAL) dan transaksi standar `BEGIN`/`COMMIT`; hasilnya harus diberi label non-MVCC dan tidak boleh dibandingkan langsung dengan benchmark MVCC.

Sampai driver yang kompatibel tervalidasi, mode benchmark sengaja fail-fast agar tidak mengklaim benchmark MVCC yang sebenarnya tidak berjalan.
