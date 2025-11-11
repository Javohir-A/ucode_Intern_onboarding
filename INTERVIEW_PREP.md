# Technical Interview Preparation - Quick Guide

## 1. OS (3%)

### Folder Structure (50%)
**Q: What are /etc, /var, /bin directories for?**
- `/etc` - Configuration files (nginx.conf, ssh_config)
- `/var` - Variable data (logs in /var/log, temp files)
- `/bin` - Essential binaries (ls, cp, cat)
- `/usr/bin` - User binaries (wget, curl)
- `/sbin` - System binaries (fdisk, iptables)
- `/opt` - Optional/third-party software
- `/home` - User home directories
- `/root` - Root user home directory
- `/tmp` - Temporary files (cleared on reboot)
    
**Q: What is PATH and GOROOT?**
- `PATH`: Environment variable listing directories where executables are located
- `GOROOT`: Location of Go installation
```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin

export PATH=$PATH:/new/directory
export GOROOT=/usr/local/go
```

**Scenario: App can't find binary**
```bash
# Problem: ./myapp: command not found
# Solution: Either move to PATH or add directory to PATH
sudo cp myapp /usr/local/bin/
# OR
export PATH=$PATH:/path/to/myapp
```

### Processes & Threads (50%)
**Q: What's the difference between process and thread?**
- **Process**: Independent program with own memory space
  - Has PID (Process ID)
  - Isolated from other processes
  - More overhead to create/switch
  
- **Thread**: Lightweight process sharing memory
  - Part of a process
  - Shares memory with other threads
  - Faster context switching
  - Risk of race conditions

**Q: What is a zombie process?**
- Child process that finished but parent hasn't read exit status
- Shows as `<defunct>` in ps
```bash
ps aux | grep defunct
# Fix: Parent should call wait() or kill parent process
```

**Scenario: Server running out of memory**
```bash
# 1. Find memory-hungry processes
ps aux --sort=-%mem | head -10

# 2. Check specific process
ps -p 1234 -o %mem,%cpu,cmd

# 3. Kill gracefully
kill -15 1234    # SIGTERM (graceful)
kill -9 1234     # SIGKILL (force)

# 4. Check if process respawns
ps aux | grep myapp
```

### Terminal Commands (50%)

**Process Management:**
```bash
# List all processes
ps aux
ps -ef

# Find process by name
ps aux | grep nginx
pgrep nginx

# Kill process
kill -15 1234     # Graceful shutdown
kill -9 1234      # Force kill
killall nginx     # Kill all nginx processes

# Background jobs
./long-running-script.sh &    # Run in background
jobs                           # List background jobs
fg %1                          # Bring job to foreground
bg %1                          # Resume job in background

# Monitor processes
htop              # Interactive process viewer
top               # Real-time process monitor
```

**File Operations:**
```bash
# List files
ls -la            # List all (including hidden)
ls -lh            # Human readable sizes
ls -lt            # Sort by time

# Change directory
cd /var/log
cd ~              # Go to home
cd -              # Go to previous directory

# Copy files
cp source.txt dest.txt
cp -r /source/dir /dest/dir    # Recursive
scp file.txt user@remote:/path # Secure copy to remote

# File permissions
chmod 755 script.sh            # rwxr-xr-x
chmod +x script.sh             # Make executable
chown user:group file.txt      # Change ownership
```

**Disk Management:**
```bash
# Disk usage
df -h             # Disk free space (human readable)
du -sh *          # Size of each item in current directory
du -h --max-depth=1 /var/log   # Folder sizes

# Find large files
find / -type f -size +100M
```

**Network Tools:**
```bash
# Capture network packets
tcpdump -i eth0
tcpdump -i any port 80         # Capture HTTP traffic
tcpdump -w capture.pcap        # Save to file

# Network scan
nmap 192.168.1.0/24            # Scan network
nmap -p 80,443 example.com     # Scan specific ports

# Check connections
netstat -tulpn                 # Show listening ports
ss -tulpn                      # Modern alternative to netstat
lsof -i :8080                  # What's using port 8080?
```

**Real-World Scenario: Debug Production Server**
```bash
# 1. Check if service is running
ps aux | grep myapp

# 2. Check resource usage
htop

# 3. Check logs
tail -f /var/log/myapp/error.log

# 4. Check network connections
netstat -tulpn | grep 8080

# 5. Check disk space (common issue!)
df -h

# 6. If disk full, find large files
du -sh /var/log/*
find /var/log -type f -size +100M

# 7. Restart service
systemctl restart myapp
```

---

## 2. Networking (10%)

### OSI Model (20%)
**Q: Name the 7 layers**
1. **Physical** - Cables, electrical signals
2. **Data Link** - MAC addresses, switches, Ethernet
3. **Network** - IP addresses, routers, IPv4/IPv6
4. **Transport** - TCP/UDP, ports, reliability
5. **Session** - Establish/manage connections
6. **Presentation** - Encryption, compression, data format
7. **Application** - HTTP, FTP, SMTP, DNS

**Mnemonic**: Please Do Not Throw Sausage Pizza Away

**Real-World Example:**
```
Sending email: john@example.com → bob@example.com

Layer 7 (Application): SMTP protocol, email content
Layer 6 (Presentation): Encrypt email with TLS
Layer 5 (Session): Establish connection with mail server
Layer 4 (Transport): TCP port 587, ensure delivery
Layer 3 (Network): Route via IP addresses
Layer 2 (Data Link): MAC address on local network
Layer 1 (Physical): Electrical signals through cable
```

### TCP vs UDP (25%)

**TCP (Transmission Control Protocol):**
- ✅ Reliable delivery
- ✅ Ordered packets
- ✅ Error checking
- ✅ Flow control
- ❌ Slower (overhead)

**UDP (User Datagram Protocol):**
- ✅ Fast
- ✅ Low overhead
- ❌ No guarantee of delivery
- ❌ No order guarantee
- ❌ No error recovery

**When to use each:**

| Protocol | Use Cases | Examples |
|----------|-----------|----------|
| **TCP** | File transfer, web browsing, email, chat | HTTP, FTP, SMTP, SSH |
| **UDP** | Real-time streaming, gaming, DNS, VoIP | Video calls, online games, DNS queries |

**Scenario: Choose protocol for video conferencing**
```
Answer: UDP
Reason: 
- Real-time communication (low latency critical)
- Acceptable to lose few frames
- TCP retransmits lost packets → delays → bad UX
- UDP sends data fast → if packet lost, skip it
```

**3-Way TCP Handshake:**
```
Client                    Server
  |                         |
  |-------- SYN -------->   |  (Client: Want to connect?)
  |                         |
  |<----- SYN-ACK -------   |  (Server: OK, ready!)
  |                         |
  |-------- ACK -------->   |  (Client: Great, let's start!)
  |                         |
  |===== Connection =====   |
```

### HTTP/HTTPS (40%)

**HTTP Status Codes:**

| Code | Meaning | Use Case |
|------|---------|----------|
| **2xx Success** |
| 200 | OK | Request succeeded |
| 201 | Created | Resource created (POST) |
| 204 | No Content | Success but no data to return |
| **3xx Redirection** |
| 301 | Moved Permanently | Resource moved to new URL |
| 302 | Found (Temporary) | Temporary redirect |
| 304 | Not Modified | Use cached version |
| **4xx Client Error** |
| 400 | Bad Request | Invalid syntax |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | No permission |
| 404 | Not Found | Resource doesn't exist |
| 429 | Too Many Requests | Rate limit exceeded |
| **5xx Server Error** |
| 500 | Internal Server Error | Generic server error |
| 502 | Bad Gateway | Invalid response from upstream |
| 503 | Service Unavailable | Server overloaded/down |
| 504 | Gateway Timeout | Upstream server timeout |

**HTTPS Explained:**
```
HTTP  + TLS/SSL = HTTPS
(Plain text)   (Encryption)   (Secure)

How it works:
1. Client requests HTTPS connection
2. Server sends SSL certificate
3. Client verifies certificate
4. Both establish encrypted session
5. All data encrypted during transmission
```

**Scenario: API returns 403 Forbidden**
```
Possible causes:
1. User authenticated but lacks permission
2. IP address blocked
3. Invalid API key
4. CORS issue
5. Rate limit exceeded

Debug:
- Check authentication token
- Verify user permissions
- Check server logs
- Test with Postman/curl
```

**HTTP Methods:**
```
GET     /users          Retrieve list (idempotent, safe)
GET     /users/1        Retrieve one user
POST    /users          Create new user (not idempotent)
PUT     /users/1        Full update (idempotent)
PATCH   /users/1        Partial update
DELETE  /users/1        Delete user (idempotent)
HEAD    /users          Like GET but no body
OPTIONS /users          Get allowed methods (CORS)
```

**Idempotent**: Same request multiple times = same result
- GET, PUT, DELETE → Idempotent ✅
- POST → Not idempotent ❌ (creates multiple resources)

### FTP (10%)

**Q: What is FTP?**
- File Transfer Protocol
- Transfer files between client and server
- Uses TCP ports 20 (data) and 21 (control)

```bash
# Connect to FTP server
ftp ftp.example.com

# Commands
ls              # List files
cd directory    # Change directory
get file.txt    # Download file
put file.txt    # Upload file
mget *.txt      # Download multiple
bye             # Disconnect

# SFTP (Secure FTP) - recommended
sftp user@example.com
```

**FTP vs SFTP vs FTPS:**
- **FTP**: Plain text, not secure
- **SFTP**: SSH File Transfer (port 22), encrypted
- **FTPS**: FTP with SSL/TLS, encrypted

### SMTP (5%)

**Q: What is SMTP?**
- Simple Mail Transfer Protocol
- Sends emails between servers
- Port 25 (plain), 587 (TLS), 465 (SSL)

**Email Flow:**
```
[Your Email Client]
       ↓ SMTP
[Your Mail Server]
       ↓ SMTP
[Recipient's Mail Server]
       ↓ IMAP/POP3
[Recipient's Email Client]
```

**SMTP Commands:**
```
HELO mail.example.com          # Identify sender
MAIL FROM:<sender@example.com> # Sender address
RCPT TO:<recipient@gmail.com>  # Recipient
DATA                           # Start message content
Subject: Hello
This is the email body.
.                              # End message
QUIT                           # Close connection
```

**Scenario: Email not being delivered**
```
Troubleshooting:
1. Check SMTP credentials
2. Verify port (587 for TLS)
3. Check DNS MX records
4. Look for spam blocking
5. Check SPF/DKIM/DMARC records
6. Review mail server logs
```

---

## 3. Git (5%)

### Basics (35%)

**Essential Commands:**
```bash
# Initialize & Clone
git init                      # Create new repo
git clone <url>              # Clone remote repo
git clone -b branch <url>    # Clone specific branch

# Status & Logs
git status                   # Check status
git log                      # View commit history
git log --oneline            # Compact log
git log --graph --all        # Visual branch history
git diff                     # See unstaged changes
git diff --staged            # See staged changes

# Staging & Committing
git add .                    # Stage all changes
git add file.txt             # Stage specific file
git commit -m "message"      # Commit with message
git commit -am "message"     # Stage & commit (tracked files)

# Branches
git branch                   # List branches
git branch feature-x         # Create branch
git checkout feature-x       # Switch branch
git checkout -b feature-x    # Create & switch
git branch -d feature-x      # Delete branch (safe)
git branch -D feature-x      # Force delete

# Remote Operations
git remote -v                # List remotes
git remote add origin <url>  # Add remote
git fetch                    # Download changes
git pull                     # Fetch + merge
git push origin main         # Push to remote
git push -u origin feature   # Push & set upstream

# Merging
git merge feature-x          # Merge branch
git merge --no-ff feature-x  # Force merge commit
```

**Scenario: Start new feature**
```bash
# 1. Update main branch
git checkout main
git pull origin main

# 2. Create feature branch
git checkout -b feature/user-auth

# 3. Make changes
# ... edit files ...

# 4. Commit changes
git add .
git commit -m "Add user authentication"

# 5. Push to remote
git push -u origin feature/user-auth

# 6. Create Pull Request on GitHub/GitLab
```

**Scenario: Undo mistakes**
```bash
# Undo last commit (keep changes)
git reset --soft HEAD~1

# Undo last commit (discard changes)
git reset --hard HEAD~1

# Undo changes to specific file
git checkout -- file.txt
git restore file.txt         # Modern command

# Unstage file
git reset HEAD file.txt
git restore --staged file.txt

# Amend last commit message
git commit --amend -m "New message"

# Amend last commit (add more changes)
git add forgotten-file.txt
git commit --amend --no-edit
```

### Workflows (30%)

**1. Centralized Workflow**
```
Everyone works on main branch (not recommended for teams)

Developer A          Developer B
    |                     |
    | git pull            | git pull
    | make changes        | make changes
    | git commit          | git commit
    | git push            | git push
```

**2. Feature Branch Workflow** ⭐ Most Common
```
main branch (always deployable)
    ↓
feature/login ───→ PR ───→ merge to main
feature/signup ──→ PR ───→ merge to main

Steps:
1. git checkout main
2. git pull origin main
3. git checkout -b feature/new-feature
4. Make changes, commit
5. git push origin feature/new-feature
6. Create Pull Request
7. Code review
8. Merge to main
9. Delete feature branch
```

**3. Gitflow Workflow**
```
main (production)
    ↓
develop (integration)
    ↓
feature branches → merge to develop
release branches → prepare release
hotfix branches → urgent fixes to main
```

**Scenario: Resolve merge conflict**
```bash
# 1. Try to merge
git checkout main
git merge feature/login

# 2. Conflict occurs
# <<<<<<< HEAD
# current changes
# =======
# incoming changes
# >>>>>>> feature/login

# 3. Edit conflicted files
# Remove conflict markers, choose what to keep

# 4. Mark as resolved
git add conflicted-file.txt

# 5. Complete merge
git commit -m "Resolve merge conflict"
```

### Advanced (35%)

**Rebasing:**
```bash
# Rebase feature onto main
git checkout feature-x
git rebase main

# Interactive rebase (clean history)
git rebase -i HEAD~3         # Edit last 3 commits

# Options in interactive rebase:
# pick   - use commit
# reword - change commit message
# edit   - amend commit
# squash - combine with previous
# drop   - remove commit
```

**Scenario: Squash commits before PR**
```bash
# You have messy commits:
# - "wip"
# - "fix typo"
# - "still working"
# - "done"

# Squash into one clean commit
git rebase -i HEAD~4

# Change in editor:
pick abc123 wip
squash def456 fix typo
squash ghi789 still working
squash jkl012 done

# Save, write new commit message
# Result: 1 clean commit
```

**Stashing:**
```bash
# Save work temporarily
git stash                    # Stash changes
git stash save "WIP: feature" # Stash with message

# List stashes
git stash list

# Apply stash
git stash apply              # Keep stash
git stash pop                # Apply & remove
git stash apply stash@{2}    # Apply specific

# Drop stash
git stash drop stash@{0}
git stash clear              # Remove all stashes
```

**Scenario: Switch branches with uncommitted changes**
```bash
# Working on feature-a, need to fix bug urgently
git stash                    # Save current work
git checkout main
git checkout -b hotfix/bug
# Fix bug, commit, push
git checkout feature-a
git stash pop                # Resume work
```

**Cherry-pick:**
```bash
# Apply specific commit from another branch
git cherry-pick abc123

# Cherry-pick without committing
git cherry-pick -n abc123
```

**Submodules:**
```bash
# Add submodule
git submodule add https://github.com/user/repo.git path/to/submodule

# Clone repo with submodules
git clone --recursive <url>

# Update submodules
git submodule update --init --recursive
```

**Reflog (recover lost commits):**
```bash
# View all ref updates
git reflog

# Recover lost commit
git reflog
# Find commit hash
git checkout abc123
git branch recovered-branch
```

**Scenario: Accidentally deleted branch with uncommitted work**
```bash
# Find lost commits
git reflog

# Output shows:
# abc123 HEAD@{0}: checkout: moving from feature-x to main
# def456 HEAD@{1}: commit: Important changes

# Recover
git checkout def456
git checkout -b recovered-feature
```

---

## 4. Programming (10%)

### Syntax & Variables (5%)

**Q: What are variables?**
- Named storage for data
- Must declare before use

```go
// Variable declaration
var name string = "John"
var age int = 30
var isActive bool = true

// Short declaration
name := "John"
age := 30

// Multiple variables
var x, y, z int = 1, 2, 3

// Constants
const PI = 3.14159
```

### Basic Data Types (10%)

**Q: Common basic types?**

```go
// Integer types
int, int8, int16, int32, int64
uint, uint8, uint16, uint32, uint64

// Floating point
float32, float64

// String
string

// Boolean
bool

// Type conversion
var i int = 42
var f float64 = float64(i)
var u uint = uint(f)
```

**Interview Q: What's the difference between int32 and int64?**
- int32: 32-bit integer (-2,147,483,648 to 2,147,483,647)
- int64: 64-bit integer (much larger range)
- Use int32 to save memory, int64 for large numbers

### More Types - Array, Map, Struct (10%)

**Arrays:**
```go
// Fixed size
var arr [5]int = [5]int{1, 2, 3, 4, 5}
arr[0] = 10  // Access element

// Array with initialization
names := [3]string{"Alice", "Bob", "Charlie"}
```

**Slices (Dynamic arrays):**
```go
// Dynamic size
slice := []int{1, 2, 3}
slice = append(slice, 4)  // Add element

// Make slice
s := make([]int, 5)      // length 5
s := make([]int, 5, 10)  // length 5, capacity 10

// Slice operations
s[1:3]   // Get elements 1 to 2
len(s)   // Length
cap(s)   // Capacity
```

**Maps (Key-Value pairs):**
```go
// Declare map
ages := map[string]int{
    "Alice": 25,
    "Bob": 30,
}

// Operations
ages["Charlie"] = 35        // Add
age := ages["Alice"]        // Get
delete(ages, "Bob")         // Delete
val, exists := ages["Bob"]  // Check existence

// Make map
m := make(map[string]int)
```

**Structs:**
```go
// Define struct
type Person struct {
    Name    string
    Age     int
    Email   string
}

// Create instance
p1 := Person{
    Name: "Alice",
    Age: 25,
    Email: "alice@example.com",
}

// Access fields
fmt.Println(p1.Name)
p1.Age = 26

// Pointer to struct
p2 := &Person{Name: "Bob", Age: 30}
```

**Interview Q: Array vs Slice?**
- Array: Fixed size, passed by value (copy)
- Slice: Dynamic size, passed by reference

### Functions (5%)

**Q: What are scopes?**

```go
// Global scope
var globalVar = "accessible everywhere"

func example() {
    // Local scope
    var localVar = "only in this function"
    
    if true {
        // Block scope
        var blockVar = "only in this block"
    }
    // blockVar not accessible here
}
```

**Pass by Value vs Pass by Reference:**

```go
// Pass by value (copy)
func addOne(x int) {
    x = x + 1  // Only changes local copy
}
num := 5
addOne(num)
fmt.Println(num)  // Still 5

// Pass by reference (pointer)
func addOneRef(x *int) {
    *x = *x + 1  // Changes original
}
num := 5
addOneRef(&num)
fmt.Println(num)  // Now 6

// Slice/Map/Channel are always passed by reference
func modifySlice(s []int) {
    s[0] = 999  // Changes original slice
}
```

**Multiple return values:**
```go
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}

result, err := divide(10, 2)
if err != nil {
    // Handle error
}
```

### Flow Control (5%)

**If/Else:**
```go
if age >= 18 {
    fmt.Println("Adult")
} else if age >= 13 {
    fmt.Println("Teenager")
} else {
    fmt.Println("Child")
}

// If with initialization
if val := getValue(); val > 0 {
    fmt.Println(val)
}
```

**Switch:**
```go
switch day {
case "Monday":
    fmt.Println("Start of week")
case "Friday":
    fmt.Println("Almost weekend")
default:
    fmt.Println("Midweek")
}

// Switch without condition (like if-else)
switch {
case age < 18:
    fmt.Println("Minor")
case age < 65:
    fmt.Println("Adult")
default:
    fmt.Println("Senior")
}
```

**For Loop:**
```go
// Traditional for
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// While-style for
for condition {
    // code
}

// Infinite loop
for {
    // Use break to exit
}

// Range over collection
for index, value := range slice {
    fmt.Println(index, value)
}

// Range over map
for key, value := range myMap {
    fmt.Println(key, value)
}
```

### Operators (10%)

**Logical Operators:**
```go
&&  // AND
||  // OR
!   // NOT

// Examples
if age > 18 && hasLicense {
    // Can drive
}

if isWeekend || isHoliday {
    // Don't work
}

if !isRaining {
    // Go outside
}
```

**Comparison Operators:**
```go
==  // Equal
!=  // Not equal
>   // Greater than
<   // Less than
>=  // Greater or equal
<=  // Less or equal

if x == y { }
if x != y { }
```

**Arithmetic Operators:**
```go
+   // Addition
-   // Subtraction
*   // Multiplication
/   // Division
%   // Modulo (remainder)
++  // Increment
--  // Decrement

result := (a + b) * c / d
remainder := x % 3
counter++
```

### OOP (10%)

**Class & Object:**
```go
// Go doesn't have classes, uses structs + methods

// "Class" definition
type Car struct {
    Brand string
    Model string
    Year  int
}

// Constructor function
func NewCar(brand, model string, year int) *Car {
    return &Car{
        Brand: brand,
        Model: model,
        Year:  year,
    }
}

// Create "object"
myCar := NewCar("Toyota", "Camry", 2020)
```

**Methods:**
```go
// Method with value receiver
func (c Car) GetInfo() string {
    return fmt.Sprintf("%s %s (%d)", c.Brand, c.Model, c.Year)
}

// Method with pointer receiver (can modify)
func (c *Car) UpdateYear(year int) {
    c.Year = year
}

// Usage
info := myCar.GetInfo()
myCar.UpdateYear(2021)
```

**Inheritance (Composition in Go):**
```go
// Base "class"
type Animal struct {
    Name string
}

func (a Animal) Speak() {
    fmt.Println("Some sound")
}

// "Inheritance" via embedding
type Dog struct {
    Animal  // Embedded struct
    Breed string
}

// Override method
func (d Dog) Speak() {
    fmt.Println("Woof!")
}

// Usage
dog := Dog{
    Animal: Animal{Name: "Buddy"},
    Breed: "Golden Retriever",
}
dog.Speak()  // Woof!
fmt.Println(dog.Name)  // Buddy (from Animal)
```

**Interfaces (Polymorphism):**
```go
// Interface
type Speaker interface {
    Speak() string
}

type Dog struct{}
func (d Dog) Speak() string { return "Woof" }

type Cat struct{}
func (c Cat) Speak() string { return "Meow" }

// Polymorphism
func MakeSound(s Speaker) {
    fmt.Println(s.Speak())
}

MakeSound(Dog{})  // Woof
MakeSound(Cat{})  // Meow
```

**Method Overloading:**
```go
// Go doesn't support overloading directly
// Use different function names or variadic parameters

func PrintInt(x int) { }
func PrintString(x string) { }

// Or use variadic
func Print(values ...interface{}) {
    for _, v := range values {
        fmt.Println(v)
    }
}
```

### Concurrency (10%)

**Goroutines (Coroutines):**
```go
// Run function concurrently
go myFunction()

// Anonymous goroutine
go func() {
    fmt.Println("Running concurrently")
}()

// Example
func fetchData(url string) {
    // Fetch data
}

// Fetch multiple URLs concurrently
urls := []string{"url1", "url2", "url3"}
for _, url := range urls {
    go fetchData(url)
}
```

**Channels:**
```go
// Create channel
ch := make(chan int)

// Send to channel
ch <- 42

// Receive from channel
value := <-ch

// Buffered channel
ch := make(chan int, 10)

// Example: Communication between goroutines
func worker(jobs <-chan int, results chan<- int) {
    for job := range jobs {
        results <- job * 2
    }
}

jobs := make(chan int, 100)
results := make(chan int, 100)

// Start workers
for w := 1; w <= 3; w++ {
    go worker(jobs, results)
}

// Send jobs
for j := 1; j <= 9; j++ {
    jobs <- j
}
close(jobs)

// Collect results
for a := 1; a <= 9; a++ {
    <-results
}
```

**Select (Multiple channels):**
```go
select {
case msg1 := <-ch1:
    fmt.Println("Received from ch1:", msg1)
case msg2 := <-ch2:
    fmt.Println("Received from ch2:", msg2)
case <-time.After(1 * time.Second):
    fmt.Println("Timeout")
}
```

**Mutex (Synchronization):**
```go
var mu sync.Mutex
var count int

func increment() {
    mu.Lock()
    count++
    mu.Unlock()
}
```

### CPU Profiling & Big O (5%)

**Big O Notation:**

**Q: What is Big O?**
- Measures algorithm efficiency
- How runtime/memory grows with input size

```
O(1)      - Constant time
O(log n)  - Logarithmic
O(n)      - Linear
O(n log n) - Log-linear
O(n²)     - Quadratic
O(2ⁿ)     - Exponential
```

**Examples:**

```go
// O(1) - Constant
func getFirst(arr []int) int {
    return arr[0]  // Always 1 operation
}

// O(n) - Linear
func findMax(arr []int) int {
    max := arr[0]
    for _, num := range arr {  // n operations
        if num > max {
            max = num
        }
    }
    return max
}

// O(n²) - Quadratic
func bubbleSort(arr []int) {
    n := len(arr)
    for i := 0; i < n; i++ {        // n times
        for j := 0; j < n-i-1; j++ { // n times
            if arr[j] > arr[j+1] {
                arr[j], arr[j+1] = arr[j+1], arr[j]
            }
        }
    }
}

// O(log n) - Binary Search
func binarySearch(arr []int, target int) int {
    left, right := 0, len(arr)-1
    for left <= right {
        mid := (left + right) / 2
        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }
    return -1
}
```

**Interview Q: Compare sorting algorithms:**
- Bubble Sort: O(n²) - Simple, slow
- Quick Sort: O(n log n) average - Fast, common
- Merge Sort: O(n log n) - Stable, consistent
- Heap Sort: O(n log n) - In-place

### Memory Management (5%)

**RAM Profiling:**

**Q: What is memory management?**
- Allocation and deallocation of memory
- Prevent memory leaks
- Optimize usage

**Stack vs Heap:**
```go
// Stack - fast, automatic cleanup
func example() {
    x := 10  // Allocated on stack
}  // x automatically freed

// Heap - manual management, slower
func createPerson() *Person {
    p := &Person{Name: "Alice"}  // Allocated on heap
    return p
}  // p persists after function returns
```

**Garbage Collection:**
- Automatic memory management
- Frees unused objects
- Can cause pause times

**Memory Leaks:**
```go
// BAD - Memory leak
var cache = make(map[string][]byte)
func loadData(key string) {
    cache[key] = []byte{...}  // Never removed!
}

// GOOD - Bounded cache
type Cache struct {
    data map[string][]byte
    maxSize int
}

func (c *Cache) Add(key string, value []byte) {
    if len(c.data) >= c.maxSize {
        // Remove oldest entry
    }
    c.data[key] = value
}
```

**Interview Q: How to optimize memory?**
1. Use pointers for large structs
2. Reuse objects (sync.Pool)
3. Avoid global variables
4. Close resources (defer file.Close())
5. Use buffered channels appropriately

### Debugging (10%)

**Q: How do you debug code?**

**1. Print Debugging:**
```go
fmt.Println("Value:", x)
fmt.Printf("Debug: x=%d, y=%d\n", x, y)
log.Println("Error:", err)
```

**2. Debugger (Delve):**
```bash
# Install
go install github.com/go-delve/delve/cmd/dlv@latest

# Run debugger
dlv debug main.go

# Breakpoint commands
break main.main
continue
next
step
print variableName
```

**3. Logging:**
```go
import "log"

log.SetFlags(log.LstdFlags | log.Lshortfile)
log.Println("Info message")
log.Printf("User %s logged in", username)
log.Fatal("Critical error")  // Exits program
```

**4. Error Handling:**
```go
result, err := someFunction()
if err != nil {
    log.Printf("Error in someFunction: %v", err)
    return err
}

// Wrap errors for context
import "fmt"
return fmt.Errorf("failed to process user %s: %w", userID, err)
```

**5. Testing:**
```go
func TestAdd(t *testing.T) {
    result := Add(2, 3)
    expected := 5
    if result != expected {
        t.Errorf("Add(2, 3) = %d; want %d", result, expected)
    }
}

// Table-driven tests
func TestAddTable(t *testing.T) {
    tests := []struct {
        a, b, want int
    }{
        {1, 2, 3},
        {0, 0, 0},
        {-1, 1, 0},
    }
    
    for _, tt := range tests {
        got := Add(tt.a, tt.b)
        if got != tt.want {
            t.Errorf("Add(%d, %d) = %d; want %d", 
                tt.a, tt.b, got, tt.want)
        }
    }
}
```

**6. Profiling:**
```go
import "runtime/pprof"

// CPU profiling
f, _ := os.Create("cpu.prof")
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()

// Memory profiling
f, _ := os.Create("mem.prof")
pprof.WriteHeapProfile(f)

// Analyze
// go tool pprof cpu.prof
```

**Common Debugging Issues:**

| Issue | Solution |
|-------|----------|
| Nil pointer | Check if pointer is nil before use |
| Index out of range | Validate array bounds |
| Race condition | Use mutex or channels |
| Memory leak | Profile and fix resource leaks |
| Deadlock | Review goroutine communication |

**Interview Q: How to debug production issues?**
1. Check logs first
2. Add more logging if needed
3. Use APM tools (monitoring)
4. Reproduce locally
5. Use profiling tools
6. Add unit tests
7. Deploy fix

---

## 🐛 Real-World Debugging Scenario

### Scenario: E-commerce Order Processing Bug

**Problem Report:**
> "Customers are complaining that sometimes their orders show wrong total prices. Some orders have $0 total, others have extremely high prices. This happens randomly!"

### Initial Buggy Code

```go
package main

import (
    "fmt"
    "log"
    "sync"
)

type Product struct {
    ID    int
    Name  string
    Price float64
}

type Order struct {
    ID       int
    Products []Product
    Total    float64
}

var orderCounter int
var orders = make(map[int]*Order)

// Calculate order total
func calculateTotal(products []Product) float64 {
    var total float64
    for i := 0; i < len(products); i++ {
        total += products[i].Price
    }
    return total
}

// Process order - BUGGY VERSION
func processOrder(products []Product) *Order {
    orderCounter++
    order := &Order{
        ID:       orderCounter,
        Products: products,
    }
    
    // Calculate total in goroutine (bug!)
    go func() {
        order.Total = calculateTotal(order.Products)
    }()
    
    orders[order.ID] = order
    return order
}

func main() {
    products := []Product{
        {ID: 1, Name: "Laptop", Price: 999.99},
        {ID: 2, Name: "Mouse", Price: 29.99},
        {ID: 3, Name: "Keyboard", Price: 79.99},
    }
    
    // Process multiple orders concurrently
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            order := processOrder(products)
            fmt.Printf("Order #%d Total: $%.2f\n", order.ID, order.Total)
        }()
    }
    wg.Wait()
}
```

---

### Step 1: Print Debugging - Initial Investigation

**Add debug prints to understand flow:**

```go
func processOrder(products []Product) *Order {
    fmt.Println("=== DEBUG: processOrder called ===")
    orderCounter++
    fmt.Printf("DEBUG: orderCounter = %d\n", orderCounter)
    
    order := &Order{
        ID:       orderCounter,
        Products: products,
    }
    
    fmt.Printf("DEBUG: Created order with ID: %d\n", order.ID)
    
    go func() {
        fmt.Printf("DEBUG: Goroutine calculating total for order %d\n", order.ID)
        order.Total = calculateTotal(order.Products)
        fmt.Printf("DEBUG: Order %d total calculated: $%.2f\n", order.ID, order.Total)
    }()
    
    orders[order.ID] = order
    fmt.Printf("DEBUG: Returning order %d with total: $%.2f\n", order.ID, order.Total)
    return order
}
```

**Output reveals the problem:**
```
=== DEBUG: processOrder called ===
DEBUG: orderCounter = 1
DEBUG: Created order with ID: 1
DEBUG: Returning order 1 with total: $0.00    ← BUG! Total is 0
DEBUG: Goroutine calculating total for order 1
Order #1 Total: $0.00
DEBUG: Order 1 total calculated: $1109.97     ← Calculated AFTER return!
```

**Discovery:** Total is calculated AFTER the order is returned! Race condition!

---

### Step 2: Proper Logging - Add Structured Logs

```go
import (
    "log"
    "os"
)

var logger *log.Logger

func init() {
    // Set up logger with timestamp and file info
    logger = log.New(os.Stdout, "ORDER: ", log.LstdFlags|log.Lshortfile)
}

func processOrder(products []Product) *Order {
    logger.Printf("Processing order with %d products", len(products))
    
    orderCounter++
    order := &Order{
        ID:       orderCounter,
        Products: products,
    }
    
    logger.Printf("Order %d created", order.ID)
    
    // BUG IDENTIFIED: Async calculation causes race condition
    go func() {
        logger.Printf("Starting calculation for order %d", order.ID)
        order.Total = calculateTotal(order.Products)
        logger.Printf("Order %d: Total calculated = $%.2f", order.ID, order.Total)
    }()
    
    orders[order.ID] = order
    logger.Printf("Order %d stored with total $%.2f", order.ID, order.Total)
    return order
}
```

---

### Step 3: Identify Race Condition

**Run with race detector:**

```bash
go run -race main.go
```

**Output:**
```
==================
WARNING: DATA RACE
Write at 0x00c0000a0018 by goroutine 7:
  main.processOrder.func1()
      /main.go:45 +0x64

Previous read at 0x00c0000a0018 by goroutine 6:
  main.main.func1()
      /main.go:62 +0x84
==================
```

**Multiple bugs identified:**
1. ⚠️ **Race condition**: `order.Total` written by goroutine, read by main
2. ⚠️ **Race condition**: `orderCounter` accessed by multiple goroutines
3. ⚠️ **Logic error**: Total calculated asynchronously but returned immediately

---

### Step 4: Write Tests First (TDD approach)

```go
package main

import "testing"

func TestCalculateTotal(t *testing.T) {
    tests := []struct {
        name     string
        products []Product
        want     float64
    }{
        {
            name: "single product",
            products: []Product{
                {ID: 1, Name: "Laptop", Price: 999.99},
            },
            want: 999.99,
        },
        {
            name: "multiple products",
            products: []Product{
                {ID: 1, Name: "Laptop", Price: 999.99},
                {ID: 2, Name: "Mouse", Price: 29.99},
            },
            want: 1029.98,
        },
        {
            name:     "empty order",
            products: []Product{},
            want:     0.0,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := calculateTotal(tt.products)
            if got != tt.want {
                t.Errorf("calculateTotal() = %.2f, want %.2f", got, tt.want)
            }
        })
    }
}

func TestProcessOrder(t *testing.T) {
    products := []Product{
        {ID: 1, Name: "Laptop", Price: 999.99},
        {ID: 2, Name: "Mouse", Price: 29.99},
    }
    
    order := processOrder(products)
    
    // Test that order is created
    if order == nil {
        t.Fatal("processOrder returned nil")
    }
    
    // Test that total is calculated correctly
    expectedTotal := 1029.98
    if order.Total != expectedTotal {
        t.Errorf("Order total = %.2f, want %.2f", order.Total, expectedTotal)
    }
    
    // Test that order is stored
    if storedOrder, exists := orders[order.ID]; !exists {
        t.Error("Order not stored in orders map")
    } else if storedOrder.Total != expectedTotal {
        t.Errorf("Stored order total = %.2f, want %.2f", storedOrder.Total, expectedTotal)
    }
}

func TestConcurrentOrders(t *testing.T) {
    products := []Product{
        {ID: 1, Name: "Test", Price: 100.0},
    }
    
    var wg sync.WaitGroup
    orderCount := 10
    
    for i := 0; i < orderCount; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            order := processOrder(products)
            if order.Total != 100.0 {
                t.Errorf("Order total = %.2f, want 100.0", order.Total)
            }
        }()
    }
    
    wg.Wait()
    
    // Check all orders were created
    if len(orders) != orderCount {
        t.Errorf("Created %d orders, want %d", len(orders), orderCount)
    }
}
```

**Run tests:**
```bash
go test -v -race
```

**Result:** Tests fail, confirming the bugs!

---

### Step 5: Fix with Proper Error Handling & Synchronization

```go
package main

import (
    "fmt"
    "log"
    "sync"
)

type Product struct {
    ID    int
    Name  string
    Price float64
}

type Order struct {
    ID       int
    Products []Product
    Total    float64
    mu       sync.RWMutex  // Protect Total field
}

var (
    orderCounter int
    counterMu    sync.Mutex
    orders       = make(map[int]*Order)
    ordersMu     sync.RWMutex
)

// Calculate order total with error handling
func calculateTotal(products []Product) (float64, error) {
    if len(products) == 0 {
        return 0, fmt.Errorf("cannot calculate total for empty order")
    }
    
    var total float64
    for _, product := range products {
        if product.Price < 0 {
            return 0, fmt.Errorf("invalid price for product %d: %.2f", product.ID, product.Price)
        }
        total += product.Price
    }
    return total, nil
}

// FIXED: Process order synchronously with proper locking
func processOrder(products []Product) (*Order, error) {
    log.Printf("Processing order with %d products", len(products))
    
    // Calculate total FIRST
    total, err := calculateTotal(products)
    if err != nil {
        log.Printf("Error calculating total: %v", err)
        return nil, fmt.Errorf("failed to calculate total: %w", err)
    }
    
    // Safely increment counter
    counterMu.Lock()
    orderCounter++
    orderID := orderCounter
    counterMu.Unlock()
    
    // Create order with calculated total
    order := &Order{
        ID:       orderID,
        Products: products,
        Total:    total,
    }
    
    // Safely store order
    ordersMu.Lock()
    orders[order.ID] = order
    ordersMu.Unlock()
    
    log.Printf("Order %d created successfully with total $%.2f", order.ID, order.Total)
    return order, nil
}

// Safe getter for order total
func (o *Order) GetTotal() float64 {
    o.mu.RLock()
    defer o.mu.RUnlock()
    return o.Total
}

func main() {
    products := []Product{
        {ID: 1, Name: "Laptop", Price: 999.99},
        {ID: 2, Name: "Mouse", Price: 29.99},
        {ID: 3, Name: "Keyboard", Price: 79.99},
    }
    
    // Process multiple orders concurrently
    var wg sync.WaitGroup
    for i := 0; i < 5; i++ {
        wg.Add(1)
        go func(iteration int) {
            defer wg.Done()
            
            order, err := processOrder(products)
            if err != nil {
                log.Printf("Error processing order: %v", err)
                return
            }
            
            fmt.Printf("Order #%d Total: $%.2f\n", order.ID, order.GetTotal())
        }(i)
    }
    wg.Wait()
    
    fmt.Println("\nAll orders processed successfully!")
}
```

---

### Step 6: Verify Fix with Tests

```bash
go test -v -race
```

**Output:**
```
=== RUN   TestCalculateTotal
=== RUN   TestCalculateTotal/single_product
=== RUN   TestCalculateTotal/multiple_products
=== RUN   TestCalculateTotal/empty_order
--- PASS: TestCalculateTotal (0.00s)
=== RUN   TestProcessOrder
--- PASS: TestProcessOrder (0.00s)
=== RUN   TestConcurrentOrders
--- PASS: TestConcurrentOrders (0.01s)
PASS
ok      order-service   0.015s
```

✅ **All tests pass! No race conditions!**

---

### Step 7: Performance Profiling

**Check if our fixes caused performance issues:**

```go
package main

import (
    "os"
    "runtime/pprof"
    "testing"
)

func BenchmarkProcessOrder(b *testing.B) {
    products := []Product{
        {ID: 1, Name: "Laptop", Price: 999.99},
        {ID: 2, Name: "Mouse", Price: 29.99},
    }
    
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        processOrder(products)
    }
}

func BenchmarkConcurrentOrders(b *testing.B) {
    products := []Product{
        {ID: 1, Name: "Laptop", Price: 999.99},
    }
    
    b.RunParallel(func(pb *testing.PB) {
        for pb.Next() {
            processOrder(products)
        }
    })
}

// CPU profiling in main
func main() {
    f, _ := os.Create("cpu.prof")
    pprof.StartCPUProfile(f)
    defer pprof.StopCPUProfile()
    
    // ... run your code ...
}
```

**Run benchmarks:**
```bash
go test -bench=. -benchmem
```

**Output:**
```
BenchmarkProcessOrder-8              500000    2847 ns/op    320 B/op    5 allocs/op
BenchmarkConcurrentOrders-8         1000000    1543 ns/op    320 B/op    5 allocs/op
```

✅ **Performance is good!**

---

### Summary of Fixes

| Bug | Root Cause | Fix |
|-----|------------|-----|
| **Wrong totals** | Total calculated in goroutine after return | Calculate synchronously before returning |
| **Race condition on Total** | Concurrent read/write to order.Total | Use mutex (RWMutex) |
| **Race condition on counter** | Multiple goroutines increment counter | Use mutex for counter |
| **No error handling** | No validation of inputs | Add error returns and validation |
| **Missing tests** | No way to verify correctness | Add comprehensive unit tests |

### Key Debugging Lessons

1. **Print debugging** - Quick first step to understand flow
2. **Race detector** - Found concurrency bugs (`go run -race`)
3. **Proper logging** - Structured logs with context
4. **Write tests** - Verify bug and confirm fix
5. **Error handling** - Validate inputs and return errors
6. **Profiling** - Ensure fix doesn't hurt performance
7. **Code review** - Have others review concurrency code

### Interview Answer Template

**When asked: "Walk me through debugging a complex issue"**

1. **Reproduce the issue** - Get consistent repro steps
2. **Gather information** - Logs, error messages, metrics
3. **Form hypothesis** - What might cause this?
4. **Test hypothesis** - Add logging/debugging
5. **Identify root cause** - Use debugger, race detector, profiling
6. **Write test** - Create failing test that proves the bug
7. **Fix the bug** - Implement solution
8. **Verify fix** - Tests pass, no race conditions
9. **Review & deploy** - Get code review, deploy with monitoring

---

## 5. Database - SQL (7%)

### SQL Concepts - CAP & Hashing (3%)

**CAP Theorem (for Distributed Databases):**

**Q: What is CAP theorem?**
- **C**onsistency: All nodes see the same data at the same time
- **A**vailability: Every request receives a response (success/failure)
- **P**artition Tolerance: System continues despite network partitions

**You can only guarantee 2 of 3:**

| Type | Choice | Examples | Use Case |
|------|--------|----------|----------|
| **CP** | Consistency + Partition | MongoDB, HBase, Redis | Banking, financial transactions |
| **AP** | Availability + Partition | Cassandra, CouchDB, DynamoDB | Social media, analytics |
| **CA** | Consistency + Availability | PostgreSQL, MySQL (single node) | Traditional RDBMS |

**Interview Q: Why can't you have all three?**
- During network partition, you must choose:
  - **Consistency**: Reject requests (reduce availability)
  - **Availability**: Accept requests (risk inconsistency)

**Hashing in Databases:**

**Q: What is hashing used for?**
1. **Password Storage** - One-way encryption
2. **Hash Indexes** - Fast lookups
3. **Consistent Hashing** - Distribute data across nodes

**Password Hashing Example:**
```sql
-- Store hashed password
INSERT INTO users (username, password_hash) 
VALUES ('john', crypt('mypassword', gen_salt('bf')));

-- Verify password
SELECT * FROM users 
WHERE username = 'john' 
AND password_hash = crypt('inputpassword', password_hash);
```

**Hash Index:**
```sql
-- Create hash index (exact match only)
CREATE INDEX idx_email_hash ON users USING HASH (email);

-- Good for: WHERE email = 'john@example.com'
-- Bad for: WHERE email LIKE '%john%'
```

---

### Normalization (6%)

**Q: What is database normalization?**
- Process of organizing data to reduce redundancy
- Split large tables into smaller, related tables

**1NF (First Normal Form):**
- ✅ Atomic values (no arrays/lists)
- ✅ Each column has unique name
- ✅ Each row is unique (has primary key)

```sql
-- ❌ NOT 1NF (phone_numbers is a list)
CREATE TABLE users_bad (
    id INT,
    name VARCHAR(50),
    phone_numbers VARCHAR(200)  -- "555-1234, 555-5678, 555-9999"
);

-- ✅ 1NF (atomic values)
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE phone_numbers (
    id INT PRIMARY KEY,
    user_id INT,
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

**2NF (Second Normal Form):**
- ✅ Must be in 1NF
- ✅ No partial dependencies (all non-key columns depend on ENTIRE primary key)

```sql
-- ❌ NOT 2NF (course_name depends only on course_id, not full key)
CREATE TABLE enrollments_bad (
    student_id INT,
    course_id INT,
    course_name VARCHAR(100),  -- Depends only on course_id!
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id)
);

-- ✅ 2NF (split into two tables)
CREATE TABLE courses (
    course_id INT PRIMARY KEY,
    course_name VARCHAR(100)
);

CREATE TABLE enrollments (
    student_id INT,
    course_id INT,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

**3NF (Third Normal Form):**
- ✅ Must be in 2NF
- ✅ No transitive dependencies (non-key columns depend only on primary key)

```sql
-- ❌ NOT 3NF (city and zip depend on each other)
CREATE TABLE employees_bad (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    zip_code VARCHAR(10),
    city VARCHAR(50)  -- city depends on zip_code (transitive)
);

-- ✅ 3NF (remove transitive dependency)
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    name VARCHAR(50),
    zip_code VARCHAR(10),
    FOREIGN KEY (zip_code) REFERENCES zip_codes(zip_code)
);

CREATE TABLE zip_codes (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(50),
    state VARCHAR(50)
);
```

**BCNF (Boyce-Codd Normal Form):**
- Stricter version of 3NF
- Every determinant must be a candidate key

**Interview Q: When to denormalize?**
- Read-heavy applications
- Performance is critical
- Joins are too expensive
- Accept some data redundancy for speed

---

### DDL - Data Definition Language (5%)

**Q: What is DDL?**
- Commands that define database structure

**CREATE:**
```sql
-- Create table
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- Create table with foreign key
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    user_id INT NOT NULL,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    created_at TIMESTAMP DEFAULT NOW(),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- Create index
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_posts_user ON posts(user_id);

-- Create unique constraint
CREATE UNIQUE INDEX idx_username ON users(username);
```

**ALTER:**
```sql
-- Add column
ALTER TABLE users ADD COLUMN last_login TIMESTAMP;

-- Modify column
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(150);

-- Add constraint
ALTER TABLE users ADD CONSTRAINT check_email CHECK (email LIKE '%@%');

-- Drop column
ALTER TABLE users DROP COLUMN last_login;

-- Rename column
ALTER TABLE users RENAME COLUMN username TO user_name;

-- Rename table
ALTER TABLE users RENAME TO app_users;
```

**DROP:**
```sql
-- Drop table
DROP TABLE posts;

-- Drop table if exists (safe)
DROP TABLE IF EXISTS posts;

-- Drop table with dependencies
DROP TABLE users CASCADE;

-- Drop index
DROP INDEX idx_users_email;

-- Drop database
DROP DATABASE old_database;
```

---

### DML - Data Manipulation Language (7%)

**INSERT:**
```sql
-- Insert single row
INSERT INTO users (username, email) 
VALUES ('john_doe', 'john@example.com');

-- Insert multiple rows
INSERT INTO users (username, email) VALUES 
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');

-- Insert with RETURNING
INSERT INTO users (username, email) 
VALUES ('dave', 'dave@example.com')
RETURNING id, created_at;

-- Insert from SELECT
INSERT INTO archived_users 
SELECT * FROM users WHERE created_at < '2020-01-01';
```

**UPDATE:**
```sql
-- Update single row
UPDATE users SET email = 'newemail@example.com' WHERE id = 1;

-- Update multiple columns
UPDATE users 
SET last_login = NOW(), is_active = TRUE 
WHERE username = 'john_doe';

-- Update with calculation
UPDATE products SET price = price * 1.1 WHERE category = 'electronics';

-- Update from another table
UPDATE orders o
SET status = 'shipped'
FROM shipments s
WHERE o.id = s.order_id AND s.shipped_at IS NOT NULL;
```

**DELETE:**
```sql
-- Delete specific rows
DELETE FROM users WHERE id = 1;

-- Delete with condition
DELETE FROM posts WHERE created_at < NOW() - INTERVAL '1 year';

-- Delete all rows (keep structure)
DELETE FROM temp_data;

-- Delete with RETURNING
DELETE FROM users WHERE is_active = FALSE RETURNING id, username;
```

**SELECT (Basic):**
```sql
-- Select all
SELECT * FROM users;

-- Select specific columns
SELECT id, username, email FROM users;

-- Select with alias
SELECT id AS user_id, username AS name FROM users;

-- Select distinct
SELECT DISTINCT country FROM users;

-- Select with WHERE
SELECT * FROM users WHERE age > 18 AND country = 'USA';
```

---

### DQL - Data Query Language (8%)

**SELECT with GROUP BY:**
```sql
-- Count users by country
SELECT country, COUNT(*) as user_count 
FROM users 
GROUP BY country;

-- Average salary by department
SELECT department, AVG(salary) as avg_salary 
FROM employees 
GROUP BY department;

-- Having clause (filter groups)
SELECT department, COUNT(*) as emp_count 
FROM employees 
GROUP BY department 
HAVING COUNT(*) > 10;

-- Multiple aggregations
SELECT 
    category,
    COUNT(*) as product_count,
    AVG(price) as avg_price,
    MIN(price) as min_price,
    MAX(price) as max_price
FROM products
GROUP BY category;
```

**ORDER BY:**
```sql
-- Sort ascending
SELECT * FROM users ORDER BY created_at ASC;

-- Sort descending
SELECT * FROM users ORDER BY created_at DESC;

-- Multiple columns
SELECT * FROM products 
ORDER BY category ASC, price DESC;

-- Order by expression
SELECT * FROM users 
ORDER BY LENGTH(username) DESC;
```

**PAGINATION:**
```sql
-- LIMIT and OFFSET
SELECT * FROM users 
ORDER BY id 
LIMIT 10 OFFSET 20;  -- Skip 20, get next 10

-- Page 1 (items 1-10)
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 0;

-- Page 2 (items 11-20)
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 10;

-- Page 3 (items 21-30)
SELECT * FROM products ORDER BY id LIMIT 10 OFFSET 20;

-- Calculate offset: OFFSET = (page_number - 1) * page_size
```

**Complex Queries:**
```sql
-- Subquery
SELECT * FROM users 
WHERE id IN (
    SELECT user_id FROM orders WHERE total > 1000
);

-- EXISTS
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o WHERE o.user_id = u.id
);

-- CASE statement
SELECT 
    username,
    CASE 
        WHEN age < 18 THEN 'Minor'
        WHEN age < 65 THEN 'Adult'
        ELSE 'Senior'
    END as age_group
FROM users;

-- Window functions
SELECT 
    username,
    salary,
    department,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) as salary_rank
FROM employees;
```

---

### DCL - Data Control Language (2%)

**GRANT:**
```sql
-- Grant SELECT on table
GRANT SELECT ON users TO read_only_user;

-- Grant multiple privileges
GRANT SELECT, INSERT, UPDATE ON orders TO app_user;

-- Grant all privileges
GRANT ALL PRIVILEGES ON DATABASE mydb TO admin_user;

-- Grant on specific columns
GRANT SELECT (id, username) ON users TO limited_user;

-- Grant with grant option
GRANT SELECT ON products TO manager WITH GRANT OPTION;
```

**REVOKE:**
```sql
-- Revoke SELECT
REVOKE SELECT ON users FROM read_only_user;

-- Revoke multiple privileges
REVOKE INSERT, UPDATE ON orders FROM app_user;

-- Revoke all
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM old_admin;
```

---

### TCL - Transaction Control Language (5%)

**Q: What is a transaction?**
- Group of SQL operations that execute as a single unit
- Either all succeed or all fail (atomicity)

**Transaction Commands:**
```sql
-- BEGIN transaction
BEGIN;  -- or START TRANSACTION;

    INSERT INTO accounts (user_id, balance) VALUES (1, 1000);
    UPDATE accounts SET balance = balance - 100 WHERE user_id = 1;
    INSERT INTO transactions (account_id, amount, type) VALUES (1, -100, 'debit');

-- COMMIT (save changes)
COMMIT;
```

**ROLLBACK:**
```sql
-- BEGIN transaction
BEGIN;

    UPDATE accounts SET balance = balance - 500 WHERE id = 1;
    UPDATE accounts SET balance = balance + 500 WHERE id = 2;
    
    -- Oh no, something went wrong!
    
-- ROLLBACK (undo all changes)
ROLLBACK;
```

**Real-World Example - Bank Transfer:**
```sql
BEGIN;

    -- Deduct from sender
    UPDATE accounts 
    SET balance = balance - 100 
    WHERE user_id = 1;
    
    -- Check if sender has enough balance
    SELECT balance FROM accounts WHERE user_id = 1;
    -- If balance < 0, ROLLBACK
    
    -- Add to receiver
    UPDATE accounts 
    SET balance = balance + 100 
    WHERE user_id = 2;
    
    -- Log transaction
    INSERT INTO transactions (from_user, to_user, amount) 
    VALUES (1, 2, 100);

-- All or nothing!
COMMIT;
```

**SAVEPOINT:**
```sql
BEGIN;

    INSERT INTO orders (user_id, total) VALUES (1, 100);
    SAVEPOINT order_created;
    
    INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 101, 2);
    SAVEPOINT items_added;
    
    -- Oops, wrong item!
    ROLLBACK TO SAVEPOINT items_added;
    
    -- Retry with correct item
    INSERT INTO order_items (order_id, product_id, quantity) VALUES (1, 102, 2);
    
COMMIT;
```

---

### JOINS (5%)

**INNER JOIN:**
```sql
-- Only matching rows from both tables
SELECT users.username, orders.total
FROM users
INNER JOIN orders ON users.id = orders.user_id;

-- Multiple joins
SELECT 
    users.username,
    orders.id as order_id,
    products.name as product_name
FROM users
INNER JOIN orders ON users.id = orders.user_id
INNER JOIN order_items ON orders.id = order_items.order_id
INNER JOIN products ON order_items.product_id = products.id;
```

**LEFT JOIN (LEFT OUTER JOIN):**
```sql
-- All rows from left table + matching from right
SELECT users.username, orders.total
FROM users
LEFT JOIN orders ON users.id = orders.user_id;

-- Result includes users with no orders (orders.total = NULL)

-- Find users with NO orders
SELECT users.username
FROM users
LEFT JOIN orders ON users.id = orders.user_id
WHERE orders.id IS NULL;
```

**RIGHT JOIN (RIGHT OUTER JOIN):**
```sql
-- All rows from right table + matching from left
SELECT users.username, orders.total
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;

-- Result includes orders with no user (orphaned orders)
```

**FULL OUTER JOIN:**
```sql
-- All rows from both tables
SELECT users.username, orders.total
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;

-- Shows users with no orders AND orders with no users
```

**CROSS JOIN:**
```sql
-- Cartesian product (every row with every row)
SELECT colors.name, sizes.name
FROM colors
CROSS JOIN sizes;

-- If colors has 5 rows and sizes has 3 rows = 15 result rows
```

**SELF JOIN:**
```sql
-- Join table to itself
-- Example: Find employees and their managers
SELECT 
    e.name as employee,
    m.name as manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

### Functions (3%)

**Built-in Functions:**
```sql
-- String functions
SELECT 
    UPPER(username),
    LOWER(email),
    LENGTH(description),
    CONCAT(first_name, ' ', last_name) as full_name,
    SUBSTRING(email, 1, POSITION('@' IN email) - 1) as email_user
FROM users;

-- Date functions
SELECT 
    NOW(),
    CURRENT_DATE,
    CURRENT_TIME,
    DATE_PART('year', created_at) as year,
    AGE(NOW(), created_at) as account_age,
    created_at + INTERVAL '7 days' as expires_at
FROM users;

-- Aggregate functions
SELECT 
    COUNT(*) as total_users,
    AVG(age) as average_age,
    SUM(balance) as total_balance,
    MIN(created_at) as first_user,
    MAX(created_at) as latest_user
FROM users;
```

**User-Defined Functions:**
```sql
-- Create function
CREATE OR REPLACE FUNCTION calculate_discount(price DECIMAL, discount_pct INT)
RETURNS DECIMAL AS $$
BEGIN
    RETURN price * (1 - discount_pct / 100.0);
END;
$$ LANGUAGE plpgsql;

-- Use function
SELECT 
    product_name,
    price,
    calculate_discount(price, 20) as discounted_price
FROM products;

-- Function with multiple statements
CREATE OR REPLACE FUNCTION create_user_with_profile(
    p_username VARCHAR,
    p_email VARCHAR,
    p_age INT
) RETURNS INT AS $$
DECLARE
    v_user_id INT;
BEGIN
    -- Insert user
    INSERT INTO users (username, email) 
    VALUES (p_username, p_email)
    RETURNING id INTO v_user_id;
    
    -- Insert profile
    INSERT INTO profiles (user_id, age) 
    VALUES (v_user_id, p_age);
    
    RETURN v_user_id;
END;
$$ LANGUAGE plpgsql;
```

---

### Indexes (4%)

**Q: What is an index?**
- Data structure that improves query speed
- Trade-off: Faster reads, slower writes

**B-Tree Index (Default):**
```sql
-- Best for: Range queries, sorting
CREATE INDEX idx_users_age ON users(age);

-- Good for:
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE age BETWEEN 18 AND 65;
SELECT * FROM users ORDER BY age;
```

**Hash Index:**
```sql
-- Best for: Equality comparisons only
CREATE INDEX idx_users_email_hash ON users USING HASH (email);

-- Good for:
SELECT * FROM users WHERE email = 'john@example.com';

-- BAD for:
SELECT * FROM users WHERE email LIKE '%john%';  -- Won't use index!
```

**GiST (Generalized Search Tree):**
```sql
-- Best for: Geometric data, full-text search
CREATE INDEX idx_locations ON places USING GIST (location);

-- Good for spatial queries
SELECT * FROM places WHERE location && box '((0,0),(1,1))';
```

**GIN (Generalized Inverted Index):**
```sql
-- Best for: Array, JSON, full-text search
CREATE INDEX idx_tags_gin ON posts USING GIN (tags);

-- Good for:
SELECT * FROM posts WHERE tags @> ARRAY['postgresql', 'database'];

-- JSON example
CREATE INDEX idx_data_gin ON documents USING GIN (data);
SELECT * FROM documents WHERE data @> '{"status": "active"}';
```

**Composite Index:**
```sql
-- Multiple columns
CREATE INDEX idx_orders_user_date ON orders(user_id, created_at);

-- Good for:
SELECT * FROM orders WHERE user_id = 1 AND created_at > '2024-01-01';
SELECT * FROM orders WHERE user_id = 1;  -- Uses index

-- Bad for:
SELECT * FROM orders WHERE created_at > '2024-01-01';  -- Won't use index efficiently
```

**When to Use Indexes:**
- ✅ Columns in WHERE clauses
- ✅ Columns in JOIN conditions
- ✅ Columns in ORDER BY
- ✅ Foreign key columns
- ❌ Small tables (< 1000 rows)
- ❌ Columns with low cardinality (few distinct values)
- ❌ Frequently updated columns

---

### Triggers (3%)

**Q: What is a trigger?**
- Automatic action executed when event occurs (INSERT, UPDATE, DELETE)

**Basic Trigger:**
```sql
-- Create trigger function
CREATE OR REPLACE FUNCTION update_modified_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER trigger_update_timestamp
BEFORE UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION update_modified_timestamp();
```

**Audit Log Trigger:**
```sql
-- Create audit table
CREATE TABLE user_audit (
    id SERIAL PRIMARY KEY,
    user_id INT,
    action VARCHAR(10),
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP DEFAULT NOW()
);

-- Create audit trigger function
CREATE OR REPLACE FUNCTION audit_user_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO user_audit (user_id, action, new_data)
        VALUES (NEW.id, 'INSERT', row_to_json(NEW));
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO user_audit (user_id, action, old_data, new_data)
        VALUES (NEW.id, 'UPDATE', row_to_json(OLD), row_to_json(NEW));
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO user_audit (user_id, action, old_data)
        VALUES (OLD.id, 'DELETE', row_to_json(OLD));
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Attach trigger
CREATE TRIGGER trigger_audit_users
AFTER INSERT OR UPDATE OR DELETE ON users
FOR EACH ROW
EXECUTE FUNCTION audit_user_changes();
```

**Validation Trigger:**
```sql
-- Prevent invalid email
CREATE OR REPLACE FUNCTION validate_email()
RETURNS TRIGGER AS $$
BEGIN
    IF NEW.email !~ '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$' THEN
        RAISE EXCEPTION 'Invalid email format: %', NEW.email;
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_validate_email
BEFORE INSERT OR UPDATE ON users
FOR EACH ROW
EXECUTE FUNCTION validate_email();
```

---

### Replication (4%)

**Q: What is database replication?**
- Copying data from one database (master/primary) to one or more databases (slaves/replicas)

**Types:**

**1. Master-Slave (Primary-Replica):**
```
      [Master DB] ← Writes
           |
    Replication
      /    |    \
  [Slave] [Slave] [Slave] ← Reads
```

**Benefits:**
- Load balancing (distribute reads)
- High availability (failover)
- Backup without affecting production

**2. Master-Master (Multi-Master):**
```
  [Master 1] ←→ [Master 2]
  Both accept writes
```

**Benefits:**
- No single point of failure
- Geographic distribution

**Challenges:**
- Conflict resolution
- Data consistency

**3. Streaming Replication (PostgreSQL):**
```sql
-- On primary server (postgresql.conf)
wal_level = replica
max_wal_senders = 3
wal_keep_size = 64MB

-- On replica server
primary_conninfo = 'host=primary_host port=5432 user=replicator'
```

**Interview Q: Replication lag?**
- Time delay between write on master and availability on slave
- Can cause read-after-write issues
- Solutions: Read from master for critical data, eventual consistency

---

### Sharding (4%)

**Q: What is sharding?**
- Horizontal partitioning: Split data across multiple databases
- Each shard holds a subset of data

**Sharding Strategies:**

**1. Range-Based Sharding:**
```
Users 1-1000      → Shard 1
Users 1001-2000   → Shard 2
Users 2001-3000   → Shard 3
```

**2. Hash-Based Sharding:**
```sql
-- Determine shard by hashing user_id
shard_id = hash(user_id) % num_shards

-- Example:
hash(12345) % 4 = 1  → Shard 1
hash(67890) % 4 = 2  → Shard 2
```

**3. Geographic Sharding:**
```
US users    → US Shard
EU users    → EU Shard
Asia users  → Asia Shard
```

**4. Directory-Based Sharding:**
```sql
-- Lookup table
CREATE TABLE shard_map (
    user_id INT PRIMARY KEY,
    shard_id INT
);
```

**Example Implementation:**
```go
// Shard router
func getShardForUser(userID int) string {
    shardID := userID % 4
    return fmt.Sprintf("shard_%d", shardID)
}

func getUserData(userID int) (User, error) {
    shard := getShardForUser(userID)
    db := connectToShard(shard)
    return db.Query("SELECT * FROM users WHERE id = ?", userID)
}
```

**Challenges:**
- ❌ Cross-shard queries are complex
- ❌ Transactions across shards
- ❌ Rebalancing shards
- ❌ Hotspots (uneven distribution)

**Benefits:**
- ✅ Horizontal scaling
- ✅ Better performance
- ✅ Fault isolation

**Interview Q: Sharding vs Replication?**
- **Sharding**: Splits data (scalability)
- **Replication**: Copies data (availability)
- Often used together!

---

## 6. NoSQL Databases (5%)

### Document Databases (5%)

**Q: What are document databases?**
- Store data as documents (JSON, BSON)
- Schema-less/flexible schema
- Examples: MongoDB, CouchDB

**MongoDB Example:**
```javascript
// Insert document
db.users.insertOne({
    _id: ObjectId("..."),
    username: "john_doe",
    email: "john@example.com",
    profile: {
        age: 30,
        city: "New York"
    },
    orders: [
        {orderId: 1, total: 99.99},
        {orderId: 2, total: 149.99}
    ],
    tags: ["premium", "verified"],
    created_at: new Date()
});

// Query
db.users.find({
    "profile.age": {$gt: 18},
    tags: "premium"
});

// Update embedded document
db.users.updateOne(
    {username: "john_doe"},
    {$set: {"profile.city": "Boston"}}
);

// Add to array
db.users.updateOne(
    {username: "john_doe"},
    {$push: {orders: {orderId: 3, total: 199.99}}}
);
```

---

### Denormalization (5%)

**Q: Why denormalize in NoSQL?**
- Optimize for reads
- Avoid expensive joins
- Data co-location

**SQL (Normalized):**
```sql
-- Multiple tables with joins
SELECT u.username, o.total, p.name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

**NoSQL (Denormalized):**
```javascript
// Single document with embedded data
{
    _id: 1,
    username: "john_doe",
    orders: [
        {
            orderId: 101,
            total: 299.99,
            items: [
                {productId: 1, name: "Laptop", price: 999.99},
                {productId: 2, name: "Mouse", price: 29.99}
            ]
        }
    ]
}
```

**Trade-offs:**
- ✅ Faster reads (single query)
- ✅ Better for nested data
- ❌ Data duplication
- ❌ Update anomalies
- ❌ Larger document size

---

### Joins in NoSQL - Lookup (3%)

**Q: How do you do joins in NoSQL?**
- NoSQL doesn't support traditional joins
- Use `$lookup` (MongoDB) for similar functionality

**MongoDB $lookup (like LEFT JOIN):**
```javascript
db.orders.aggregate([
    {
        $lookup: {
            from: "users",           // Foreign collection
            localField: "user_id",   // Field in orders
            foreignField: "_id",     // Field in users
            as: "user_info"          // Output array
        }
    }
]);

// Result:
{
    _id: 1,
    user_id: 123,
    total: 299.99,
    user_info: [
        {_id: 123, username: "john_doe", email: "john@example.com"}
    ]
}
```

**Multiple Lookups:**
```javascript
db.orders.aggregate([
    {
        $lookup: {
            from: "users",
            localField: "user_id",
            foreignField: "_id",
            as: "user"
        }
    },
    {
        $lookup: {
            from: "products",
            localField: "product_ids",
            foreignField: "_id",
            as: "products"
        }
    },
    {
        $unwind: "$user"  // Convert array to object
    }
]);
```

---

### CAP Theorem for NoSQL (5%)

**Recap:**
- **C**onsistency: All nodes see same data
- **A**vailability: Every request gets response
- **P**artition Tolerance: Works despite network issues

**NoSQL Database Classification:**

| Database | Type | Trade-off | Best For |
|----------|------|-----------|----------|
| MongoDB | CP | Consistency over Availability | Financial data, inventory |
| Cassandra | AP | Availability over Consistency | Analytics, IoT, social media |
| Redis | CP | Consistency over Availability | Caching, sessions |
| CouchDB | AP | Availability over Consistency | Mobile sync, offline apps |
| DynamoDB | AP | Availability over Consistency | High-scale web apps |

**Interview Q: Why is CAP important?**
- Helps choose right database for use case
- Understand trade-offs during outages
- Design systems accordingly

---

### Indexes in NoSQL (5%)

**MongoDB Indexes:**
```javascript
// Create index
db.users.createIndex({email: 1});  // 1 = ascending

// Compound index
db.users.createIndex({country: 1, age: -1});  // -1 = descending

// Unique index
db.users.createIndex({email: 1}, {unique: true});

// Text index (full-text search)
db.posts.createIndex({content: "text", title: "text"});

// Search with text index
db.posts.find({$text: {$search: "mongodb tutorial"}});

// Geospatial index
db.places.createIndex({location: "2dsphere"});

// Query nearby locations
db.places.find({
    location: {
        $near: {
            $geometry: {type: "Point", coordinates: [40.7128, -74.0060]},
            $maxDistance: 5000  // 5km
        }
    }
});
```

**Cassandra Indexes:**
```sql
-- Primary key (automatically indexed)
CREATE TABLE users (
    user_id UUID PRIMARY KEY,
    email TEXT,
    username TEXT
);

-- Secondary index
CREATE INDEX ON users (email);

-- Query with secondary index
SELECT * FROM users WHERE email = 'john@example.com';
```

---

## 7. In-Memory Databases & Caching (10%)

### Key-Value Stores (5%)

**Q: What is a key-value store?**
- Simplest NoSQL database
- Store data as key-value pairs
- Examples: Redis, Memcached

**Redis Examples:**
```bash
# String operations
SET user:1:name "John Doe"
GET user:1:name
INCR user:1:views
EXPIRE user:1:session 3600  # TTL in seconds

# Hash (like object)
HSET user:1 name "John" email "john@example.com" age 30
HGET user:1 name
HGETALL user:1

# List (ordered collection)
LPUSH recent_orders order123 order124 order125
LRANGE recent_orders 0 9  # Get first 10

# Set (unique values)
SADD user:1:tags "premium" "verified" "active"
SMEMBERS user:1:tags

# Sorted Set (with scores)
ZADD leaderboard 100 "player1" 200 "player2" 150 "player3"
ZRANGE leaderboard 0 -1 WITHSCORES  # Get all with scores
ZREVRANGE leaderboard 0 2  # Top 3
```

---

### Types of In-Memory Databases (3%)

| Type | Example | Use Case |
|------|---------|----------|
| **Key-Value** | Redis, Memcached | Caching, sessions |
| **Document** | Couchbase | Mobile apps, real-time |
| **Column** | SAP HANA | Analytics, OLAP |
| **Graph** | Neo4j (in-memory) | Social networks |

---

### Caching Strategies (10%)

**1. In-Memory Caching:**
```go
// Simple in-memory cache
var cache = make(map[string]interface{})
var mu sync.RWMutex

func Get(key string) (interface{}, bool) {
    mu.RLock()
    defer mu.RUnlock()
    val, exists := cache[key]
    return val, exists
}

func Set(key string, value interface{}) {
    mu.Lock()
    defer mu.Unlock()
    cache[key] = value
}
```

**2. Distributed Caching (Redis):**
```go
import "github.com/go-redis/redis"

// Connect to Redis
rdb := redis.NewClient(&redis.Options{
    Addr: "localhost:6379",
})

// Get from cache
func GetUser(userID int) (User, error) {
    cacheKey := fmt.Sprintf("user:%d", userID)
    
    // Try cache first
    cached, err := rdb.Get(ctx, cacheKey).Result()
    if err == nil {
        var user User
        json.Unmarshal([]byte(cached), &user)
        return user, nil
    }
    
    // Cache miss - get from DB
    user, err := db.GetUser(userID)
    if err != nil {
        return User{}, err
    }
    
    // Store in cache (TTL: 1 hour)
    data, _ := json.Marshal(user)
    rdb.Set(ctx, cacheKey, data, time.Hour)
    
    return user, nil
}
```

**3. Cache-Aside (Lazy Loading):**
```
1. Check cache
2. If miss → Query database
3. Store in cache
4. Return data
```

**4. Write-Through:**
```
1. Write to cache
2. Immediately write to database
3. Return success
```

**5. Write-Behind (Write-Back):**
```
1. Write to cache
2. Return success
3. Asynchronously write to database later
```

---

### Cache Eviction Policies (10%)

**Q: What happens when cache is full?**

**1. LRU (Least Recently Used):**
```go
// Remove item that hasn't been accessed longest
type LRUCache struct {
    capacity int
    cache    map[string]*Node
    list     *List  // Doubly linked list
}

// When accessed, move to front
// When full, remove from back
```

**2. LFU (Least Frequently Used):**
```
Track access count
Remove item with lowest count
```

**3. FIFO (First In First Out):**
```
Remove oldest item added
Like a queue
```

**4. TTL (Time To Live):**
```redis
# Set expiration
SET session:abc123 "data" EX 3600  # Expires in 1 hour

# Check TTL
TTL session:abc123
```

**5. Random:**
```
Remove random item
Fast, unpredictable
```

**Interview Q: Which eviction policy to choose?**
- **LRU**: General purpose, most common
- **LFU**: When some items always popular
- **TTL**: Session data, temporary data
- **FIFO**: Simple, predictable

---

### Caching Best Practices

**1. Cache Invalidation:**
```go
// Invalidate on update
func UpdateUser(userID int, data User) error {
    // Update database
    err := db.UpdateUser(userID, data)
    if err != nil {
        return err
    }
    
    // Invalidate cache
    cacheKey := fmt.Sprintf("user:%d", userID)
    rdb.Del(ctx, cacheKey)
    
    return nil
}
```

**2. Cache Stampede Prevention:**
```go
// Use mutex to prevent multiple DB queries
var mu sync.Mutex

func GetData(key string) (Data, error) {
    // Check cache
    if cached, exists := cache.Get(key); exists {
        return cached, nil
    }
    
    // Lock to prevent stampede
    mu.Lock()
    defer mu.Unlock()
    
    // Double-check cache
    if cached, exists := cache.Get(key); exists {
        return cached, nil
    }
    
    // Query DB
    data := db.Get(key)
    cache.Set(key, data, 1*time.Hour)
    return data, nil
}
```

**3. Cache Warming:**
```go
// Pre-populate cache with popular data
func WarmCache() {
    popularUsers := db.GetPopularUsers(100)
    for _, user := range popularUsers {
        cacheKey := fmt.Sprintf("user:%d", user.ID)
        data, _ := json.Marshal(user)
        rdb.Set(ctx, cacheKey, data, time.Hour)
    }
}
```

**Interview Q: Two hardest things in computer science?**
1. Cache invalidation
2. Naming things
3. Off-by-one errors

---

### Types
**Q: Caching strategies?**
- **In-Memory**: Redis, Memcached (fast access)
- **Distributed**: Shared across servers
- **CDN**: Cache static content near users

### Eviction Policies
- **LRU**: Least Recently Used
- **LFU**: Least Frequently Used
- **FIFO**: First In First Out
- **TTL**: Time To Live

---

## 8. Docker (7%)

### Docker Concepts (15%)

**Q: What is Docker?**
- Containerization platform
- Packages app + dependencies into container
- Runs consistently anywhere (dev, staging, production)
- Lightweight alternative to VMs

**Container vs Virtual Machine:**
```
Virtual Machine:
[App A] [App B]
[Guest OS] [Guest OS]
[Hypervisor]
[Host OS]
[Hardware]

Docker Container:
[App A] [App B]
[Docker Engine]
[Host OS]
[Hardware]

Containers: Lightweight, fast startup, share OS kernel
VMs: Heavy, slow startup, full OS isolation
```

**Key Concepts:**
- **Image**: Blueprint/template for container
- **Container**: Running instance of image
- **Dockerfile**: Instructions to build image
- **Registry**: Storage for images (Docker Hub)
- **Volume**: Persistent data storage
- **Network**: Connect containers

**Common Docker Commands:**
```bash
# Images
docker images                    # List images
docker pull nginx:latest        # Download image
docker build -t myapp:1.0 .     # Build image
docker rmi image-id             # Remove image
docker tag myapp:1.0 user/myapp:1.0  # Tag image

# Containers
docker ps                       # List running containers
docker ps -a                    # List all containers
docker run -d --name web nginx  # Run container
docker run -it ubuntu bash      # Interactive mode
docker start container-id       # Start stopped container
docker stop container-id        # Stop container
docker rm container-id          # Remove container
docker exec -it web bash        # Execute command in container
docker logs web                 # View logs
docker logs -f web              # Follow logs

# System
docker system df                # Disk usage
docker system prune             # Clean up
docker inspect container-id     # Detailed info
docker stats                    # Resource usage
```

**Scenario: Debug container that won't start**
```bash
# 1. Check logs
docker logs container-name

# 2. Try running interactively
docker run -it myapp sh

# 3. Check if port is already in use
docker ps | grep 8080

# 4. Inspect container config
docker inspect container-name

# 5. Override entrypoint for debugging
docker run -it --entrypoint sh myapp
```

### Dockerfile (15%)

**Dockerfile Instructions:**
```dockerfile
# Base image
FROM node:16-alpine

# Set working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy application code
COPY . .

# Expose port
EXPOSE 3000

# Set environment variables
ENV NODE_ENV=production

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001
USER nodejs

# Start command
CMD ["node", "server.js"]
```

**Best Practices:**
```dockerfile
# ✅ GOOD - Multi-stage build
FROM node:16 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:16-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package*.json ./
RUN npm ci --only=production
CMD ["node", "dist/server.js"]

# Benefits:
# - Smaller image (only production files)
# - Faster builds (cache dependencies)
# - More secure (no build tools in production)
```

**Common Patterns:**
```dockerfile
# Go application
FROM golang:1.20 AS builder
WORKDIR /app
COPY go.* ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o main .

FROM alpine:latest
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=builder /app/main .
EXPOSE 8080
CMD ["./main"]

# Python application
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**Scenario: Optimize large Docker image**
```dockerfile
# ❌ BAD - 1.2GB image
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3 python3-pip
COPY requirements.txt .
RUN pip3 install -r requirements.txt
COPY . .
CMD ["python3", "app.py"]

# ✅ GOOD - 150MB image
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]

# ✅ BETTER - 50MB image (multi-stage)
FROM python:3.11 AS builder
WORKDIR /app
RUN pip install --user -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

### Docker Compose (15%)

**docker-compose.yml Structure:**
```yaml
version: '3.8'

services:
  # Web application
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DB_HOST=db
      - DB_PORT=5432
    depends_on:
      - db
      - redis
    volumes:
      - ./logs:/app/logs
    networks:
      - app-network
    restart: unless-stopped

  # Database
  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myuser"]
      interval: 10s
      timeout: 5s
      retries: 5

  # Cache
  redis:
    image: redis:7-alpine
    networks:
      - app-network
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data

  # Nginx reverse proxy
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - web
    networks:
      - app-network

volumes:
  postgres-data:
  redis-data:

networks:
  app-network:
    driver: bridge
```

**Docker Compose Commands:**
```bash
# Start all services
docker-compose up
docker-compose up -d              # Detached mode

# Build and start
docker-compose up --build

# Stop services
docker-compose stop
docker-compose down               # Stop and remove containers
docker-compose down -v            # Also remove volumes

# View logs
docker-compose logs
docker-compose logs -f            # Follow logs
docker-compose logs web           # Specific service

# Execute commands
docker-compose exec web sh        # Shell in running container
docker-compose exec db psql -U myuser -d mydb

# Scale services
docker-compose up -d --scale web=3

# List services
docker-compose ps
```

**Real-World Example: Full-Stack App**
```yaml
version: '3.8'

services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    environment:
      - REACT_APP_API_URL=http://localhost:8000
    volumes:
      - ./frontend:/app
      - /app/node_modules

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - DB_HOST=postgres
      - REDIS_HOST=redis
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14
    environment:
      POSTGRES_DB: appdb
      POSTGRES_USER: appuser
      POSTGRES_PASSWORD: apppass
    volumes:
      - pg-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine

  adminer:
    image: adminer
    ports:
      - "8080:8080"

volumes:
  pg-data:
```

**Scenario: Dev environment with Docker Compose**
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  app:
    build:
      context: .
      target: development    # Multi-stage build
    volumes:
      - .:/app              # Mount source code
      - /app/node_modules   # Don't override node_modules
    environment:
      - NODE_ENV=development
      - DEBUG=app:*
    command: npm run dev    # Hot reload

  db:
    image: postgres:14
    ports:
      - "5432:5432"         # Expose for local tools
    environment:
      POSTGRES_PASSWORD: dev

# Run with:
# docker-compose -f docker-compose.dev.yml up
```

**Scenario: Troubleshoot container networking**
```bash
# 1. Check if containers are on same network
docker network ls
docker network inspect bridge

# 2. Test connection between containers
docker-compose exec web ping db

# 3. Check if service is listening
docker-compose exec db netstat -tulpn

# 4. View service DNS resolution
docker-compose exec web nslookup db

# 5. Check environment variables
docker-compose exec web env | grep DB
```

---

## 9. API - REST (15%)

### RESTful Concepts (15%)

**Q: What is REST?**
- **RE**presentational **S**tate **T**ransfer
- Architectural style for designing APIs
- Uses HTTP methods for operations
- Resource-based URLs
- Stateless (each request independent)

**REST Principles:**
1. **Client-Server**: Separation of concerns
2. **Stateless**: No client context stored on server
3. **Cacheable**: Responses can be cached
4. **Uniform Interface**: Standard HTTP methods
5. **Layered System**: Client doesn't know if connected to end server
6. **Code on Demand** (optional): Server can send executable code

**Resource-Based URLs:**
```
✅ GOOD (Nouns, representing resources):
/users              - Collection of users
/users/123          - Specific user
/users/123/orders   - User's orders
/products           - Collection of products
/products/456       - Specific product

❌ BAD (Verbs in URL):
/getUsers
/createUser
/deleteUser/123
/updateProduct
```

### HTTP Methods (15%)

**CRUD Operations:**
```
CREATE  → POST   /users         → 201 Created
READ    → GET    /users         → 200 OK
READ    → GET    /users/1       → 200 OK
UPDATE  → PUT    /users/1       → 200 OK
UPDATE  → PATCH  /users/1       → 200 OK
DELETE  → DELETE /users/1       → 204 No Content
```

**Detailed Examples:**

**GET** - Retrieve resources
```http
GET /api/users HTTP/1.1
Host: example.com

Response: 200 OK
[
  {"id": 1, "name": "John", "email": "john@example.com"},
  {"id": 2, "name": "Jane", "email": "jane@example.com"}
]

GET /api/users/1 HTTP/1.1
Response: 200 OK
{"id": 1, "name": "John", "email": "john@example.com"}

GET /api/users/999 HTTP/1.1
Response: 404 Not Found
{"error": "User not found"}
```

**POST** - Create new resource
```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Alice",
  "email": "alice@example.com"
}

Response: 201 Created
Location: /api/users/3
{
  "id": 3,
  "name": "Alice",
  "email": "alice@example.com",
  "created_at": "2024-01-15T10:30:00Z"
}
```

**PUT** - Full update (replace entire resource)
```http
PUT /api/users/1 HTTP/1.1
Content-Type: application/json

{
  "name": "John Updated",
  "email": "john.new@example.com",
  "phone": "555-1234"
}

Response: 200 OK
{
  "id": 1,
  "name": "John Updated",
  "email": "john.new@example.com",
  "phone": "555-1234"
}
```

**PATCH** - Partial update (modify specific fields)
```http
PATCH /api/users/1 HTTP/1.1
Content-Type: application/json

{
  "email": "john.updated@example.com"
}

Response: 200 OK
{
  "id": 1,
  "name": "John",  // Unchanged
  "email": "john.updated@example.com"
}
```

**DELETE** - Remove resource
```http
DELETE /api/users/1 HTTP/1.1

Response: 204 No Content
(Empty body)
```

### Status Codes (15%)

**Success (2xx):**
```
200 OK              - Request succeeded
201 Created         - Resource created successfully
202 Accepted        - Async request accepted
204 No Content      - Success but no data to return
```

**Redirection (3xx):**
```
301 Moved Permanently - Resource permanently moved
302 Found            - Temporary redirect
304 Not Modified     - Use cached version
```

**Client Errors (4xx):**
```
400 Bad Request      - Invalid request syntax/data
401 Unauthorized     - Authentication required
403 Forbidden        - Authenticated but no permission
404 Not Found        - Resource doesn't exist
405 Method Not Allowed - HTTP method not supported
409 Conflict         - Request conflicts with current state
422 Unprocessable Entity - Validation error
429 Too Many Requests - Rate limit exceeded
```

**Server Errors (5xx):**
```
500 Internal Server Error - Generic server error
502 Bad Gateway      - Invalid response from upstream
503 Service Unavailable - Server temporarily unavailable
504 Gateway Timeout  - Upstream server timeout
```

**Scenario: Design RESTful API for Blog**
```
Resources: Users, Posts, Comments

GET    /users               - List all users
GET    /users/1             - Get user by ID
POST   /users               - Create user
PUT    /users/1             - Update user
DELETE /users/1             - Delete user

GET    /posts               - List all posts
GET    /posts/1             - Get post by ID
POST   /posts               - Create post
PUT    /posts/1             - Update post
DELETE /posts/1             - Delete post
GET    /posts?author=1      - Filter posts by author
GET    /posts?sort=date     - Sort posts

GET    /posts/1/comments    - Get comments for post
POST   /posts/1/comments    - Create comment on post
PUT    /comments/5          - Update comment
DELETE /comments/5          - Delete comment

GET    /users/1/posts       - Get posts by user
```

### CORS (10%)

**Q: What is CORS?**
- **C**ross-**O**rigin **R**esource **S**haring
- Security feature implemented by browsers
- Controls which domains can access your API
- Prevents malicious websites from accessing your API

**How CORS Works:**
```
1. Browser makes request from example.com to api.example.com
2. Browser checks if cross-origin
3. Browser sends preflight OPTIONS request
4. Server responds with allowed origins
5. If allowed, browser sends actual request
```

**CORS Headers:**
```http
# Server response headers
Access-Control-Allow-Origin: https://example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Content-Type, Authorization
Access-Control-Max-Age: 3600
Access-Control-Allow-Credentials: true
```

**CORS in Practice:**
```javascript
// Node.js/Express
app.use((req, res, next) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  next();
});

// Or use cors package
const cors = require('cors');
app.use(cors({
  origin: 'https://example.com',
  methods: ['GET', 'POST'],
  credentials: true
}));
```

**Scenario: Frontend can't access API (CORS error)**
```
Error in browser console:
"Access to fetch at 'http://api.example.com/users' from origin 
'http://localhost:3000' has been blocked by CORS policy"

Solutions:
1. Add CORS headers on server
2. Configure allowed origins
3. Use proxy in development
4. Enable credentials if needed

Backend fix (Go):
w.Header().Set("Access-Control-Allow-Origin", "*")
w.Header().Set("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE")
w.Header().Set("Access-Control-Allow-Headers", "Content-Type")
```

### Rate Limiting (10%)

**Q: What is rate limiting?**
- Restrict number of requests per time period
- Prevents abuse and DDoS
- Ensures fair usage

**Common Strategies:**
```
1. Fixed Window:    100 requests/minute
2. Sliding Window:  100 requests/rolling minute
3. Token Bucket:    Burst allowed, refills over time
4. Leaky Bucket:    Constant rate
```

**Rate Limit Headers:**
```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1642080000

Response when exceeded:
HTTP/1.1 429 Too Many Requests
Retry-After: 60
{
  "error": "Rate limit exceeded",
  "retry_after": 60
}
```

**Implementation Example (pseudocode):**
```
IF requests_in_window[user] >= limit:
    RETURN 429 Too Many Requests
ELSE:
    requests_in_window[user]++
    RETURN 200 OK
```

### Documentation - Swagger/OpenAPI (10%)

**Q: Why document APIs?**
- Helps developers understand endpoints
- Shows request/response formats
- Provides examples
- Enables testing

**OpenAPI/Swagger Example:**
```yaml
openapi: 3.0.0
info:
  title: User API
  version: 1.0.0

paths:
  /users:
    get:
      summary: List all users
      responses:
        '200':
          description: Successful response
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/User'
    post:
      summary: Create user
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UserInput'
      responses:
        '201':
          description: User created

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
    UserInput:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
        email:
          type: string
```

**Swagger UI Benefits:**
- Interactive documentation
- Try API endpoints in browser
- Auto-generated from code
- Industry standard

### RESTful Best Practices

**1. Use Nouns, Not Verbs:**
```
✅ GET /users
❌ GET /getUsers
```

**2. Use Plural Nouns:**
```
✅ /users, /products, /orders
❌ /user, /product, /order
```

**3. Nested Resources:**
```
✅ /users/1/orders
❌ /orders?userId=1 (also okay, but less RESTful)
```

**4. Filtering, Sorting, Pagination:**
```
GET /users?status=active&sort=created_at&page=2&limit=20
```

**5. Versioning:**
```
/api/v1/users
/api/v2/users
```

**6. Consistent Naming:**
```
✅ snake_case or camelCase (be consistent)
created_at, createdAt
❌ Mixed: created_at, updatedDate
```

**7. Error Responses:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid email format",
    "details": {
      "field": "email",
      "value": "invalid-email"
    }
  }
}
```

**Real-World API Example:**
```
E-commerce API:

Products:
GET    /products                    - List products
GET    /products/123                - Get product details
GET    /products?category=electronics&min_price=100&sort=price
POST   /products                    - Create product (admin)
PUT    /products/123                - Update product (admin)
DELETE /products/123                - Delete product (admin)

Cart:
GET    /cart                        - Get current user's cart
POST   /cart/items                  - Add item to cart
PUT    /cart/items/456              - Update quantity
DELETE /cart/items/456              - Remove from cart

Orders:
GET    /orders                      - List user's orders
GET    /orders/789                  - Get order details
POST   /orders                      - Create order (checkout)
GET    /orders/789/status           - Track order

Authentication:
POST   /auth/login                  - Login
POST   /auth/logout                 - Logout
POST   /auth/refresh                - Refresh token
POST   /auth/register               - Register new user
```

---

## 10. JSON Schema (10%)

**Q: What is JSON Schema?**
- Standard for describing JSON data structure
- Validates JSON documents
- Documents expected format
- Used in API contracts

**Basic JSON Schema:**
```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User",
  "type": "object",
  "required": ["name", "email"],
  "properties": {
    "id": {
      "type": "integer",
      "description": "User ID"
    },
    "name": {
      "type": "string",
      "minLength": 1,
      "maxLength": 100
    },
    "email": {
      "type": "string",
      "format": "email"
    },
    "age": {
      "type": "integer",
      "minimum": 0,
      "maximum": 120
    },
    "role": {
      "type": "string",
      "enum": ["admin", "user", "guest"]
    }
  }
}
```

**Validation Example:**
```json
// Valid JSON
{
  "id": 1,
  "name": "John Doe",
  "email": "john@example.com",
  "age": 30,
  "role": "user"
}

// Invalid JSON (missing required field)
{
  "id": 1,
  "name": "John Doe"
  // Error: "email" is required
}
```

---

## 11. GraphQL (10%)

**Q: What is GraphQL?**
- Query language for APIs
- Single endpoint
- Request exactly what you need
- Strongly typed

### Schema Definition (10%)
```graphql
# Define types
type User {
  id: ID!
  name: String!
  email: String!
  posts: [Post]
  createdAt: String
}

type Post {
  id: ID!
  title: String!
  content: String
  author: User!
  comments: [Comment]
}

type Comment {
  id: ID!
  text: String!
  author: User!
}

# Define queries
type Query {
  user(id: ID!): User
  users: [User]
  post(id: ID!): Post
  posts(limit: Int): [Post]
}

# Define mutations
type Mutation {
  createUser(name: String!, email: String!): User
  updateUser(id: ID!, name: String, email: String): User
  deleteUser(id: ID!): Boolean
  createPost(title: String!, content: String!, authorId: ID!): Post
}
```

### Query (10%)
```graphql
# Simple query
query {
  user(id: "1") {
    name
    email
  }
}

# Nested query
query {
  user(id: "1") {
    name
    email
    posts {
      title
      comments {
        text
        author {
          name
        }
      }
    }
  }
}

# Query with variables
query GetUser($userId: ID!) {
  user(id: $userId) {
    name
    email
  }
}

# Variables:
{
  "userId": "1"
}

# Multiple queries (aliases)
query {
  user1: user(id: "1") {
    name
  }
  user2: user(id: "2") {
    name
  }
}

# Fragments (reusable)
fragment UserInfo on User {
  id
  name
  email
}

query {
  user(id: "1") {
    ...UserInfo
    posts {
      title
    }
  }
}
```

### Mutation (10%)
```graphql
# Create user
mutation {
  createUser(name: "John", email: "john@example.com") {
    id
    name
    email
  }
}

# Update user
mutation {
  updateUser(id: "1", name: "John Updated") {
    id
    name
  }
}

# Delete user
mutation {
  deleteUser(id: "1")
}

# With variables
mutation CreateUser($name: String!, $email: String!) {
  createUser(name: $name, email: $email) {
    id
    name
    email
  }
}

# Variables:
{
  "name": "Jane",
  "email": "jane@example.com"
}
```

**GraphQL vs REST:**

| Feature | GraphQL | REST |
|---------|---------|------|
| **Endpoints** | Single `/graphql` | Multiple `/users`, `/posts` |
| **Data fetching** | Request exactly what you need | Fixed response structure |
| **Over-fetching** | No | Yes (get more than needed) |
| **Under-fetching** | No | Yes (need multiple requests) |
| **Versioning** | Not needed (additive changes) | Often needed (v1, v2) |
| **Learning curve** | Steeper | Simpler |
| **Caching** | Complex | Simple (HTTP caching) |
| **File uploads** | Tricky | Straightforward |

**Scenario: Why choose GraphQL?**
```
Mobile app with slow network:
- Need user name, profile picture, and post count
- REST: 3 API calls
  GET /users/1        → {id, name, email, created_at, ...}
  GET /users/1/avatar → {url, size, ...}
  GET /users/1/posts  → [{id, title, ...}, ...]
  
- GraphQL: 1 API call
  query {
    user(id: "1") {
      name
      avatar {
        url
      }
      postCount
    }
  }
  
Result: Faster load, less data transfer
```

---

## 12. gRPC (10%)

**Q: What is gRPC?**
- Google Remote Procedure Call
- High-performance RPC framework
- Uses HTTP/2
- Binary protocol (Protocol Buffers)
- Strongly typed

### Protocol Buffers (10%)

**Define .proto file:**
```protobuf
syntax = "proto3";

package user;

// Message definition
message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
  int32 age = 4;
}

message UserRequest {
  int32 id = 1;
}

message UserListRequest {
  int32 page = 1;
  int32 limit = 2;
}

message UserListResponse {
  repeated User users = 1;
  int32 total = 2;
}

// Service definition
service UserService {
  // Unary call (request-response)
  rpc GetUser(UserRequest) returns (User);
  
  // Server streaming
  rpc ListUsers(UserListRequest) returns (stream User);
  
  // Client streaming
  rpc CreateUsers(stream User) returns (UserListResponse);
  
  // Bidirectional streaming
  rpc Chat(stream Message) returns (stream Message);
}
```

**Generate code:**
```bash
# Generate Go code
protoc --go_out=. --go-grpc_out=. user.proto

# Generate Python code
protoc --python_out=. --grpc_python_out=. user.proto
```

### Unary Calls (10%)

**Server Implementation (Go):**
```go
type server struct {
    pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.User, error) {
    // Fetch user from database
    user := &pb.User{
        Id:    req.Id,
        Name:  "John Doe",
        Email: "john@example.com",
        Age:   30,
    }
    return user, nil
}

func main() {
    lis, _ := net.Listen("tcp", ":50051")
    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{})
    s.Serve(lis)
}
```

**Client Implementation (Go):**
```go
func main() {
    conn, _ := grpc.Dial("localhost:50051", grpc.WithInsecure())
    defer conn.Close()
    
    client := pb.NewUserServiceClient(conn)
    
    req := &pb.UserRequest{Id: 1}
    user, err := client.GetUser(context.Background(), req)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Printf("User: %s, Email: %s\n", user.Name, user.Email)
}
```

**gRPC vs REST:**

| Feature | gRPC | REST |
|---------|------|------|
| **Protocol** | HTTP/2, Binary (Protobuf) | HTTP/1.1, Text (JSON) |
| **Performance** | Faster, smaller payload | Slower, larger payload |
| **Streaming** | Built-in (bidirectional) | Limited (SSE, WebSocket) |
| **Browser support** | Limited (needs proxy) | Native |
| **Type safety** | Strong (code generation) | Weak |
| **Human readable** | No | Yes |
| **Use cases** | Microservices, mobile | Web APIs, public APIs |

**When to use gRPC:**
- Microservices communication
- Real-time bidirectional streaming
- Performance critical applications
- Polyglot environments (multiple languages)
- Internal APIs

**When to use REST:**
- Public APIs
- Browser-based applications
- Simple CRUD operations
- Third-party integrations

---

## 13. API Gateway (10%)

**Q: What is API Gateway?**
- Single entry point for all client requests
- Acts as reverse proxy
- Centralized control for microservices

### Functions (5%)

**1. Routing/Proxy:**
```
Client Request → API Gateway → Microservice

GET /api/users → Gateway → User Service (port 8001)
GET /api/orders → Gateway → Order Service (port 8002)
GET /api/products → Gateway → Product Service (port 8003)
```

**2. Authentication & Authorization:**
```
1. Client sends request with JWT token
2. Gateway validates token
3. If valid, forward to service
4. If invalid, return 401
```

**3. Rate Limiting:**
```
Track requests per client:
- Allow 100 requests/minute per API key
- Reject with 429 if exceeded
```

**4. Load Balancing:**
```
Gateway distributes requests:
User Service Instance 1 (25% traffic)
User Service Instance 2 (25% traffic)
User Service Instance 3 (25% traffic)
User Service Instance 4 (25% traffic)
```

**5. Request/Response Transformation:**
```
Client sends XML → Gateway converts to JSON → Microservice
Microservice returns JSON → Gateway converts to XML → Client
```

**6. Caching:**
```
GET /products
↓
Check cache
↓
Cache hit → Return cached response
Cache miss → Forward to service → Cache response → Return
```

### Request/Response Flow (5%)

**Without API Gateway:**
```
Mobile App → Auth Service (auth.example.com)
          → User Service (users.example.com)
          → Order Service (orders.example.com)
          → Product Service (products.example.com)

Problems:
- Multiple hostnames to manage
- CORS configuration for each
- Authentication in each service
- Rate limiting in each service
```

**With API Gateway:**
```
Mobile App → API Gateway (api.example.com)
                 ↓
          Routes to appropriate service
                 ↓
        [Auth] [Users] [Orders] [Products]

Benefits:
- Single hostname
- Centralized auth
- Centralized rate limiting
- Service discovery
- Monitoring & logging
```

### Request/Response + Protocol Translation (10%)

**Example: REST to gRPC:**
```
1. Client sends REST request:
   POST /api/users
   {
     "name": "John",
     "email": "john@example.com"
   }

2. Gateway translates to gRPC:
   CreateUser(UserRequest{
     name: "John",
     email: "john@example.com"
   })

3. Microservice (gRPC) processes request

4. Gateway translates gRPC response back to JSON:
   {
     "id": 123,
     "name": "John",
     "email": "john@example.com"
   }
```

**Popular API Gateways:**
- Kong
- AWS API Gateway
- NGINX
- Traefik
- Envoy
- Zuul (Netflix)

**Configuration Example (Kong):**
```yaml
services:
  - name: user-service
    url: http://users:8001
    routes:
      - name: user-route
        paths:
          - /api/users
    plugins:
      - name: rate-limiting
        config:
          minute: 100
      - name: jwt
      - name: cors
```

**Scenario: Microservices Architecture**
```
Without Gateway:
Frontend → Service A (3001)
        → Service B (3002)
        → Service C (3003)

Problems:
- Frontend knows all service URLs
- Complex to add/remove services
- Authentication scattered

With Gateway:
Frontend → Gateway (80)
              ↓
         [Service A] [Service B] [Service C]

Benefits:
- Frontend only knows gateway URL
- Easy to add services
- Centralized security
- Monitoring & logging
```

---

## 14. Serverless (25%)

### Concepts (25%)

**Q: What is serverless?**
- No server management (infrastructure handled by provider)
- Event-driven architecture
- Pay per execution (not per hour)
- Auto-scaling
- Stateless functions

**Benefits:**
- ✅ No server management
- ✅ Automatic scaling
- ✅ Pay only for what you use
- ✅ Faster time to market
- ✅ Built-in high availability

**Drawbacks:**
- ❌ Cold start latency
- ❌ Vendor lock-in
- ❌ Limited execution time
- ❌ Difficult debugging
- ❌ Stateless (need external storage)

**Use Cases:**
- API backends
- Data processing
- Scheduled tasks (cron jobs)
- Event-driven workflows
- Chatbots
- IoT backends

### OpenFaaS (25%)

**Q: What is OpenFaaS?**
- Open-source serverless framework
- Run functions on Kubernetes/Docker
- Any language supported
- Self-hosted (no vendor lock-in)

**Function Structure:**
```yaml
# stack.yml
provider:
  name: openfaas
  gateway: http://localhost:8080

functions:
  hello-python:
    lang: python3
    handler: ./hello-python
    image: hello-python:latest
    environment:
      write_timeout: 10s
      read_timeout: 10s
    secrets:
      - api-key
```

**Create Function:**
```bash
# Create new function
faas-cli new hello-python --lang python3

# Build
faas-cli build -f stack.yml

# Deploy
faas-cli deploy -f stack.yml

# Invoke
curl http://localhost:8080/function/hello-python -d "Hello"
```

**Handler Example (Python):**
```python
# hello-python/handler.py
def handle(req):
    """Handle request"""
    return f"Hello, {req}!"
```

**Handler Example (Go):**
```go
// handler.go
package function

import (
    "fmt"
)

func Handle(req []byte) string {
    return fmt.Sprintf("Hello, %s!", string(req))
}
```

### Knative (5%)

**Q: What is Knative?**
- Kubernetes-based serverless platform
- Developed by Google
- Auto-scaling to zero
- Event-driven

**Knative Service:**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: hello
spec:
  template:
    spec:
      containers:
        - image: gcr.io/knative-samples/helloworld-go
          env:
            - name: TARGET
              value: "World"
```

**Deploy:**
```bash
kubectl apply -f service.yaml

# Get URL
kubectl get ksvc hello
```

**Features:**
- Scale to zero (no requests = no pods)
- Revision management
- Traffic splitting
- Built-in metrics

**Serverless Comparison:**

| Platform | Provider | Language Support | Execution Time | Pricing |
|----------|----------|------------------|----------------|---------|
| **AWS Lambda** | AWS | Many | 15 min | Per invocation |
| **Google Cloud Functions** | Google | Many | 9 min | Per invocation |
| **Azure Functions** | Microsoft | Many | Unlimited | Per invocation |
| **OpenFaaS** | Self-hosted | Any | Configurable | Infrastructure only |
| **Knative** | Kubernetes | Any | Configurable | Infrastructure only |

**Scenario: Choose serverless platform**
```
Requirement: Process image uploads

Option 1: AWS Lambda
- Quick to deploy
- Auto-scaling
- Vendor lock-in
- 15 min timeout sufficient

Option 2: OpenFaaS on Kubernetes
- Control over infrastructure
- No vendor lock-in
- More setup required
- Unlimited timeout

Choose AWS Lambda if:
- Want simplicity
- Small to medium scale
- Fast time to market

Choose OpenFaaS if:
- Need control
- Avoid vendor lock-in
- Already use Kubernetes
```

---

## Quick Interview Tips

### Common Questions

**1. "Tell me about a technical challenge you faced"**
- Describe problem
- Your approach
- Solution implemented
- Results/learnings

**2. "How do you optimize database queries?"**
- Add indexes
- Optimize JOINs
- Use EXPLAIN ANALYZE
- Cache results
- Denormalize if needed

**3. "Explain your development workflow"**
- Git feature branch
- Write code + tests
- Local testing
- Pull request
- Code review
- CI/CD pipeline
- Deploy

**4. "How do you debug production issues?"**
- Check logs
- Monitor metrics
- Reproduce locally
- Use profiling tools
- Add logging
- Test fix
- Deploy

### System Design Questions

**"Design a URL shortener"**
```
Requirements:
- Shorten long URLs
- Redirect to original URL
- Track analytics (clicks, countries)
- Handle millions of requests

Components:
1. API Server:
   - POST /shorten → Create short URL
   - GET /{shortCode} → Redirect

2. Database (PostgreSQL):
   - urls table: id, short_code, long_url, user_id, created_at
   - clicks table: id, url_id, ip, country, clicked_at

3. Cache (Redis):
   - Cache popular URLs for fast lookups
   - Key: short_code → Value: long_url

4. Hash Generation:
   - Base62 encoding (a-z, A-Z, 0-9)
   - 7 characters = 62^7 = 3.5 trillion combinations

Flow:
1. User submits long URL
2. Generate unique hash (check DB for collisions)
3. Store in database
4. Return short URL: example.com/abc123
5. On redirect: Check cache → If miss, query DB → Cache it → Redirect

Scale:
- Load balancer for API servers
- Database replication (read replicas)
- Sharding by short_code
- CDN for static content
```

**"Design a chat application"**
```
Requirements:
- Real-time messaging
- Group chats
- Online status
- Message history
- File sharing

Components:
1. WebSocket Server:
   - Maintain persistent connections
   - Broadcast messages to recipients
   - Handle presence (online/offline)

2. Message Queue (Kafka/RabbitMQ):
   - Decouple message processing
   - Ensure message delivery
   - Handle offline users

3. Database:
   - PostgreSQL: Users, chat rooms, metadata
   - MongoDB: Message storage (document DB)
   - Redis: Online users, typing indicators

4. File Storage (S3):
   - Store images, videos, files
   - CDN for fast delivery

5. API Gateway:
   - REST API for non-real-time operations
   - WebSocket for real-time

Architecture:
┌──────────┐
│  Client  │
└────┬─────┘
     │ WebSocket
┌────▼──────────┐
│   Gateway     │
└────┬──────────┘
     │
┌────▼──────────┐     ┌──────────┐
│  WS Server    │────▶│  Kafka   │
└───────────────┘     └────┬─────┘
                           │
                      ┌────▼─────┐
                      │ Database │
                      └──────────┘

Message Flow:
1. User A sends message
2. WS server receives
3. Publish to Kafka topic
4. Save to database
5. If User B online → Send via WebSocket
6. If User B offline → Store for later delivery
```

**"Design Instagram Feed"**
```
Requirements:
- Show posts from followed users
- Chronological order
- Pagination
- Handle millions of users

Components:
1. Feed Service:
   - Generate feed when user opens app
   - Pull from multiple sources

2. Post Service:
   - Store posts
   - Media processing

3. Cache (Redis):
   - Cache user feeds
   - Cache popular posts

4. Database:
   - Users: id, username, followers_count
   - Posts: id, user_id, image_url, caption, created_at
   - Follows: follower_id, followee_id
   - Likes: user_id, post_id

Approaches:
1. Fan-out on Write (Push):
   - When user posts, write to all followers' feeds
   - Pro: Fast reads
   - Con: Slow writes for celebrities

2. Fan-out on Read (Pull):
   - Generate feed on request
   - Pro: Fast writes
   - Con: Slow reads

3. Hybrid:
   - Use push for regular users
   - Use pull for celebrities
   - Cache aggressively

Feed Generation:
1. Get list of followed users
2. Query recent posts from each
3. Merge and sort by time
4. Cache result for 5 minutes
5. Return paginated response
```

**"Design Rate Limiter"**
```
Requirements:
- Limit requests per user/IP
- Different limits for different endpoints
- Return 429 when exceeded

Algorithms:
1. Token Bucket:
   - Bucket holds tokens
   - Each request consumes 1 token
   - Tokens refill at constant rate
   - Allow bursts

2. Fixed Window:
   - Count requests in time window
   - Reset counter at window boundary
   - Simple but has edge case

3. Sliding Window:
   - Track requests with timestamps
   - Count requests in rolling window
   - More accurate

Implementation (Redis):
key: user:123:api:/users
value: request_count
expire: 60 seconds

Algorithm:
1. Check Redis for key
2. If not exists → Create with count=1
3. If exists → Increment count
4. If count > limit → Return 429
5. If count <= limit → Allow request
```

---

## Practice Questions

### OS
1. **Q: How do you find a process using a specific port?**
   ```bash
   # Linux/Mac
   lsof -i :8080
   netstat -tulpn | grep 8080
   
   # Windows
   netstat -ano | findstr :8080
   ```

2. **Q: What's the difference between soft and hard links?**
   - **Hard link**: Another name for same file (same inode)
     - `ln file.txt hardlink.txt`
     - Both point to same data
     - Delete one, other still works
   - **Soft link**: Shortcut/pointer to file (different inode)
     - `ln -s file.txt softlink.txt`
     - Points to filename, not data
     - Delete original, link breaks

3. **Q: Explain process scheduling**
   - OS decides which process runs when
   - Algorithms:
     - **FCFS**: First Come First Serve
     - **Round Robin**: Time slices
     - **Priority**: High priority first
     - **SJF**: Shortest Job First

4. **Q: What's the difference between kill -9 and kill -15?**
   - `kill -15` (SIGTERM): Graceful shutdown, process can clean up
   - `kill -9` (SIGKILL): Force kill immediately, no cleanup

5. **Q: What happens when system runs out of memory?**
   - OOM (Out of Memory) killer activated
   - Kills processes to free memory
   - Prioritizes by memory usage and importance

### Networking
1. **Q: What happens when you type a URL in browser?**
   ```
   1. Browser checks cache
   2. DNS lookup: URL → IP address
   3. TCP 3-way handshake
   4. TLS handshake (if HTTPS)
   5. Browser sends HTTP request
   6. Server processes request
   7. Server sends response
   8. Browser renders page
   9. Browser fetches additional resources (CSS, JS, images)
   10. Page loads
   ```

2. **Q: Explain the 3-way TCP handshake**
   ```
   Client                    Server
     |-------- SYN -------->   |  (Want to connect)
     |<----- SYN-ACK -------   |  (OK, acknowledged)
     |-------- ACK -------->   |  (Thanks, let's start)
     |=== Connection Open ===  |
   ```

3. **Q: How does HTTPS work?**
   ```
   1. Client requests HTTPS connection
   2. Server sends SSL certificate (public key)
   3. Client verifies certificate with CA
   4. Client generates session key
   5. Client encrypts session key with server's public key
   6. Server decrypts with private key
   7. Both use session key for symmetric encryption
   8. All data encrypted during session
   ```

4. **Q: What's the difference between HTTP/1.1 and HTTP/2?**
   - **HTTP/1.1**: One request per connection, head-of-line blocking
   - **HTTP/2**: Multiplexing (multiple requests), header compression, server push

5. **Q: Explain DNS lookup process**
   ```
   1. Browser cache
   2. OS cache
   3. Router cache
   4. ISP DNS server
   5. Root DNS server
   6. TLD DNS server (.com)
   7. Authoritative DNS server
   ```

### Database
1. **Q: Design a schema for e-commerce**
   ```sql
   -- Users
   CREATE TABLE users (
       id SERIAL PRIMARY KEY,
       email VARCHAR(100) UNIQUE NOT NULL,
       password_hash VARCHAR(255) NOT NULL,
       name VARCHAR(100),
       created_at TIMESTAMP DEFAULT NOW()
   );
   
   -- Products
   CREATE TABLE products (
       id SERIAL PRIMARY KEY,
       name VARCHAR(200) NOT NULL,
       description TEXT,
       price DECIMAL(10,2) NOT NULL,
       stock INT DEFAULT 0,
       category_id INT REFERENCES categories(id)
   );
   
   -- Orders
   CREATE TABLE orders (
       id SERIAL PRIMARY KEY,
       user_id INT REFERENCES users(id),
       total DECIMAL(10,2),
       status VARCHAR(20),
       created_at TIMESTAMP DEFAULT NOW()
   );
   
   -- Order Items
   CREATE TABLE order_items (
       id SERIAL PRIMARY KEY,
       order_id INT REFERENCES orders(id),
       product_id INT REFERENCES products(id),
       quantity INT,
       price DECIMAL(10,2)
   );
   ```

2. **Q: Write a query to find 2nd highest salary**
   ```sql
   -- Method 1: Using LIMIT and OFFSET
   SELECT DISTINCT salary 
   FROM employees 
   ORDER BY salary DESC 
   LIMIT 1 OFFSET 1;
   
   -- Method 2: Using subquery
   SELECT MAX(salary) 
   FROM employees 
   WHERE salary < (SELECT MAX(salary) FROM employees);
   
   -- Method 3: Using DENSE_RANK
   SELECT salary
   FROM (
       SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) as rank
       FROM employees
   ) ranked
   WHERE rank = 2;
   ```

3. **Q: When to use NoSQL vs SQL?**
   
   **Use SQL when:**
   - Need ACID transactions
   - Complex queries with JOINs
   - Data has clear relationships
   - Schema is well-defined
   - Examples: Banking, e-commerce, ERP
   
   **Use NoSQL when:**
   - Need horizontal scaling
   - Flexible schema
   - High write throughput
   - Denormalized data is acceptable
   - Examples: Social media, real-time analytics, IoT

4. **Q: Explain database normalization**
   - Process of reducing redundancy
   - Split data into related tables
   - 1NF: Atomic values, no repeating groups
   - 2NF: No partial dependencies
   - 3NF: No transitive dependencies

5. **Q: What's a database index and when to use it?**
   - Data structure that improves query speed
   - Trade-off: Faster reads, slower writes
   - Use on: WHERE, JOIN, ORDER BY columns
   - Don't use on: Small tables, frequently updated columns

### API
1. **Q: Design RESTful API for blog**
   ```
   Users:
   POST   /api/users             - Register
   POST   /api/auth/login        - Login
   GET    /api/users/me          - Get current user
   
   Posts:
   GET    /api/posts             - List all posts
   GET    /api/posts/{id}        - Get post
   POST   /api/posts             - Create post
   PUT    /api/posts/{id}        - Update post
   DELETE /api/posts/{id}        - Delete post
   
   Comments:
   GET    /api/posts/{id}/comments - Get comments
   POST   /api/posts/{id}/comments - Create comment
   DELETE /api/comments/{id}       - Delete comment
   ```

2. **Q: How to handle authentication in API?**
   
   **JWT (JSON Web Token):**
   ```
   1. User logs in with credentials
   2. Server validates, creates JWT
   3. Client stores JWT (localStorage/cookie)
   4. Client sends JWT in header: Authorization: Bearer <token>
   5. Server validates JWT on each request
   
   JWT Structure:
   Header.Payload.Signature
   {alg: "HS256"}.{user_id: 123, exp: ...}.{signature}
   ```
   
   **Session-based:**
   ```
   1. User logs in
   2. Server creates session, stores in Redis
   3. Server sends session ID in cookie
   4. Client sends cookie on each request
   5. Server validates session
   ```

3. **Q: What's rate limiting and why use it?**
   - Restrict number of requests per time period
   - **Why use:**
     - Prevent abuse/DDoS
     - Ensure fair usage
     - Control costs
     - Prevent resource exhaustion
   - **Implementation:** Track requests in Redis/memory
   - **Response:** 429 Too Many Requests

4. **Q: How to version APIs?**
   ```
   Method 1: URL versioning
   /api/v1/users
   /api/v2/users
   
   Method 2: Header versioning
   Accept: application/vnd.myapi.v2+json
   
   Method 3: Query parameter
   /api/users?version=2
   
   Best practice: URL versioning (most common, clear)
   ```

5. **Q: What's the difference between PUT and PATCH?**
   - **PUT**: Replace entire resource (idempotent)
   - **PATCH**: Update specific fields (partial update)
   ```
   PUT /users/1
   {name: "John", email: "john@example.com", age: 30}
   // Must send all fields
   
   PATCH /users/1
   {email: "john.new@example.com"}
   // Only send changed fields
   ```

### Docker & DevOps
1. **Q: What's the difference between Docker image and container?**
   - **Image**: Blueprint/template (read-only)
   - **Container**: Running instance of image (writable)
   - Analogy: Class vs Object

2. **Q: How to reduce Docker image size?**
   - Use alpine base images
   - Multi-stage builds
   - Remove unnecessary files
   - Use .dockerignore
   - Combine RUN commands

3. **Q: Explain Docker volumes**
   - Persist data outside container
   - Survive container deletion
   - Share data between containers
   ```bash
   docker run -v /host/path:/container/path myapp
   ```

### System Design
1. **Q: How to handle millions of concurrent users?**
   - Load balancers (distribute traffic)
   - Horizontal scaling (more servers)
   - Caching (Redis)
   - Database replication (read replicas)
   - CDN (static content)
   - Async processing (message queues)

2. **Q: How to ensure high availability?**
   - No single point of failure
   - Redundancy (multiple instances)
   - Health checks
   - Auto-scaling
   - Database replication
   - Multi-region deployment

3. **Q: Explain microservices vs monolith**
   
   **Monolith:**
   - Single codebase
   - Easier to develop initially
   - Harder to scale
   - One tech stack
   
   **Microservices:**
   - Multiple independent services
   - Each has own database
   - Scale independently
   - Different tech stacks possible
   - More complex to manage

---

## Final Interview Tips

### Technical Interview Strategy
1. **Clarify requirements** before coding
2. **Think out loud** - explain your approach
3. **Start with simple solution**, optimize later
4. **Test your code** with examples
5. **Ask questions** when unsure
6. **Admit what you don't know** honestly

### Common Mistakes to Avoid
- ❌ Jumping into code immediately
- ❌ Not asking clarifying questions
- ❌ Assuming requirements
- ❌ Ignoring edge cases
- ❌ Not testing code
- ❌ Being silent during problem-solving

### Behavioral Questions Framework (STAR)
- **S**ituation: Set the context
- **T**ask: Describe the challenge
- **A**ction: What you did
- **R**esult: Outcome and learning

**Example:**
> "Tell me about a time you solved a difficult technical problem"

**Good Answer:**
- **Situation**: "In my previous project, our API was timing out under load"
- **Task**: "I was tasked with improving performance to handle 10x traffic"
- **Action**: "I profiled the code, added database indexes, implemented caching, and optimized queries"
- **Result**: "Reduced response time from 2s to 200ms, handled 10x traffic successfully"

---

## Quick Reference Cheat Sheet

### Time Complexity
```
O(1)      - Constant      - Array access
O(log n)  - Logarithmic   - Binary search
O(n)      - Linear        - Loop through array
O(n log n)- Log-linear    - Merge sort
O(n²)     - Quadratic     - Nested loops
O(2ⁿ)     - Exponential   - Recursive fibonacci
```

### HTTP Status Codes
```
2xx - Success:     200 OK, 201 Created, 204 No Content
3xx - Redirect:    301 Permanent, 302 Temporary, 304 Not Modified
4xx - Client Error: 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found
5xx - Server Error: 500 Internal Error, 502 Bad Gateway, 503 Unavailable
```

### Git Commands
```
git status              - Check status
git add .               - Stage all
git commit -m "msg"     - Commit
git push origin main    - Push
git pull                - Pull changes
git checkout -b feature - Create branch
git merge feature       - Merge branch
git stash              - Save changes
git log --oneline      - View history
```

### SQL Quick Reference
```
SELECT * FROM users WHERE age > 18;
INSERT INTO users (name, email) VALUES ('John', 'j@ex.com');
UPDATE users SET email = 'new@ex.com' WHERE id = 1;
DELETE FROM users WHERE id = 1;
JOIN: INNER, LEFT, RIGHT, FULL OUTER
```

### Docker Commands
```
docker ps               - List containers
docker images           - List images
docker build -t name .  - Build image
docker run name         - Run container
docker stop id          - Stop container
docker logs id          - View logs
docker exec -it id sh   - Enter container
```

---

**Good luck with your interview! 🚀**

**Remember:** 
- Stay calm and confident
- Think before you code
- Communication is key
- It's okay to not know everything
- Show your problem-solving process

