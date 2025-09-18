# Data Transfer Tutorial: rsync, scp, rclone (+ GNU Parallel & R)

> Goal: practice fast, reliable file transfers from your **local workstation** to the **SCC server** over SSH.  
> You’ll learn when to use each tool, common flags, and how to run them from **R**.

---

## 0) Setup & Lab Conventions

**You’ll need:**
- An SCC account with SSH access: `user@user-01.al.umces.edu`
- A local folder with sample files: `~/SCCUG/demo/`
- A destination folder on the server: `/SCCUG/test/` (create it or ask an admin)

**Variables (edit for your setup):**
```bash
LOCAL=~/SCCUG/demo
REMOTE=user@user-01.al.umces.edu:/SCCUG/test
REMOTE_HOST=user@server.al.umces.edu
```

**Safety tips**
- Always dry-run first when possible (`--dry-run`).
- Quoted paths avoid spaces issues: `"/path/with spaces/"`.
- Never commit keys/passwords to GitHub.

---

## 1) Tool Selection Cheatsheet

| Tool     | Best for | Pros | Cons |
|----------|----------|------|------|
| **rsync** | Sync folders, incremental updates, robust resumes | Delta transfer (only changes), includes/excludes, verify, deletes, resumes | Slightly more flags to learn |
| **scp**   | One-off copies | Simple, built-in | No delta/partial resume, slower on big trees |
| **rclone**| Cloud/object stores & SSH | Supports S3/Drive/Box/SSH, checksums, parallel transfers | Extra install, more concepts |

**Rule of thumb**
- Need **incremental sync** (only changed files)? → **rsync**  
- Need **just copy these files once**? → **scp**  
- Need **S3/Drive and/or parallel segmented transfers**? → **rclone**

---

## 2) rsync (the workhorse)

**Basic push (local → server)**
```bash
rsync -av --progress "$LOCAL"/ "$REMOTE"/
```
- `-a` archive (perm/times/symlinks), `-v` verbose  
- `--progress` shows per-file progress  
- Trailing slashes mean “copy *contents* of LOCAL into REMOTE”

**Dry-run first**
```bash
rsync -av --dry-run "$LOCAL"/ "$REMOTE"/
```

**Resume robustly & be gentle on the network**
```bash
rsync -av --partial --inplace --progress --bwlimit=50m "$LOCAL"/ "$REMOTE"/
```
- `--partial --inplace` resume large files efficiently  
- `--bwlimit=50m` cap to 50 MB/s (adjust to taste)

**Exclude/Include patterns**
```bash
rsync -av --exclude="*.tmp" --exclude=".DS_Store" "$LOCAL"/ "$REMOTE"/
```

**Delete files on server that were removed locally (mirror)**
```bash
rsync -av --delete "$LOCAL"/ "$REMOTE"/
```
> ⚠️ `--delete` is destructive on the destination. Use `--dry-run` first!

**Integrity check with checksums (slower, but precise)**
```bash
rsync -av --checksum "$LOCAL"/ "$REMOTE"/
```

**Speed tip: compression**
```bash
rsync -avz "$LOCAL"/ "$REMOTE"/
```
- `-z` compress in transit; helps on slow links (hurts on already-compressed data like ZIP/TIF.LZW)

---

## 3) scp (simple copies)

**Copy a whole directory (recursive)**
```bash
scp -r "$LOCAL" "$REMOTE"
```

**Copy a single file**
```bash
scp "$LOCAL/file.tif" "$REMOTE"
```

**Show progress**
```bash
scp -v -r "$LOCAL" "$REMOTE"
```

> scp is great for small, one-off jobs. For **incremental updates** or resuming partially copied big files, prefer **rsync**.

---

## 4) rclone (SSH & clouds, parallel by default)

**Install:** <https://rclone.org/install/>

**Configure a remote (SSH over SFTP)**
```bash
rclone config
# n) New remote -> name: scc
# Storage: sftp
# host: server.al.umces.edu
# user: user
# auth: ssh agent / key file as appropriate
```

**Copy local → SCC (SFTP via rclone)**
```bash
rclone copy "$LOCAL" scc:/project/user/demo -P --transfers=8 --checkers=16
```
- `-P` progress bar  
- `--transfers` concurrent file transfers  
- `--checkers` concurrent listings/checks

**Sync (mirror)**
```bash
rclone sync "$LOCAL" scc:/project/user/demo -P --delete-excluded
```
> ⚠️ Like rsync’s `--delete`, this will make destination match source (deletes extras).

**Cloud examples** (after `rclone config`):
```bash
# To S3:
rclone copy "$LOCAL" s3:mybucket/path -P

# To Google Drive:
rclone copy "$LOCAL" gdrive:my-folder -P
```

---

## 5) Speeding up batches with GNU Parallel

**Install**  
Debian/Ubuntu:
```bash
sudo apt update && sudo apt install parallel -y
```

**Parallelize many `rsync` file pushes**  
(Use **when you have many independent files** and network/IO can keep up.)
```bash
# Make a list of files to send
find "$LOCAL" -type f -name "*.tif" > filelist.txt

# Push 4 files at a time
cat filelist.txt | parallel -j 4 rsync -av --partial --inplace --progress {} "$REMOTE"/
```
- `a` → archive mode = preserve symlinks, permissions, timestamps, group, owner, etc. (basically "copy as-is").
- `v` → verbose = print what’s happening.
- `--partial` → keep partially transferred files if interrupted, so they can be resumed instead of restarting.
- `--inplace` → write changes directly into the destination file instead of creating a temp file. Makes resuming more efficient for big files.
- `--progress` → show per-file transfer progress (bytes sent, % complete, speed, ETA).
- `-j 4` → run 4 transfers concurrently (tune based on bandwidth/disk)  
- `{}` placeholder becomes each file path

**Parallel scp (one file per job)**
```bash
cat filelist.txt | parallel -j 4 scp {} "$REMOTE"/
```

**When to use Parallel**
- Many **medium-sized files** (not one single 200 GB file).  
- Your link & disks can handle concurrent I/O.  
- If the server rate-limits, reduce `-j`.

---

## 6) Calling these tools from R

You can run any shell command from R with `system()` or `system2()`.

**rsync from R**
```r
local  <- "~/SCCUG/demo"
remote <- "user@user-01.al.umces.edu:/SCCUG/test"

cmd <- sprintf('rsync -av --partial --inplace --progress "%s"/ "%s"/', local, remote)
status <- system(cmd)
if (status != 0) stop("rsync failed")
```

**scp from R**
```r
cmd <- sprintf('scp -r "%s" "%s"', local, remote)
system(cmd)
```

**rclone from R**
```r
cmd <- sprintf('rclone copy "%s" scc:/project/user/demo -P --transfers=8 --checkers=16', local)
system(cmd)
```

**GNU Parallel from R**
```r
listfile <- tempfile(fileext = ".txt")
writeLines(list.files(local, pattern="\.tif$", full.names=TRUE), listfile)

cmd <- sprintf('parallel -j 4 rsync -av --partial --inplace --progress {} "%s"/ < "%s"', remote, listfile)
system(cmd)
```

---

## 7) Hands-On Lab

### Step A — Dry run & simple push with rsync
```bash
rsync -av --dry-run "$LOCAL"/ "$REMOTE"/
rsync -av --progress "$LOCAL"/ "$REMOTE"/
```

### Step B — Exclude junk & resume
```bash
rsync -av --exclude=".DS_Store" --exclude="*.tmp" --partial --inplace --progress "$LOCAL"/ "$REMOTE"/
```

### Step C — Mirror (delete extras on server)
```bash
rsync -av --delete --dry-run "$LOCAL"/ "$REMOTE"/
# If looks right:
rsync -av --delete "$LOCAL"/ "$REMOTE"/
```

### Step D — Try scp for a single file
```bash
scp "$LOCAL"/example.tif "$REMOTE"/
```

### Step E — Try rclone (after `rclone config`)
```bash
rclone copy "$LOCAL" scc:/project/user/demo -P --transfers=8 --checkers=16
```

### Step F — Parallelize many medium files
```bash
find "$LOCAL" -type f -name "*.tif" > filelist.txt
cat filelist.txt | parallel -j 4 rsync -av --partial --inplace --progress {} "$REMOTE"/
```

---

## 8) Troubleshooting

- **Permission denied / auth failed**  
  - Test SSH: `ssh $REMOTE_HOST`  
  - Install/copy your SSH key: `ssh-keygen -t ed25519` + `ssh-copy-id $REMOTE_HOST`

- **Slow transfers**  
  - Try `--bwlimit=50m` to be nice on shared links  
  - Omit `-z` for already compressed data  
  - Use `-P` (rsync) for better progress and resume

- **Path/space issues**  
  - Quote paths `"like this"`  
  - Use trailing slashes correctly in rsync

- **Parallel overload**  
  - Lower `-j` in `parallel` (e.g., `-j 2`)  
  - Avoid parallelizing to a slow spinning disk

---

## 9) Quick Reference

**rsync common flags**  
`-a` archive | `-v` verbose | `-z` compress | `--progress` progress bar |  
`--partial --inplace` resume big files | `--delete` mirror |  
`--exclude` / `--include` filters | `--checksum` integrity

**scp**  
`-r` recursive | `-v` verbose

**rclone**  
`copy` (one-way) | `sync` (mirror) | `-P` progress | `--transfers` concurrency

**GNU Parallel**  
`parallel -j 4 <cmd> {}` — run 4 jobs at once; `{}` expands to the input line

---

Happy (and efficient) transferring! 🚀
