# sysmon: Step-by-Step Build Guide

A terminal tool that shows disk, RAM, and process information using system calls and the `/proc` and `/sys` files they read from. It's user-space code, so no kernel module is needed.

---

## Step 1: Set Up Your Environment

Install a C compiler and the man pages:

```bash
sudo apt install build-essential manpages-dev
```

Create a project folder with a `Makefile` and a `src/` directory.

```make
CC     = gcc
CFLAGS = -Wall -Wextra -O2
SRC    = src/main.c
OUT    = sysmon

$(OUT): $(SRC)
	$(CC) $(CFLAGS) -o $(OUT) $(SRC)
```

---

## Step 2: Build a Skeleton with a Menu

Write `main.c` with a loop that:

- Prints a menu: `1` = RAM, `2` = Disk, `3` = Processes, `q` = Quit
- Reads a choice from stdin
- Exits cleanly on `q`

Get this running first so you can confirm the program builds, runs, and exits.

---

## Step 3: Inspect RAM

Read `/proc/meminfo`:

1. `open()` the file
2. `read()` its contents into a buffer
3. `close()` the file
4. Parse lines like `MemTotal:` and `MemAvailable:`
5. Print used and total memory in MB

Check every return value, and call `perror()` on failure.

---

## Step 4: Inspect Disk

Use `statvfs()` for each mount point:

1. Read mount points from `/proc/mounts` (one mount per line)
2. Call `statvfs()` on each mount point
3. Compute total, used, and free space from `f_frsize`, `f_blocks`, and `f_bfree`
4. Print a table

---

## Step 5: Inspect Processes

1. `opendir("/proc")` and loop with `readdir()`
2. Keep only entries whose names are all digits (these are PIDs)
3. For each PID, read:
   - `/proc/<pid>/stat` for the process name and state
   - `/proc/<pid>/status` for memory usage (`VmRSS`)
4. Print a table of PID, name, state, and memory
5. Use `getpid()` to show the tool's own PID as a check

---

## Step 6: Make It a Terminal Interface

- **Refresh loop:** Use `sleep()` or `nanosleep()` between updates.
- **Clear the screen:** Print the ANSI codes `\033[2J\033[H`.
- **Clean exit on Ctrl+C:** Install a handler with `sigaction()` (or `signal()`) for `SIGINT` so the program restores the terminal and exits.
- **Optional:** Add `ncurses` later for a nicer layout.

---

## Step 7: Handle Errors and Edge Cases

- A `/proc/<pid>` entry can disappear while you read it, because the process exited. Treat a failed `open` as "skip this process," not as a fatal error.
- Run the tool as a normal user first, then with `sudo`, and compare what each run can see.

---

## Step 8: Test and Extend

**Verify your numbers** against these tools:

- `free -m` for RAM
- `df -h` for disk
- `ps aux` for processes

**Ideas to add next:**

- Sort processes by memory use
- Show CPU usage, computed from two samples of `/proc/stat`
- Add keyboard controls to switch between views

---

## Syscalls to Learn Along the Way

`open`, `read`, `close`, `opendir`, `readdir`, `statvfs`, `getpid`, `sigaction`, `nanosleep`

Look each one up with `man 2` or `man 3` as you use it.