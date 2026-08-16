# Linux Fundamentals & Commands — DevOps Reference

A practical guide to Linux: how it works under the hood, then the commands you'll actually use day to day when deploying applications, managing servers, and working with tools like Docker and Terraform.

---

## Part 1: Linux Fundamentals

### What is Linux, really?

Linux is the **kernel** — the core program that talks directly to hardware (CPU, memory, disk, network) and manages resources for everything else running on the machine. What people call "Linux" (Ubuntu, Amazon Linux, CentOS, Debian) is actually a **distribution**: the Linux kernel plus a package manager, shell, system utilities, and default configuration, bundled by an organization.

Every distro shares the same kernel fundamentals — this is why the commands in this guide work almost identically on Ubuntu, Amazon Linux, CentOS, or the base image inside a Docker container.

### The core mental model

Everything in Linux is built around a few simple ideas:

**1. Everything is a file**
Devices, processes, sockets, and configuration are all represented as files somewhere in the filesystem — not just documents. `/dev/sda` is your disk. `/proc/1234` is a running process. This is why so many Linux tools just read and write files rather than needing special APIs.

**2. A single-rooted filesystem tree**
There's no `C:\` or `D:\` like Windows. Everything hangs off one root, `/`. External drives, network shares, and other partitions get *mounted* into this tree at some folder, rather than getting their own letter.

```
/               ← root, everything starts here
├── bin, usr/bin      ← executable programs
├── etc               ← configuration files
├── home               ← user home directories
├── var                ← logs, variable data (var/log is huge in ops work)
├── tmp                ← temporary files, cleared on reboot
├── opt                 ← optional/third-party software
├── proc                ← virtual filesystem representing running processes
└── root                ← the root user's home directory (not the same as /)
```

**3. Users, groups, and permissions**
Every file has an owner (user) and a group, and permissions define who can read/write/execute it. The `root` user can do anything — this is why so much ops work involves `sudo`.

**4. Processes**
Every running program is a process with a unique ID (PID). Process #1 (`init`/`systemd`) is the ancestor of every other process. When you deploy an app, you're essentially starting a new process (or, in Docker, a process that *is* the container).

**5. The shell**
The shell (usually `bash` or `zsh`) is a program that reads the commands you type and asks the kernel to execute them. Everything in this guide is a shell command.

### Why this matters for deployment/Terraform work

- **Docker containers are just Linux processes** with restricted views of the filesystem, network, and process list — so debugging a container is really just Linux debugging (`docker exec` drops you into a shell where all these commands apply).
- **Terraform, when provisioning EC2 instances,** often needs `user_data` scripts written in shell to bootstrap the server (install packages, start services) — so shell/Linux command knowledge directly feeds into IaC work.
- **SSH-ing into a server to debug a failed deployment** is 90% Linux commands: checking logs, checking disk space, checking if a process/port is in use, checking permissions.

---

## Part 2: File & Directory Navigation

| Command | Use |
|---|---|
| `pwd` | Print current working directory |
| `ls` | List files in current directory |
| `ls -la` | List all files (including hidden `.` files) with details (permissions, size, owner) |
| `cd /path/to/dir` | Change directory |
| `cd ..` | Move up one directory |
| `cd ~` or `cd` | Go to home directory |
| `cd -` | Go to the previous directory |
| `mkdir myfolder` | Create a directory |
| `mkdir -p a/b/c` | Create nested directories in one go |
| `rmdir myfolder` | Remove an empty directory |
| `tree` | Show directory structure as a tree (may need `apt install tree`) |
| `find / -name "*.log"` | Search the filesystem for files matching a pattern |
| `find . -type f -mtime -1` | Find files modified in the last day (useful for debugging recent changes) |
| `locate filename` | Fast file search using a prebuilt index (needs `updatedb` run first) |

---

## Part 3: File Operations

| Command | Use |
|---|---|
| `touch file.txt` | Create an empty file / update its timestamp |
| `cp file.txt backup.txt` | Copy a file |
| `cp -r dir1 dir2` | Copy a directory recursively |
| `mv file.txt newname.txt` | Move or rename a file |
| `rm file.txt` | Delete a file |
| `rm -rf directory/` | Delete a directory and everything inside it (careful — no undo) |
| `cat file.txt` | Print entire file contents |
| `less file.txt` | View a file page by page (great for large logs — `q` to quit) |
| `head -n 20 file.txt` | Show the first 20 lines |
| `tail -n 20 file.txt` | Show the last 20 lines |
| `tail -f app.log` | Follow a log file in real time — **used constantly when debugging running apps** |
| `wc -l file.txt` | Count lines in a file |
| `diff file1 file2` | Show differences between two files |
| `ln -s /path/target /path/link` | Create a symbolic link (shortcut) |

---

## Part 4: Viewing & Editing Files

| Command | Use |
|---|---|
| `nano file.txt` | Simple terminal text editor — beginner-friendly |
| `vim file.txt` | Powerful terminal editor — steep learning curve but extremely common on servers (`i` to insert, `Esc` then `:wq` to save & quit, `:q!` to quit without saving) |
| `cat > file.txt` | Quickly write content to a file from the terminal (`Ctrl+D` to save) |
| `echo "text" > file.txt` | Overwrite a file with text |
| `echo "text" >> file.txt` | Append text to a file |

---

## Part 5: Permissions & Ownership

Permissions in Linux are shown as something like `-rwxr-xr--`:
- First character: file type (`-` file, `d` directory, `l` symlink)
- Next 3: owner permissions (read/write/execute)
- Next 3: group permissions
- Last 3: everyone else's permissions

| Command | Use |
|---|---|
| `chmod 755 script.sh` | Set permissions numerically (owner: rwx, group: r-x, others: r-x) |
| `chmod +x script.sh` | Make a file executable (very common for deployment scripts) |
| `chown user:group file.txt` | Change file owner and group |
| `chown -R user:group dir/` | Change ownership recursively |
| `sudo command` | Run a command as root/superuser |
| `su - username` | Switch to another user |
| `whoami` | Show current logged-in user |
| `id` | Show current user's UID, GID, and group memberships |

**Common permission numbers you'll actually use:**
- `755` — standard for executable scripts (owner can write, everyone can read/run)
- `644` — standard for regular files (owner can write, everyone can read)
- `600` — private files like SSH keys (owner only)

---

## Part 6: Process Management

| Command | Use |
|---|---|
| `ps aux` | List all running processes |
| `ps aux \| grep node` | Find a specific running process (e.g., is my app still running?) |
| `top` | Live view of running processes, CPU/memory usage |
| `htop` | Improved, more readable version of `top` (may need install) |
| `kill 1234` | Terminate a process by PID (graceful, `SIGTERM`) |
| `kill -9 1234` | Force-kill a process (`SIGKILL`, no cleanup) |
| `killall node` | Kill all processes matching a name |
| `bg` / `fg` | Move a process to background/foreground |
| `nohup command &` | Run a command that keeps running after you log out |
| `jobs` | List background jobs in the current shell session |

---

## Part 7: Disk & System Info

| Command | Use |
|---|---|
| `df -h` | Show disk space usage per mounted filesystem (human-readable) — **first thing to check if a deploy fails with "no space left"** |
| `du -sh *` | Show size of files/folders in current directory |
| `du -sh /var/log` | Check how big a specific folder is (logs often eat disk space) |
| `free -h` | Show memory (RAM) usage |
| `uptime` | Show how long the system has been running and load average |
| `uname -a` | Show kernel/system info |
| `lscpu` | Show CPU details |
| `hostname` | Show the machine's hostname |
| `hostnamectl` | Show/set detailed hostname and OS info |

---

## Part 8: Networking

| Command | Use |
|---|---|
| `ping google.com` | Test network connectivity |
| `curl https://api.example.com` | Make an HTTP request from the terminal — used constantly to test APIs/health checks |
| `curl -I https://example.com` | Fetch just the response headers (quick status check) |
| `wget https://example.com/file.tar.gz` | Download a file |
| `ss -tulpn` | Show listening ports and which process owns them (modern replacement for `netstat`) |
| `netstat -tulpn` | Older equivalent, still common on many systems |
| `ip a` | Show network interfaces and IP addresses |
| `nslookup example.com` | DNS lookup |
| `dig example.com` | More detailed DNS lookup |
| `traceroute example.com` | Trace the network path to a host |
| `scp file.txt user@host:/path/` | Copy a file to a remote server over SSH |
| `ssh user@host` | Connect to a remote server |
| `ssh -i key.pem user@host` | Connect using a specific private key (very common for EC2) |

---

## Part 9: Package Management

Varies by distro family:

**Debian/Ubuntu (`apt`):**
```bash
sudo apt update                  # refresh package index
sudo apt install nginx           # install a package
sudo apt remove nginx            # remove a package
sudo apt upgrade                 # upgrade all installed packages
```

**RHEL/CentOS/Amazon Linux (`yum` / `dnf`):**
```bash
sudo yum install nginx
sudo yum update
sudo dnf install nginx           # newer systems use dnf
```

---

## Part 10: Environment Variables & Shell Config

| Command | Use |
|---|---|
| `export VAR_NAME=value` | Set an environment variable for the current session |
| `echo $VAR_NAME` | Print the value of a variable |
| `env` | List all environment variables |
| `unset VAR_NAME` | Remove an environment variable |
| `which node` | Show the path to an executable (useful to check what version/install is being used) |
| `alias ll='ls -la'` | Create a shortcut command |
| `source ~/.bashrc` | Reload shell config after editing it |

---

## Part 11: Archiving & Compression

| Command | Use |
|---|---|
| `tar -czvf archive.tar.gz folder/` | Create a compressed archive |
| `tar -xzvf archive.tar.gz` | Extract a compressed archive |
| `zip -r archive.zip folder/` | Create a zip archive |
| `unzip archive.zip` | Extract a zip archive |

---

## Part 12: Text Processing (very useful for logs & scripting)

| Command | Use |
|---|---|
| `grep "ERROR" app.log` | Search for a pattern in a file |
| `grep -r "TODO" ./src` | Search recursively across files in a directory |
| `grep -i "error"` | Case-insensitive search |
| `grep -v "DEBUG"` | Show lines that **don't** match |
| `cat app.log \| grep "500"` | Pipe output of one command into another (core Linux concept) |
| `awk '{print $1}' file.txt` | Extract a specific column from text |
| `sed 's/old/new/g' file.txt` | Find and replace text |
| `sort file.txt` | Sort lines alphabetically/numerically |
| `uniq` | Remove duplicate adjacent lines (often combined with `sort`) |
| `cut -d',' -f2 file.csv` | Extract a column from delimited text |
| `xargs` | Build and execute commands from piped input |

---

## Part 13: Systemd / Service Management (managing running services on a server)

| Command | Use |
|---|---|
| `systemctl status nginx` | Check if a service is running |
| `systemctl start nginx` | Start a service |
| `systemctl stop nginx` | Stop a service |
| `systemctl restart nginx` | Restart a service |
| `systemctl enable nginx` | Make a service start automatically on boot |
| `systemctl disable nginx` | Prevent a service from auto-starting |
| `journalctl -u nginx` | View logs for a specific service |
| `journalctl -f` | Follow system logs live |

---

## Part 14: Common Real-World Command Combos

**Check what's using up disk space on a server:**
```bash
df -h
du -sh /var/* | sort -rh | head -10
```

**Find and tail the right log file after a deploy:**
```bash
find /var/log -name "*app*"
tail -f /var/log/myapp/app.log
```

**Check if your app's process/port is actually running:**
```bash
ps aux | grep myapp
ss -tulpn | grep 8080
```

**Make a deployment script executable and run it:**
```bash
chmod +x deploy.sh
./deploy.sh
```

**Search logs for errors around a specific time:**
```bash
grep "ERROR" app.log | tail -50
journalctl -u myapp --since "10 minutes ago"
```

**Quick health check on an API after deploying:**
```bash
curl -I http://localhost:8080/health
```

**Copy a file to an EC2 instance and SSH in:**
```bash
scp -i key.pem app.zip ec2-user@1.2.3.4:/home/ec2-user/
ssh -i key.pem ec2-user@1.2.3.4
```

---

## Part 15: Where This Connects to Terraform & Docker

- **Terraform `user_data` scripts** (used to bootstrap EC2 instances) are plain shell scripts — you'll use `apt`/`yum`, `systemctl`, `export`, and file operations from this guide directly inside them.
- **Dockerfiles' `RUN` instructions** execute shell commands during image build — package installs, permission changes (`chmod`), and file setup all use this same command set.
- **`docker exec -it container bash`** drops you into a Linux shell inside the container — from there, it's 100% standard Linux commands to debug what's happening.
- **CI/CD pipeline scripts** (GitHub Actions, GitLab CI, Jenkins) are also just shell commands under the hood — the same `grep`, `curl`, `find`, and `chmod` patterns you'd use manually.

---

## Part 16: Bare-Minimum Command Set (if you only learn 20 commands)

If you want the absolute essentials to be productive on a Linux server:

```
pwd, ls, cd, mkdir, rm, cp, mv          → navigation & file basics
cat, less, tail -f, grep                → reading & searching files/logs
chmod, chown, sudo                      → permissions
ps aux, kill, top                       → process management
df -h, du -sh                           → disk space
curl, ssh, scp                          → networking & remote access
systemctl status/start/stop/restart     → managing services
tar -xzvf / -czvf                       → archives
```

Master these first — everything else builds on top of this core set.
