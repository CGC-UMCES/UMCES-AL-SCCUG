# Fast, Reliable Bulk Downloads with `wget` (+ GNU Parallel)

This hands-on tutorial shows how to use **`wget`** to download data efficiently
(single files, lists of URLs, whole directory trees), how to **parallelize**
downloads with **GNU Parallel**, and how to **benchmark** parallel vs. serial
runs. It is designed for command-line users working with large research datasets.

## Why use `wget`?
- **Automatable & reproducible**: scriptable commands beat manual browser clicks.
- **Resumable**: `-c/--continue` resumes partial downloads after interruptions.
- **Robust**: retries, backoff, throttling, and logging are built-in.
- **Bulk-friendly**: read URLs from files, crawl directory trees, filter by type.
- **Portable**: available on Linux/macOS (and on Windows via WSL/MSYS2).

> Alternative tools like `curl` or GUI managers are great, but for **repeatable,
> large-scale** transfers, `wget` excels.

---

## 0) Setup

### Install
- **Ubuntu/Debian:** `sudo apt update && sudo apt install -y wget parallel`
- **Fedora/RHEL:** `sudo dnf install wget parallel`
- **macOS (Homebrew):** `brew install wget parallel`
- **Windows:** use **WSL** (Ubuntu) or MSYS2 (then `pacman -S wget parallel`).

### Conventions used below
```bash
# Edit these for your system
OUTDIR=~/downloads              # where files will be saved
URLS=~/urls.txt                 # a file containing one URL per line
JOBS=8                          # number of parallel downloads
```

---

## 1) `wget` basics

### 1.1 Download a single file
```bash
wget -P "$OUTDIR" https://example.com/data/file1.tif
```

### 1.2 Resume an interrupted download
```bash
wget -c -P "$OUTDIR" https://example.com/data/largefile.zip
```

### 1.3 Show a nicer progress bar
```bash
wget --show-progress -P "$OUTDIR" https://example.com/data/file2.tif
```

### 1.4 Retry logic & throttling
```bash
wget -c --tries=20 --wait=1 --random-wait --limit-rate=20m      -P "$OUTDIR" https://example.com/data/file3.tif
```

### 1.5 Read URLs from a file (sequential)
```bash
wget -c --show-progress -P "$OUTDIR" -i "$URLS"
```

### 1.6 Authentication and headers
```bash
wget --user "$USER" --password "$PASS" -P "$OUTDIR" https://example.com/protected/file.nc
wget --header="Authorization: Bearer $TOKEN" -P "$OUTDIR" https://api.example.com/download/123
```

---

## 2) Recursive downloading (mirroring a directory)
```bash
wget -r -np -nH -nd -c --accept=.tif,.tif.gz      --cut-dirs=3 --no-verbose -P "$OUTDIR"      https://data.example.org/pub/projects/imagery/2020/tiles/
```

---

## 3) Logging & reproducibility
```bash
wget -c -i "$URLS" -P "$OUTDIR"      --timeout=30 --tries=15 --retry-connrefused      --show-progress -o "$OUTDIR/wget.log"
```

---

## 4) Parallelizing with GNU Parallel

### 4.1 From a URL list
```bash
cat "$URLS" | parallel -j "$JOBS" wget -c --show-progress -P "$OUTDIR" {}
```

### 4.2 Custom names
```bash
cat "$URLS" | parallel -j "$JOBS" 'wget -c -O "$OUTDIR"/{#}_{/} {}'
```

### 4.3 With rate limiting & retries
```bash
cat "$URLS" | parallel -j "$JOBS"   'wget -c --tries=20 --wait=1 --random-wait --limit-rate=20m         --directory-prefix="$OUTDIR" --append-output="$OUTDIR/wget_parallel.log" {}'
```

---

## 5) Benchmark: parallel vs sequential

### Sequential
```bash
/usr/bin/time -f "Elapsed: %E" wget -c --show-progress -P "$OUTDIR/seq" -i "$URLS"
```

### Parallel
```bash
mkdir -p "$OUTDIR/par"
/usr/bin/time -f "Elapsed: %E"   bash -c 'cat "$URLS" | parallel -j 8 wget -c --show-progress -P "$OUTDIR/par" {}'
```

---

## 6) Quick reference

```bash
# Single file
wget -c --show-progress -P "$OUTDIR" URL

# From list
wget -c --show-progress -P "$OUTDIR" -i "$URLS"

# Parallel N jobs
cat "$URLS" | parallel -j N wget -c --show-progress -P "$OUTDIR" {}

# Recursive, only .tif
wget -r -np -nH -nd -c --accept=.tif -P "$OUTDIR" BASE_URL/
```
