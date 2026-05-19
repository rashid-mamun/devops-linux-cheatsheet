# 🐧 DevOps Linux Command Cheat Sheet


> **⚠ সতর্কতা:** `sudo` ও `rm -rf` ব্যবহারের আগে সবসময় double-check করো। Production server-এ একটা ভুল command অনেক বড় ক্ষতি করতে পারে।

---

## 📋 বিষয়সূচি

1. [File System Commands](#1-file-system-commands)
2. [Text Processing & Log Analysis](#2-text-processing--log-analysis)
3. [File Permissions & Ownership](#3-file-permissions--ownership)
4. [Process Management](#4-process-management)
5. [System Monitoring & Performance](#5-system-monitoring--performance)
6. [Network Commands](#6-network-commands)
7. [SSH — Secure Remote Access](#7-ssh--secure-remote-access)
8. [Git — Version Control](#8-git--version-control)
9. [Docker — Container Management](#9-docker--container-management)
10. [Kubernetes (kubectl)](#10-kubernetes-kubectl)
11. [Systemd — Service Management](#11-systemd--service-management)
12. [Cron — Task Scheduling](#12-cron--task-scheduling)
13. [Environment Variables & Shell Tricks](#13-environment-variables--shell-tricks)
14. [Permission Numbers — Quick Reference](#14-permission-numbers--quick-reference)

---

## 1. File System Commands

> DevOps কাজের ৬০% হলো files নিয়ে কাজ — logs দেখা, config edit করা, backup নেওয়া। এই commands না জানলে server-এ কাজই করা যাবে না।

### `ls` — Directory contents দেখাও

```bash
ls                        # বর্তমান directory-র files দেখাও
ls -la                    # সব files দেখাও — hidden সহ, permission ও size সহ
ls -lh                    # Human-readable size সহ (KB, MB, GB)
ls -lhS                   # Size অনুযায়ী sort করে দেখাও (বড় আগে)
ls -la /var/log/nginx/    # নির্দিষ্ট directory-র contents দেখাও
```

**ব্যাখ্যা:**
- `-l` = long format (permission, owner, size, date সহ)
- `-a` = all files (hidden files যেগুলো `.` দিয়ে শুরু সেগুলোও)
- `-h` = human-readable (bytes-এর বদলে KB/MB/GB দেখাবে)
- `-S` = sort by size

---

### `cd` — Directory পরিবর্তন করো

```bash
cd /var/log               # নির্দিষ্ট path-এ যাও
cd ..                     # এক ধাপ উপরে যাও
cd ~                      # Home directory-তে ফিরে যাও
cd -                      # আগের directory-তে ফিরে যাও
cd /etc/nginx/conf.d/     # Nginx config directory-তে যাও
```

**ব্যাখ্যা:** `cd` মানে "Change Directory"। DevOps-এ সারাদিন বিভিন্ন directory-তে যেতে হয় — log দেখতে, config edit করতে।

---

### `pwd` — বর্তমান location দেখাও

```bash
pwd                       # Output: /home/ubuntu/projects
```

**ব্যাখ্যা:** "Print Working Directory" — তুমি এখন কোথায় আছো তা দেখায়। Script লেখার সময় current path জানতে কাজে লাগে।

---

### `mkdir` — নতুন directory তৈরি করো

```bash
mkdir myproject                    # একটি folder তৈরি করো
mkdir -p /app/logs/2024/jan        # Nested folders একসাথে তৈরি করো
mkdir -p /opt/{app,config,logs}    # একসাথে তিনটি folder তৈরি করো
```

**ব্যাখ্যা:** `-p` flag দিলে intermediate directories automatically তৈরি হয়, এবং already exist করলে error দেয় না।

---

### `cp` — File/Directory copy করো

```bash
cp file.txt /tmp/                        # File copy করো
cp -r myfolder/ /backup/                 # পুরো folder recursively copy করো
cp -rp /app /backup/app_$(date +%F)      # Permission ও timestamp preserve করে copy
cp nginx.conf nginx.conf.bak             # Config-এর backup নাও
```

**ব্যাখ্যা:**
- `-r` = recursive (folder সহ সব কিছু copy)
- `-p` = preserve (permission, timestamp অপরিবর্তিত রাখো)
- `$(date +%F)` = আজকের date automatically যোগ হবে (যেমন: 2024-01-15)

---

### `mv` — File/Directory সরাও বা rename করো

```bash
mv old.txt new.txt                   # Rename করো
mv report.pdf /home/user/docs/       # অন্য directory-তে সরাও
mv app.conf app.conf.bak             # Config backup করার সহজ উপায়
```

**ব্যাখ্যা:** Same filesystem-এ `mv` instant হয় — data copy হয় না, শুধু নাম পরিবর্তন হয়।

---

### `rm` — File/Directory মুছো

```bash
rm file.txt                    # একটি file মুছো
rm -rf myfolder/               # পুরো folder মুছো (confirmation ছাড়া)
rm -rf /tmp/old_builds/        # Temporary files পরিষ্কার করো
```

**⚠ বিপদজ্জনক:** `-r` = recursive, `-f` = force। Linux-এ Recycle Bin নেই — একবার মুছলে ফেরত আসবে না! Production-এ `rm -rf` চালানোর আগে path দুইবার check করো।

---

### `find` — File খোঁজো

```bash
find /home -name '*.txt'              # নাম দিয়ে file খোঁজো
find . -type d                        # শুধু directories খোঁজো
find /var -name '*.log' -mtime +30    # ৩০ দিনের বেশি পুরনো log খোঁজো
find / -size +100M -type f 2>/dev/null  # ১০০MB-এর বেশি বড় file খোঁজো
find /tmp -mtime +7 -delete           # ৭ দিনের পুরনো files মুছে দাও
```

**ব্যাখ্যা:**
- `-name` = file-এর নাম দিয়ে filter
- `-type f` = শুধু files, `-type d` = শুধু directories
- `-mtime +30` = ৩০ দিনের বেশি পুরনো
- `-size +100M` = ১০০MB-এর বেশি বড়
- `2>/dev/null` = permission error messages hide করো

---

### `ln` — Symbolic link তৈরি করো

```bash
ln -s /opt/app/v2.0 /app/current     # Symbolic link তৈরি করো
ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
```

**ব্যাখ্যা:** Symbolic link হলো shortcut-এর মতো। DevOps-এ নতুন version deploy করার সময় শুধু link update করলেই হয়।

---

### `tar` — Archive তৈরি ও extract করো

```bash
# Archive তৈরি করো (compress)
tar -czvf backup.tar.gz myfolder/
tar -czvf backup_$(date +%F).tar.gz /etc/

# Archive extract করো
tar -xzvf backup.tar.gz
tar -xzvf app.tar.gz -C /opt/          # নির্দিষ্ট directory-তে extract করো

# Archive-এর contents দেখো (extract না করে)
tar -tzvf backup.tar.gz
```

**ব্যাখ্যা:**
- `-c` = create, `-x` = extract, `-t` = list
- `-z` = gzip compression ব্যবহার করো
- `-v` = verbose (কী হচ্ছে দেখাও)
- `-f` = file নাম specify করো

---

### `rsync` — Files efficiently sync করো

```bash
rsync -avz /app/ user@server:/backup/app/     # Remote server-এ sync করো
rsync -avz --delete /src/ /dst/               # Source-এর মতো করে sync (extra files মুছবে)
rsync -avzn /app/ /backup/                    # Dry run — আসলে copy করবে না, দেখাবে শুধু
```

**ব্যাখ্যা:** `rsync` শুধু changed files-ই transfer করে, তাই `cp`-এর চেয়ে অনেক fast। Network-এ যাওয়া connection break হলে resume করতে পারে।

---

## 2. Text Processing & Log Analysis

> DevOps engineer-এর প্রতিদিনের কাজ হলো logs analyze করা। Error খোঁজা, pattern বের করা, real-time monitoring — সব এই commands দিয়ে হয়।

### `cat` — File-এর content দেখাও

```bash
cat /etc/hosts                             # File-এর সব content দেখাও
cat file1.txt file2.txt > combined.txt     # দুটো file জোড়া লাগাও
cat /dev/null > app.log                    # File খালি করো (মুছবে না)
```

---

### `less` — বড় file page-by-page দেখাও

```bash
less /var/log/syslog      # বড় log file দেখাও
```

**Shortcuts:**
- `q` = বের হও
- `/keyword` = search করো
- `n` = next match, `N` = previous match
- `G` = শেষে যাও, `g` = শুরুতে যাও

---

### `tail` — File-এর শেষ অংশ দেখাও

```bash
tail -f /var/log/nginx/error.log        # Real-time log follow করো (DevOps-এ সবচেয়ে বেশি ব্যবহৃত)
tail -n 100 /var/log/app.log            # শেষ ১০০ line দেখাও
tail -f app.log | grep 'ERROR'          # Real-time-এ শুধু error দেখাও
```

**ব্যাখ্যা:** `-f` মানে "follow" — file-এ নতুন কিছু লেখা হলে সাথে সাথে দেখাবে। Production-এ incident দেখার সময় এটাই প্রথম command।

---

### `grep` — Pattern খোঁজো

```bash
grep 'ERROR' app.log                     # 'ERROR' আছে এমন line দেখাও
grep -i 'error' app.log                  # Case-insensitive search
grep -v 'DEBUG' app.log                  # 'DEBUG' নেই এমন line দেখাও (invert)
grep -c 'ERROR' app.log                  # কতবার 'ERROR' আছে গণনা করো
grep -n 'WARN' app.log                   # Line number সহ দেখাও
grep -r 'TODO' ./src/                    # সব subfolder-এও খোঁজো
grep -E 'ERROR|WARN|FATAL' app.log       # Multiple pattern খোঁজো (regex)
grep -A 3 -B 2 'Exception' app.log       # Error-এর আগে ২ ও পরে ৩ line দেখাও
```

---

### `awk` — Column-based data processing

```bash
awk '{print $1}' access.log              # প্রথম column (IP address) দেখাও
awk '{print $1, $9}' access.log          # IP ও status code দেখাও
awk '$9 == 500' access.log               # শুধু HTTP 500 error দেখাও
awk '{sum+=$10} END {print sum/NR}' access.log   # Average response time বের করো
awk -F: '{print $1}' /etc/passwd         # : দিয়ে split করে প্রথম column দেখাও
```

**ব্যাখ্যা:** `awk` হলো column-based data processing-এর জন্য সবচেয়ে powerful tool। Nginx/Apache access log analyze করতে অসাধারণ।

---

### `sed` — Text replace করো

```bash
sed 's/old/new/g' file.txt               # old কে new দিয়ে replace করো
sed -i 's/localhost/prod-db/g' config.yml  # File-এই replace করো (-i = in-place)
sed -n '10,20p' file.txt                 # ১০ থেকে ২০ নম্বর line দেখাও
sed '/^#/d' config.conf                  # Comment lines মুছে দাও
```

---

### `sort` ও `uniq` — Sort ও duplicates সরাও

```bash
sort file.txt                            # Alphabetically sort করো
sort -n numbers.txt                      # Numerically sort করো
sort -rn file.txt                        # Reverse order-এ sort করো
uniq -c sorted.txt                       # Duplicate count সহ দেখাও

# Real-world example: Top 10 IP addresses যারা সবচেয়ে বেশি request করেছে
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head -10
```

---

### `wc` — Count করো

```bash
wc -l file.txt          # Line count
wc -w file.txt          # Word count
wc -c file.txt          # Byte count
wc -l /var/log/app.log  # Log-এ কতটি entry আছে
```

---

### `pipe (|)` — Commands জোড়া লাগাও

```bash
# এক command-এর output অন্য command-এর input হিসেবে যায়
ls -la | grep '.txt'
cat app.log | grep error | wc -l
ps aux | sort -k3 -rn | head -5           # CPU-hungry top 5 process
tail -f app.log | grep -i 'error\|warn'   # Real-time error monitoring
```

**ব্যাখ্যা:** Pipe (`|`) হলো Linux-এর সবচেয়ে শক্তিশালী feature। ছোট ছোট commands জোড়া লাগিয়ে জটিল কাজ করা যায়।

---

### `xargs` — Output কে argument হিসেবে পাঠাও

```bash
find . -name '*.log' | xargs grep 'ERROR'      # সব log file-এ ERROR খোঁজো
find /tmp -mtime +7 | xargs rm -f              # পুরনো files মুছো
cat servers.txt | xargs -I{} ssh {} uptime     # সব server-এ uptime check করো
```

---

## 3. File Permissions & Ownership

> Linux security-র মূল ভিত্তি হলো permissions। Wrong permissions মানে security breach বা application crash। **r=read(4), w=write(2), x=execute(1)** — এই তিনটি মনে রাখো।

### Permission কীভাবে কাজ করে

```
-rwxr-xr-x  1  ubuntu  www-data  1234  Jan 15  deploy.sh
 ↑↑↑↑↑↑↑↑↑     ↑       ↑
 |└──┬──┘└──┬──┘└──┬──┘
 |  owner  group  others
 |
 file type (- = file, d = directory, l = link)
```

- **Owner permissions** (প্রথম তিনটি): file-এর মালিক কী করতে পারবে
- **Group permissions** (পরের তিনটি): একই group-এর users কী করতে পারবে
- **Others permissions** (শেষ তিনটি): বাকি সবাই কী করতে পারবে

---

### `chmod` — Permission পরিবর্তন করো

```bash
chmod 755 script.sh          # Owner: rwx, Group: r-x, Others: r-x
chmod 644 config.conf        # Owner: rw-, Group: r--, Others: r--
chmod 600 ~/.ssh/id_rsa      # Owner: rw-, Group: ---, Others: ---  ← SSH key-এর জন্য
chmod 400 prod.pem           # Owner: r--, বাকি সবাই কিছুই না ← AWS key-এর জন্য
chmod +x deploy.sh           # Execute permission সবার জন্য যোগ করো
chmod -R 755 /var/www/html/  # -R = recursive (সব files ও folders)
```

---

### `chown` — Owner পরিবর্তন করো

```bash
chown rahim file.txt                      # Owner পরিবর্তন করো
chown rahim:developers file.txt           # Owner ও group একসাথে পরিবর্তন করো
chown -R www-data:www-data /var/www/      # Web files-এর ownership দাও
chown -R deploy:deploy /opt/app/          # App files-এর ownership দাও
```

---

## 4. Process Management

> Production server-এ কোন process কতটুকু resource নিচ্ছে, কোনটা hang করেছে, কোনটা restart দিতে হবে — এসব জানাটা DevOps-এ critical।

### `ps` — Running processes দেখাও

```bash
ps aux                          # সব user-এর সব processes দেখাও
ps aux | grep nginx             # Nginx-সংক্রান্ত processes দেখাও
ps aux --sort=-%mem | head -10  # সবচেয়ে বেশি memory ব্যবহারকারী top 10
ps aux --sort=-%cpu | head -10  # সবচেয়ে বেশি CPU ব্যবহারকারী top 10
```

**Column-এর মানে:**
- `PID` = Process ID (unique নম্বর)
- `%CPU` = CPU ব্যবহার
- `%MEM` = Memory ব্যবহার
- `COMMAND` = কোন program চলছে

---

### `top` / `htop` — Real-time monitoring

```bash
top                      # Real-time CPU ও RAM usage দেখাও
top -u www-data          # নির্দিষ্ট user-এর processes দেখাও
htop                     # Color-coded, interactive version (আগে install করতে হবে)
```

**`top` shortcuts:**
- `q` = বের হও
- `k` = process kill করো
- `P` = CPU অনুযায়ী sort
- `M` = Memory অনুযায়ী sort

---

### `kill` — Process বন্ধ করো

```bash
kill 1234               # Gracefully বন্ধ করো (SIGTERM)
kill -15 1234           # SIGTERM — process নিজে cleanup করে বন্ধ হয়
kill -9 1234            # SIGKILL — জোর করে বন্ধ করো (last resort)
pkill nginx             # নাম দিয়ে process kill করো
pkill -f 'python app'   # Full command match করে kill করো
killall firefox         # এই নামের সব processes kill করো

# PID automatically খুঁজে kill করো
kill -9 $(pgrep nginx)
```

---

### Background Processes

```bash
nohup python app.py > app.log 2>&1 &    # Terminal বন্ধ হলেও চলতে থাকবে
command &                                # Background-এ চালাও
jobs                                     # Background jobs দেখাও
fg %1                                    # Job 1 কে foreground-এ আনো
bg %1                                    # Job 1 কে background-এ পাঠাও
Ctrl+Z                                   # Current process pause করো
```

---

### `lsof` — Port ও file usage দেখাও

```bash
lsof -i :8080              # কোন process port 8080 ব্যবহার করছে
lsof -i :80 -i :443        # Multiple ports check করো
lsof -u www-data           # নির্দিষ্ট user কোন files open রেখেছে
lsof /var/log/app.log      # কোন process এই file ব্যবহার করছে
```

---

## 5. System Monitoring & Performance

> Server কতটা healthy সেটা বোঝার জন্য এই commands। CPU, RAM, Disk, Network — সব monitor করতে হয় DevOps engineer-কে।

### Disk Usage

```bash
df -h                              # Disk কতটুকু full — human-readable
df -h | grep -v tmpfs              # tmpfs বাদ দিয়ে দেখাও
du -sh /var/log/                   # Specific folder কতটুকু জায়গা নিচ্ছে
du -sh /var/log/* | sort -h        # ছোট থেকে বড় sort করে দেখাও
du -sh * | sort -h | tail -10      # সবচেয়ে বড় ১০টি
```

**💡 Tip:** Disk ৮০% এর বেশি হলে সতর্ক হও। `df -h` নিয়মিত দেখো।

---

### Memory Usage

```bash
free -h                            # RAM usage দেখাও — human-readable
watch -n 2 free -h                 # প্রতি ২ সেকেন্ডে update করো
```

---

### System Load

```bash
uptime                             # Load average দেখাও
# Output: load average: 0.5, 0.8, 0.9
# এই তিনটি সংখ্যা = গত 1, 5, 15 মিনিটের average load
# Load average > CPU core count হলে server overloaded
```

---

### Advanced Monitoring

```bash
vmstat 1 5                         # CPU, memory, I/O প্রতি ১ সেকেন্ডে (৫ বার)
iostat -x 1 5                      # Disk I/O statistics
sar -u 1 10                        # CPU history (sysstat package দরকার)
dmesg -T | tail -20 | grep -i error  # Kernel error messages
```

---

### `watch` — Command বারবার চালাও

```bash
watch -n 2 'df -h'                 # প্রতি ২ সেকেন্ডে disk usage দেখাও
watch -n 5 'docker ps'             # প্রতি ৫ সেকেন্ডে containers দেখাও
watch -n 2 'free -h && df -h'      # একসাথে memory ও disk দেখাও
```

---

## 6. Network Commands

> Microservice architecture-এ network troubleshooting অনেক জরুরি। Service-এর মধ্যে connection হচ্ছে কিনা, DNS কাজ করছে কিনা, firewall block করছে কিনা — এই commands দিয়ে বের করা যায়।

### Connectivity Check

```bash
ping -c 4 google.com               # Internet connection আছে কিনা
ping -c 4 192.168.1.10             # Internal server reachable কিনা
traceroute google.com              # Packet কোন path দিয়ে যাচ্ছে
```

---

### HTTP Testing

```bash
curl https://myapp.com             # Simple GET request
curl -I https://myapp.com          # Header দেখাও (status code, server type)
curl -v https://myapp.com          # Verbose — সব detail দেখাও
curl -w '%{http_code}' https://myapp.com  # শুধু status code দেখাও

# POST request পাঠাও (API test)
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"rahim","email":"rahim@example.com"}' \
  https://api.myapp.com/users

# Basic auth সহ
curl -u username:password https://api.myapp.com/data

# File download করো
curl -O https://example.com/file.zip
wget https://example.com/file.zip
wget -c https://example.com/bigfile.iso  # Resume করতে পারে
```

---

### DNS Lookup

```bash
nslookup myapp.com                 # DNS resolution check করো
nslookup myapp.com 8.8.8.8         # Google DNS দিয়ে check করো
dig myapp.com                      # বিস্তারিত DNS information
dig myapp.com +short               # শুধু IP দেখাও
dig @8.8.8.8 myapp.com             # নির্দিষ্ট DNS server দিয়ে query করো
```

---

### Port Testing

```bash
telnet db-server 5432              # Database port open আছে কিনা
nc -zv redis-server 6379           # Redis port check (netcat)
nc -zv server 22                   # SSH port open আছে কিনা

# সব open ports দেখাও
ss -tuln                           # netstat-এর modern replacement
netstat -tuln                      # Traditional way
ss -tuap | grep LISTEN             # কোন process কোন port listen করছে
```

---

### Network Interface

```bash
ip addr show                       # All network interfaces ও IP
ip addr show eth0                  # Specific interface
ip route show                      # Routing table
ip route get 8.8.8.8               # Specific destination-এ কোন route যাবে

# Firewall rules
iptables -L -n -v                  # সব firewall rules দেখাও
iptables -L -n | grep DROP         # Blocked rules দেখাও
```

---

### Packet Capture (Advanced)

```bash
tcpdump -i eth0                    # সব traffic capture করো
tcpdump -i eth0 port 80            # শুধু HTTP traffic
tcpdump -i eth0 port 80 -w capture.pcap  # File-এ save করো
```

---

## 7. SSH — Secure Remote Access

> SSH হলো DevOps engineer-এর সবচেয়ে গুরুত্বপূর্ণ tool। Remote server manage, file transfer, tunnel — সব SSH দিয়েই হয়।

### Basic Connection

```bash
ssh user@192.168.1.10              # IP দিয়ে connect করো
ssh user@myserver.com              # Domain দিয়ে connect করো
ssh -p 2222 user@server            # Custom port-এ connect করো
ssh -i ~/.ssh/prod-key.pem ubuntu@ec2-ip  # Specific key file দিয়ে
```

---

### SSH Key Setup

```bash
# নতুন SSH key তৈরি করো
ssh-keygen -t ed25519 -C "devops@company.com"

# Public key server-এ copy করো (এরপর password ছাড়া login হবে)
ssh-copy-id user@server
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@server

# Key-এর permission ঠিক করো (অবশ্যই করতে হবে)
chmod 600 ~/.ssh/id_rsa
chmod 700 ~/.ssh/
```

---

### SSH Config File (~/.ssh/config)

```bash
# এই file তৈরি করলে short alias দিয়ে connect করা যাবে
Host prod
    HostName 192.168.1.10
    User ubuntu
    IdentityFile ~/.ssh/prod-key.pem
    Port 22

Host staging
    HostName staging.myapp.com
    User deploy
    IdentityFile ~/.ssh/staging-key.pem

# এরপর এভাবে connect করো:
ssh prod           # পুরো command টাইপ করতে হবে না
ssh staging
```

---

### File Transfer

```bash
# SCP — Secure Copy
scp file.txt user@server:/remote/path/           # Local থেকে Remote-এ
scp user@server:/remote/file.txt /local/path/    # Remote থেকে Local-এ
scp -r folder/ user@server:/remote/              # Folder copy করো
scp -i key.pem app.jar ubuntu@server:/opt/app/   # Key দিয়ে

# SFTP — Interactive file transfer
sftp user@server
```

---

### Port Forwarding & Tunneling

```bash
# Local port forwarding — local machine থেকে remote database access করো
ssh -L 3306:db-server:3306 jump-server
# এখন localhost:3306 আসলে db-server:3306 এ যাবে

# Jump Host (Bastion) দিয়ে private server-এ connect করো
ssh -J bastion.prod ubuntu@private-server

# Dynamic SOCKS proxy তৈরি করো
ssh -D 8080 user@server
```

---

## 8. Git — Version Control

> Git ছাড়া DevOps কল্পনাও করা যায় না। CI/CD pipeline, infrastructure as code, configuration management — সব কিছু Git-এ থাকে।

### Basic Workflow

```bash
git clone https://github.com/org/repo.git    # Repository download করো
git clone --depth 1 https://...              # শুধু latest version (দ্রুত)
git status                                   # কোন files changed তা দেখো
git diff                                     # কী পরিবর্তন হয়েছে দেখো
git add -A                                   # সব changes stage করো
git add file.txt                             # Specific file stage করো
git commit -m "fix: resolve nginx 502 error"  # Commit করো
git push origin main                         # Remote-এ push করো
git pull --rebase origin main                # Remote থেকে update নাও
```

---

### Branching

```bash
git branch                          # Local branches দেখাও
git branch -a                       # সব branches (remote সহ)
git checkout -b feature/new-api     # নতুন branch তৈরি করো ও সেখানে যাও
git checkout main                   # main branch-এ যাও
git merge feature/new-api           # Branch merge করো
git branch -d feature/new-api       # Branch delete করো (merged হলে)
git branch -D feature/old           # Force delete করো
```

---

### History ও Rollback

```bash
git log --oneline                   # Compact commit history
git log --oneline --graph --all     # Visual branch tree
git show abc1234                    # Specific commit-এর changes দেখো

# Rollback করো
git revert HEAD                     # Last commit undo করো (safe — নতুন commit তৈরি করে)
git revert abc1234                  # Specific commit undo করো
git reset --hard HEAD~1             # ⚠ Last commit মুছে দাও (pushed হলে ব্যবহার করো না)
```

---

### Tags ও Releases

```bash
git tag                             # সব tags দেখাও
git tag v1.0.0                      # Lightweight tag তৈরি করো
git tag -a v1.0.0 -m "Release 1.0.0"  # Annotated tag (recommended)
git push --tags                     # Tags remote-এ push করো
git push origin v1.0.0              # Specific tag push করো
```

---

### Useful Tricks

```bash
git stash                           # Current changes সরিয়ে রাখো
git stash pop                       # Stash ফিরিয়ে আনো
git stash list                      # সব stash দেখাও

git cherry-pick abc1234             # Specific commit অন্য branch-এ নিয়ে যাও
git blame file.txt                  # কোন line কে কখন লিখেছে দেখো
git bisect start                    # Bug কোন commit-এ ঢুকেছে খুঁজে বের করো
```

---

## 9. Docker — Container Management

> Docker হলো modern DevOps-এর অন্যতম প্রধান tool। Application containerize করলে "works on my machine" সমস্যা চলে যায়। Production deployment সহজ ও consistent হয়।

### Containers দেখাও ও Manage করো

```bash
docker ps                           # Running containers দেখাও
docker ps -a                        # সব containers (stopped সহ)
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"  # Custom format

docker start myapp                  # Stopped container start করো
docker stop myapp                   # Gracefully stop করো
docker restart myapp                # Restart করো
docker rm myapp                     # Container মুছো (বন্ধ থাকতে হবে)
docker rm -f myapp                  # Running container force remove করো
```

---

### Container চালাও

```bash
# Basic run
docker run nginx                    # Foreground-এ চালাও
docker run -d nginx                 # Background-এ চালাও (-d = detached)

# Port mapping
docker run -d -p 8080:80 nginx      # Host:Container port map করো
docker run -d -p 80:80 -p 443:443 nginx  # Multiple ports

# Full example
docker run -d \
  --name myapp \
  -p 8080:8000 \
  -e DB_URL=postgresql://... \
  -e ENV=production \
  -v /opt/app/data:/app/data \
  --restart unless-stopped \
  myapp:v1.0
```

**ব্যাখ্যা:**
- `-d` = detached (background)
- `-p host:container` = port mapping
- `-e` = environment variable
- `-v host:container` = volume mount
- `--restart unless-stopped` = crash হলে auto-restart

---

### Logs ও Debugging

```bash
docker logs myapp                   # সব logs দেখাও
docker logs -f myapp                # Real-time follow করো
docker logs --tail 100 myapp        # শেষ ১০০ line দেখাও
docker logs --since 1h myapp        # গত ১ ঘণ্টার logs

# Container-এর ভেতরে ঢুকো
docker exec -it myapp bash          # Bash shell খোলো
docker exec -it myapp sh            # Sh shell (bash না থাকলে)
docker exec myapp ls /app           # ভেতরে command চালাও (ঢুকা ছাড়া)
```

---

### Images Manage করো

```bash
docker images                       # Local images দেখাও
docker pull nginx:latest            # Image download করো
docker build -t myapp:v1.0 .        # Dockerfile থেকে image build করো
docker build -t myapp:v1.0 --no-cache .  # Cache ছাড়া fresh build
docker push myregistry.com/myapp:v1.0    # Registry-তে push করো
docker rmi myapp:old                # Image মুছো
docker tag myapp:v1.0 myapp:latest  # Tag copy করো
```

---

### Cleanup

```bash
docker system prune                 # Unused সব কিছু মুছো
docker system prune -af             # সব unused images-ও মুছো
docker volume prune                 # Unused volumes মুছো
docker image prune                  # Dangling images মুছো
```

---

### Docker Compose

```bash
docker-compose up -d                # সব services start করো (background)
docker-compose up -d --build        # Build করে start করো
docker-compose down                 # সব services stop ও remove করো
docker-compose down -v              # Volumes সহ মুছো
docker-compose logs -f              # সব services-এর logs দেখাও
docker-compose logs -f myservice    # Specific service-এর logs
docker-compose ps                   # Services-এর status দেখাও
docker-compose restart myservice    # Specific service restart করো
docker-compose exec myservice bash  # Service-এর ভেতরে ঢুকো
```

---

### Monitoring

```bash
docker stats                        # Real-time resource usage সব containers-এর
docker stats --no-stream            # Snapshot (একবার দেখাবে)
docker inspect myapp                # সব details দেখাও (JSON format)
docker inspect myapp | jq '.[0].NetworkSettings.IPAddress'  # IP দেখাও
```

---

## 10. Kubernetes (kubectl)

> Kubernetes হলো container orchestration platform। Large scale deployment, auto-scaling, self-healing — এগুলো K8s ছাড়া করা কঠিন।

### Resources দেখাও

```bash
kubectl get pods                               # সব pods দেখাও
kubectl get pods -n production                 # Specific namespace-এ
kubectl get pods -A                            # সব namespace-এ
kubectl get pods -o wide                       # IP ও Node সহ বিস্তারিত
kubectl get all -n myapp                       # সব resources একসাথে
kubectl get deployments                        # Deployments দেখাও
kubectl get services                           # Services দেখাও
kubectl get nodes                              # Cluster nodes দেখাও
```

---

### Debugging

```bash
# Pod-এর বিস্তারিত দেখো (সবচেয়ে কাজের debug command)
kubectl describe pod myapp-pod-xxxx -n prod

# Logs দেখো
kubectl logs myapp-pod -n prod
kubectl logs -f myapp-pod -n prod              # Follow করো
kubectl logs --tail=100 myapp-pod -n prod      # শেষ ১০০ line
kubectl logs myapp-pod -n prod --previous      # Crashed pod-এর logs

# Pod-এর ভেতরে ঢুকো
kubectl exec -it myapp-pod -n prod -- bash

# Events দেখো (problem identify করতে সবচেয়ে আগে দেখো)
kubectl get events -n prod --sort-by=lastTimestamp
```

---

### Deploy ও Manage

```bash
kubectl apply -f deployment.yaml              # YAML থেকে resources তৈরি/update করো
kubectl apply -f /k8s/                        # পুরো directory apply করো
kubectl delete -f deployment.yaml             # Resources মুছো
kubectl delete pod myapp-pod -n prod          # Specific pod মুছো (নতুন তৈরি হবে)
```

---

### Rollout Management

```bash
kubectl rollout status deployment/myapp -n prod    # Deployment-এর progress দেখো
kubectl rollout history deployment/myapp -n prod   # History দেখো
kubectl rollout undo deployment/myapp -n prod      # Last deploy rollback করো
kubectl rollout undo deployment/myapp --to-revision=2  # Specific version-এ যাও
```

---

### Scaling

```bash
kubectl scale deployment myapp --replicas=5 -n prod   # Manually scale করো
kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80  # Auto-scaling
```

---

### Config ও Context

```bash
kubectl config get-contexts                   # সব clusters দেখাও
kubectl config use-context prod-cluster       # Cluster switch করো
kubectl config current-context                # বর্তমান cluster দেখো
```

---

### Port Forwarding ও Debug

```bash
kubectl port-forward pod/myapp-pod 8080:80 -n prod    # Local-এ pod access করো
kubectl port-forward svc/myapp-service 8080:80 -n prod # Service forward করো
kubectl top pods -n prod                               # Resource usage দেখো
kubectl top nodes                                      # Node-এর resource usage
```

---

## 11. Systemd — Service Management

> Linux-এ সব services (nginx, mysql, app) systemd দিয়ে manage হয়। Server restart হলে auto-start, crash হলে auto-restart — সব systemd করে।

### Service Control

```bash
systemctl status nginx             # Service-এর status দেখো
systemctl start nginx              # Service start করো
systemctl stop nginx               # Service stop করো
systemctl restart nginx            # Restart করো (downtime হবে)
systemctl reload nginx             # Config reload করো (downtime হবে না)
systemctl enable nginx             # Boot-এ auto-start চালু করো
systemctl disable nginx            # Auto-start বন্ধ করো
```

---

### Logs ও Monitoring

```bash
journalctl -u nginx                          # Service-এর সব logs
journalctl -u nginx -f                       # Real-time follow করো
journalctl -u nginx --since "1 hour ago"     # গত ১ ঘণ্টার logs
journalctl -u nginx --since today            # আজকের logs
journalctl -p err --since today              # শুধু error-level logs
journalctl -u nginx -n 50                    # শেষ ৫০ line
```

---

### System Status

```bash
systemctl list-units --failed                # Failed services দেখাও
systemctl list-units --type=service          # সব services দেখাও
systemctl is-active nginx                    # Active কিনা check করো
systemctl daemon-reload                      # Unit file পরিবর্তনের পর reload করো
```

---

### Custom Service তৈরি করো

```bash
# /etc/systemd/system/myapp.service file তৈরি করো:
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
User=deploy
WorkingDirectory=/opt/app
ExecStart=/usr/bin/python3 app.py
Restart=always
RestartSec=10
EnvironmentFile=/opt/app/.env

[Install]
WantedBy=multi-user.target

# এরপর:
systemctl daemon-reload
systemctl enable myapp
systemctl start myapp
```

---

## 12. Cron — Task Scheduling

> Automated backup, log rotation, health check — এসব scheduled tasks cron দিয়ে manage করা হয়।

### Crontab Format

```
*  *  *  *  *  command
│  │  │  │  │
│  │  │  │  └─── Weekday (0-7, 0 ও 7 = Sunday)
│  │  │  └────── Month (1-12)
│  │  └───────── Day (1-31)
│  └──────────── Hour (0-23)
└─────────────── Minute (0-59)
```

### Common Examples

```bash
crontab -e                                   # Crontab edit করো
crontab -l                                   # Current crontab দেখাও
crontab -r                                   # ⚠ সব cron jobs মুছে দাও

# প্রতিদিন রাত ২টায় backup নাও
0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1

# প্রতি ৫ মিনিটে health check করো
*/5 * * * * curl -sf http://localhost/health || systemctl restart app

# প্রতি রবিবার মধ্যরাতে পুরনো logs মুছো
0 0 * * 0 find /var/log -name '*.log' -mtime +30 -delete

# প্রতিদিন সকাল ৬টায় database backup নাও
0 6 * * * /usr/bin/pg_dump mydb > /backup/db_$(date +%F).sql

# প্রতি ঘণ্টায় disk usage check করো
0 * * * * df -h | grep -E '9[0-9]%' | mail -s "Disk Alert" admin@company.com

# Server reboot হলে app start করো
@reboot /opt/app/start.sh >> /var/log/app.log 2>&1

# প্রতি ঘণ্টায় একবার
@hourly /opt/scripts/cleanup.sh

# প্রতিদিন একবার
@daily /opt/scripts/daily-report.sh
```

---

## 13. Environment Variables & Shell Tricks

> Configuration management-এর আধুনিক পদ্ধতি হলো environment variables। Passwords, API keys — সব env variable হিসেবে রাখো, code-এ hardcode করো না।

### Environment Variables

```bash
export DB_URL='postgresql://user:pass@host/db'   # Variable set করো
echo $DB_URL                                      # Value দেখাও
env | grep DB                                     # DB সংক্রান্ত সব variables দেখাও
unset OLD_API_KEY                                 # Variable মুছো
printenv                                          # সব env variables দেখাও

# .env file load করো
source /opt/app/.env
. /opt/app/.env                                   # Same thing, shorter
```

---

### Redirection

```bash
command > file.txt          # Stdout file-এ লেখো (overwrite)
command >> file.txt         # Append করো (add করো, মুছবে না)
command 2> error.log        # Stderr file-এ লেখো
command > file.txt 2>&1     # Stdout ও stderr দুটোই file-এ
command 2>/dev/null         # Error messages hide করো
```

---

### Useful Shortcuts

```bash
Ctrl+R                      # Reverse search — আগের command খোঁজো (সবচেয়ে useful shortcut)
Ctrl+C                      # Current command বন্ধ করো
Ctrl+Z                      # Current command pause করো
Ctrl+L                      # Screen clear করো (clear command-এর মতো)
Ctrl+A                      # Line-এর শুরুতে যাও
Ctrl+E                      # Line-এর শেষে যাও
!!                          # আগের command আবার চালাও
!$                          # আগের command-এর last argument ব্যবহার করো
sudo !!                     # আগের command sudo দিয়ে চালাও
```

---

### Aliases — Shortcuts তৈরি করো

```bash
# ~/.bashrc বা ~/.zshrc-এ যোগ করো:
alias ll='ls -la'
alias la='ls -la'
alias k='kubectl'
alias d='docker'
alias dc='docker-compose'
alias gs='git status'
alias gp='git push'
alias gl='git log --oneline --graph'

# Reload করো
source ~/.bashrc
```

---

### Command Chaining

```bash
# && = আগেরটা succeed হলে পরেরটা চালাও
make build && make test && make deploy

# || = আগেরটা fail হলে পরেরটা চালাও
ping -c 1 server || echo "Server is down!"

# ; = একটির পর আরেকটি চালাও (success/fail যাই হোক)
cd /tmp; ls; pwd
```

---

## 14. Permission Numbers — Quick Reference

### Number মানে কী

| Number | Binary | Permission | মানে |
|--------|--------|------------|------|
| 7 | 111 | rwx | Read + Write + Execute |
| 6 | 110 | rw- | Read + Write |
| 5 | 101 | r-x | Read + Execute |
| 4 | 100 | r-- | Read only |
| 0 | 000 | --- | কোনো permission নেই |

### Common Permission Combinations

| chmod | Owner | Group | Others | কখন ব্যবহার করবে |
|-------|-------|-------|--------|-----------------|
| `777` | rwx | rwx | rwx | **কখনো না!** Security nightmare |
| `755` | rwx | r-x | r-x | Web directories, shell scripts |
| `644` | rw- | r-- | r-- | Config files, web assets |
| `600` | rw- | --- | --- | SSH private keys (অবশ্যই এটা) |
| `400` | r-- | --- | --- | AWS .pem files, read-only secrets |
| `750` | rwx | r-x | --- | Team-accessible scripts |
| `640` | rw- | r-- | --- | Sensitive config (group-readable) |

---

## 🚀 DevOps Daily Workflow — Real Examples

### Incident Response (Server Down)

```bash
# Step 1: Server-এ connect করো
ssh -i prod-key.pem ubuntu@prod-server

# Step 2: System status দেখো
uptime && free -h && df -h

# Step 3: Service status check করো
systemctl status nginx app-service

# Step 4: Recent logs দেখো
journalctl -u nginx --since "30 minutes ago"
tail -n 100 /var/log/app/error.log

# Step 5: Port চালু আছে কিনা দেখো
ss -tuln | grep ':80\|:443\|:8080'

# Step 6: CPU/Memory কোনটা বেশি নিচ্ছে দেখো
ps aux --sort=-%cpu | head -10

# Step 7: Restart করো
systemctl restart nginx
```

---

### New Application Deploy করো

```bash
# Step 1: Code নামাও
cd /opt/app
git pull --rebase origin main

# Step 2: Docker image build করো
docker build -t myapp:$(git rev-parse --short HEAD) .

# Step 3: Deploy করো
docker-compose down
docker-compose up -d

# Step 4: Health check করো
sleep 5
curl -f http://localhost/health && echo "Deploy successful!" || echo "Deploy failed!"

# Step 5: Logs দেখো
docker-compose logs -f --tail=50
```

---

### Disk Full হলে কী করবে

```bash
# Step 1: কোথায় জায়গা নেই দেখো
df -h

# Step 2: কোন folder বেশি জায়গা নিচ্ছে
du -sh /var/log/* | sort -h | tail -10
du -sh /home/* | sort -h

# Step 3: বড় files খোঁজো
find / -size +500M -type f 2>/dev/null

# Step 4: পুরনো logs মুছো
find /var/log -name '*.log' -mtime +30 -delete
find /var/log -name '*.gz' -mtime +7 -delete

# Step 5: Docker cleanup করো
docker system prune -af
```

---

## ⚠ Production Safety Rules

1. **`rm -rf` চালানোর আগে** `echo` দিয়ে path print করো, তারপর run করো
2. **Database-এ কিছু করার আগে** অবশ্যই backup নাও
3. **`.env` file** কখনো git-এ push করো না — `.gitignore`-এ রাখো
4. **Production-এ** সরাসরি কাজ না করে staging-এ test করো আগে
5. **`sudo -i`** বা `sudo su` ব্যবহার কমাও — নির্দিষ্ট command-এ `sudo` দাও
6. **SSH key** সবসময় `chmod 600` রাখো
7. **Cron job** চালানোর আগে manually একবার test করো

---

