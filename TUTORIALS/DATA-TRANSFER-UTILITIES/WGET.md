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
- **macOS (Homebrew):** `brew install wget parallel`
- **Windows:** use **WSL** (Ubuntu) (wsl --install in Powershell).

### Conventions used below
```bash
# Edit these for your system
OUTDIR=~/SCCUG                                  # where files will be saved
URLS=~/SCCUG/wget_lidar_tutorial_urls.txt       # a file containing one URL per line
JOBS=8                                          # number of parallel downloads
```

---

## 1) `wget` basics (with flags explained)

### 1.1 Download a single file
```bash
wget -P "$OUTDIR" https://example.com/data/file1.tif
```
- `-P DIR` → save file into directory `DIR` (creates it if needed).

### 1.2 Resume an interrupted download
```bash
wget -c -P "$OUTDIR" https://example.com/data/largefile.zip
```
- `-c` or `--continue` → resume partial files if download was interrupted.

### 1.3 Show a nicer progress bar
```bash
wget --show-progress -P "$OUTDIR" https://example.com/data/file2.tif
```
- `--show-progress` → human-readable progress bar per file.

### 1.4 Retry logic & throttling
```bash
wget -c --tries=20 --wait=1 --random-wait --limit-rate=20m -P "$OUTDIR" https://example.com/data/file3.tif
```
- `--tries=N` → attempt N times.  
- `--wait=SECONDS` → wait between retries.  
- `--random-wait` → add jitter so you don’t hammer the server.  
- `--limit-rate=20m` → max 20 MB/s per download.  

### 1.5 Read URLs from a file (sequential)
```bash
wget -c --show-progress -P "$OUTDIR" -i "$URLS"
```
- `-i FILE` → read list of URLs from a file.  

### 1.6 Authentication and headers
```bash
wget --user "$USER" --password "$PASS" -P "$OUTDIR" https://example.com/protected/file.nc
wget --header="Authorization: Bearer $TOKEN" -P "$OUTDIR" https://api.example.com/download/123
```

---

## 2) Recursive downloading (mirroring a directory)

```bash
wget -r -np -nH -nd -c --accept=.tif,.tif.gz --cut-dirs=3 --no-verbose -P "$OUTDIR" https://data.example.org/pub/projects/imagery/2020/tiles/
```

### Explanation of flags:
- `-r` → recursive download (follow links to fetch entire directory trees).  
- `-np` → *no parent*, don’t ascend into parent directories.  
- `-nH` → don’t create a top-level folder named after the hostname.  
- `-nd` → don’t recreate directory structure, save files flat in `OUTDIR`.  
- `-c` → resume partial files.  
- `--accept=LIST` → only grab files with listed extensions.  
- `--cut-dirs=N` → skip the first N directory components when saving.  
- `--no-verbose` → less chatty output, just summaries.  
- `-P OUTDIR` → set output directory.  

Example:  
Remote path: `/pub/projects/imagery/2020/tiles/file01.tif`  
With `--cut-dirs=3`, `pub/projects/imagery` is dropped → locally saved as `2020/tiles/file01.tif` (or just `file01.tif` if combined with `-nd`).

---

## 3) Logging & reproducibility

```bash
wget -c -i "$URLS" -P "$OUTDIR" --timeout=30 --tries=15 --retry-connrefused --show-progress -o "$OUTDIR/wget.log"
```
- `--timeout=SECONDS` → network timeout.  
- `--retry-connrefused` → retry even on refused connections.  
- `-o FILE` → log output to file (use `-a` to append).  

---

## 4) Parallelizing with GNU Parallel 

### 4.1 From a URL list
```bash
cat "$URLS" | parallel -j "$JOBS" wget -c --show-progress -P "$OUTDIR" {}
```
- `parallel` runs jobs concurrently.  
- `-j JOBS` → run JOBS in parallel.  
- `{}` → replaced by each URL line.  

### 4.2 Custom names
```bash
cat "$URLS" | parallel -j "$JOBS" 'wget -c -O "$OUTDIR"/{#}_{/} {}'
```
- `{#}` → job index.  
- `{/}` → basename of file.  

### 4.3 With rate limiting & retries
```bash
cat "$URLS" | parallel -j "$JOBS" 'wget -c --tries=20 --wait=1 --random-wait --limit-rate=20m --directory-prefix="$OUTDIR" --append-output="$OUTDIR/wget_parallel.log" {}'
```
- `--append-output=FILE` → append logs to a file.  

---

## 5) Benchmark: parallel vs sequential

### Sequential
```bash
time wget -c --show-progress -P "$OUTDIR/seq" -i "$URLS"
```

### Parallel
```bash
# It will make the folder if it doesn't exist
time cat "$URLS" | parallel -j "$JOBS" wget -c --show-progress -P "$OUTDIR/par" {}

# Cleaner way to write the same thing with ::::
time parallel -j "$JOBS" wget -c --show-progress -P "$OUTDIR/par" {} :::: "$URLS"
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

---

## 7) Suggested exercises

1) Download a single file with `wget -P`. Cancel midway, then resume with `-c`.  
2) Download from a `urls.txt` file sequentially and note the time.  
3) Repeat with `parallel -j 4`. Compare times.  
4) Try recursive mirroring with `-r -np -nd`.  
5) Add filters (`--accept`, `--reject`) and log files.  

---

### Final tips
- Start with **sequential** to test URLs, then scale up with `parallel`.  
- Always keep **resume (`-c`)** enabled for large files.  
- Document your exact commands in a README for reproducibility.  
