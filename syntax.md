# Syscall Cheat Sheet for sysmon

Each entry shows the call, what it does, and where you'll use it. Check `man 2 <name>` (or `man 3` for libc wrappers like `opendir`) for full details.

## File I/O

| Call | Purpose | Used for |
|------|---------|----------|
| `open(path, flags)` | Open a file, returns a file descriptor (or -1) | Reading `/proc/meminfo`, `/proc/<pid>/stat` |
| `read(fd, buf, n)` | Read up to `n` bytes into `buf`, returns bytes read | Pulling file contents into a buffer |
| `close(fd)` | Release a file descriptor | After every `open` |
| `fopen` / `fgets` *(libc)* | Buffered file read, line by line | Easier parsing of `/proc` text files |

## Directory Traversal

| Call | Purpose | Used for |
|------|---------|----------|
| `opendir(path)` | Open a directory stream | Opening `/proc` |
| `readdir(dir)` | Return the next entry (or NULL) | Listing PIDs and mount points |
| `closedir(dir)` | Close the directory stream | Cleanup |

## Filesystem Stats

| Call | Purpose | Used for |
|------|---------|----------|
| `statvfs(path, &buf)` | Get block size, total/free blocks for a mount | Disk usage per mount point |
| `statvfs` fields | `f_frsize`, `f_blocks`, `f_bfree`, `f_bavail` | Computing total, used, and free space |

## Process Info

| Call | Purpose | Used for |
|------|---------|----------|
| `getpid()` | Get this process's PID | Showing the tool's own PID |
| `getppid()` | Get the parent's PID | Optional process tree info |
| `getuid()` | Get the real user ID | Showing who is running the tool |
| `kill(pid, 0)` | Check if a PID exists without signaling it | Verifying a process is still alive |

## Signals (Clean Exit on Ctrl+C)

| Call | Purpose | Used for |
|------|---------|----------|
| `sigaction(sig, &act, NULL)` | Install a signal handler | Catching `SIGINT` (Ctrl+C) |
| `signal(sig, handler)` | Simpler signal handler (less portable) | Quick prototypes |
| `SIGINT`, `SIGTERM` | Signals to catch | Restore terminal and exit cleanly |

## Timing

| Call | Purpose | Used for |
|------|---------|----------|
| `nanosleep(&ts, NULL)` | Sleep for a precise interval | Refresh delay between updates |
| `sleep(n)` | Sleep for `n` seconds | Simple refresh loop |
| `clock_gettime(CLOCK_MONOTONIC, &ts)` | Read a monotonic clock | Computing CPU % over time |

## Terminal Control

| Call | Purpose | Used for |
|------|---------|----------|
| `write(STDOUT_FILENO, ...)` | Write raw bytes to the terminal | Printing ANSI escape codes |
| `isatty(fd)` | Check if a fd is a terminal | Refusing to run when piped |
| `tcgetattr` / `tcsetattr` | Read and set terminal modes | Optional raw-key input for menus |

## Memory

| Call | Purpose | Used for |
|------|---------|----------|
| `malloc` / `free` *(libc)* | Allocate and free heap memory | Storing process lists |
| `brk` / `mmap` *(syscalls behind malloc)* | Ask the kernel for memory | Only if you go lower level |

## Useful `/proc` Files (read with `open`/`read`)

| File | Contains | Step |
|------|----------|------|
| `/proc/meminfo` | `MemTotal`, `MemAvailable`, etc. | RAM |
| `/proc/mounts` | One mount point per line | Disk |
| `/proc/<pid>/stat` | Name, state, and CPU time fields | Processes |
| `/proc/<pid>/status` | `VmRSS` (memory), `Name`, `State` | Processes |
| `/proc/stat` | Total CPU time counters | CPU % |
| `/proc/uptime` | Seconds since boot | Header info |

## Quick Tips

- **Always check return values.** On failure, most calls return `-1` and set `errno`. Use `perror("open")` to print the reason.
- **Expect races.** A PID can vanish between `readdir` and `open`. Skip it rather than exiting.
- **Close what you open.** Every successful `open` needs a matching `close`, and every `opendir` needs a `closedir`.