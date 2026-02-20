## What _userspace_ actually is

**Userspace = everything that runs _outside the kernel_** and talks to the kernel using **system calls**.
So yes — commands, binaries and libraries are part of it… but `userspace` is _much broader_.

### It includes:

1. **Shells (command interpreters)**
    - `bash`, `zsh`, `sh`
    - They let humans interact with the system

2. **User programs / applications**
    - Commands: `ls`, `cp`, `grep`
    - Editors: `vim`, `nano`
    - Browsers: Firefox, Chrome
    - Servers: nginx, apache, mysql
    - GUI apps: file manager, terminal, etc.

3. **Binaries (executables)**
    - Files in `/bin`, `/usr/bin`, `/usr/sbin`
    - Any compiled program you run

4. **Libraries**
    - Shared libs like `libc.so`
    - Provide functions so programs don’t talk to kernel directly

5. **System services / daemons**
    - `systemd`
    - `sshd`
    - `cron`
    - Network managers

6. **Graphical environment**
    - X11 / Wayland
    - Desktop environments (GNOME, KDE)
## Key idea

Userspace **cannot directly access hardware**.
Instead it requests the kernel:

```
userspace program → system call → kernel → hardware
```

Example:

```
cat file.txt  
   ↓  
read() system call  
   ↓  
kernel reads disk  
   ↓  
returns data to program
```

## Simple definition

> Userspace is all software running on top of the kernel that interacts with hardware only through system calls.

## Quick contrast

|Kernel space|User space|
|---|---|
|Drivers|Applications|
|Memory management|Commands|
|Process scheduling|Shell|
|Hardware control|Libraries|

So your statement:   "userspace is nothing but commands, binaries, libraries"

✔ Partially correct  
❌ Missing: services, GUI, daemons, interpreters, servers, applications — basically **everything you interact with in Linux**

## What BusyBox actually is

BusyBox is a **single executable program** that bundles _hundreds of Linux utilities_ into one binary.

Think of it like:  a miniature Linux userspace packed into one file

It is heavily used in:
- embedded systems (routers, TVs, IoT)
- initramfs
- recovery environments
- containers

## Does it include a shell?

Yes. BusyBox provides a shell called:   **`ash` (Almquist shell)**

When you run:

```
/bin/sh
```

On many minimal systems → you are actually running **BusyBox ash**
## What BusyBox provides

Inside one binary:

|Category|Examples|
|---|---|
|Shell|`sh` (ash)|
|Commands|`ls`, `cp`, `mv`, `cat`, `grep`|
|System tools|`mount`, `ifconfig`, `ps`|
|Network|`ping`, `wget`, `telnet`|
|Init|init system (sometimes)|

All symlink to the same binary:

```
ls -l /bin/ls  
→ /bin/ls -> busybox
```

BusyBox checks `argv[0]` to decide what command to behave as.
## Important concept

**BusyBox = multi-call binary**

```
1 program → acts like 300+ commands
```
## So final answer

> BusyBox is not a shell.  
> It is a collection of utilities that _includes_ a shell (ash).

> **YES — in that custom Linux you build, you really are using BusyBox `ash` as your actual shell.**

And not just briefly — it is your _entire user environment_.
## What that tutorial actually creates

The article builds a Linux system made of only:

- Linux kernel
- initramfs
- BusyBox
- your handwritten `init`

No GNU tools, no bash, no systemd, no coreutils.

The author even says the goal is a system that _boots straight into a BusyBox shell_.
## The boot chain in your mini-Linux

From the article:

```
Kernel → Initramfs → init script → BusyBox shell
```

So after boot:  **PID 1 (init)** runs your script  
then your script runs:

```
exec setsid cttyhack /bin/sh
```

And `/bin/sh` is a symlink:

```
sh -> busybox
```

Meaning:

```
/bin/sh = BusyBox ash
```

Because BusyBox provides the shell implementation.
## So what shell are you actually typing into?

|System|Shell you interact with|
|---|---|
|Ubuntu/Fedora|bash/zsh|
|Alpine Linux|ash (BusyBox)|
|Your tiny Linux|**ash (BusyBox)**|

In your build:   the operating system literally **is kernel + BusyBox + your script**

There is no other shell available.

## Why this matters (the big realization)

You’ve basically built:  the _minimum possible Unix_:

```
kernel + shell = operating system
```

Everything else in normal distros is just extra layers.

> `ash` _is itself a userspace program_ that lets _you_ interact with the kernel.
## Correct chain

```
You (human)  
   ↓  
Shell (ash)  
   ↓  
Programs / system calls  
   ↓  
Kernel  
   ↓  
Hardware
```

So:   ash = command interpreter (UI between human and OS)
## What the shell actually does

The `ash` shell from BusyBox:
1. Reads your command
2. Parses it
3. Decides what to run
4. Starts a program OR makes system calls

Example:

```
cat file.txt
```

### What really happens

```
You type command  
→ ash parses "cat"  
→ ash launches /bin/cat  
→ cat calls read() syscall  
→ kernel reads disk  
→ hardware returns data  
→ kernel → cat → ash → you
```

## Key understanding

```
The shell does NOT talk to hardware  
The shell does NOT control the kernel
```

It only:   **requests the kernel to run programs**
## So refine your sentence

Your sentence:

> ash is a way to interact with userspace in turn with kernel in turn with hardware

Better:

> ash is a userspace program that lets the user request services from the kernel, which then controls hardware.

## One-line mental model

**Kernel = power**  
**Userspace programs = workers**  
**Shell = receptionist between you and workers**

You never talk to workers directly — only through the receptionist.> **In userspace, the shell is just one program, and we use it to run other userspace utilities.**
## Structure of userspace

```
Userspace  
 ├─ Shell (ash/bash)  
 ├─ Utilities (ls, cat, mount, cp…)  
 ├─ Libraries (libc)  
 └─ Applications (editors, servers, etc.)
```

The shell itself does almost nothing alone — it mainly **starts other programs**.
## Example in your tiny Linux

Your shell is `ash` from BusyBox
You type:  ls /dev
What happens:

```
ash → runs "ls" utility  
ls  → asks kernel for directory entries  
kernel → reads filesystem → disk
```

So:

> shell = launcher  
> utilities = workers

## Important insight

The shell is **not special to the kernel**.

To Linux, `ash` is just another normal program like `cat` or `grep`.

The kernel doesn’t know:
- commands
- pipes
- redirection
- scripting


Those are all features implemented by the shell.
## Final refined statement

> Userspace contains many programs.  
> The shell is a userspace program that we use to run other userspace utilities, which then request services from the kernel.


## In normal distros (Ubuntu, Fedora, Arch…)

We do **NOT** rely on BusyBox for userspace.
Instead we use the **GNU userspace** — mainly the project:   GNU Project
So a typical system is actually:

```
Linux kernel  +  GNU userspace
```

That’s why people often say **GNU/Linux**.

## Main components of normal userspace

### 1️⃣ Shell

Most common:

|Shell|Where|
|---|---|
|bash|default in many distros|
|zsh|popular modern setups|
|fish|user-friendly shell|

Example:  GNU Bash is default in many installations
### 2️⃣ Core utilities (NOT BusyBox ones)

Instead of BusyBox tools, you use:  GNU Core Utilities
These provide:

```
ls, cp, mv, rm, cat, chmod, chown, mkdir...
```

Full-featured, bigger, more standards-compliant than BusyBox versions.

### 3️⃣ System libraries

Programs don’t talk to kernel directly — they use:  glibc
This is the most important userspace component after the kernel.

### 4️⃣ Init system

Modern distros boot with:   systemd
(not BusyBox init)
## Comparison

|Feature|Tiny Linux|Ubuntu/Fedora/Arch|
|---|---|---|
|Shell|ash|bash/zsh|
|Utilities|BusyBox|GNU coreutils|
|libc|musl/uclibc|glibc|
|Init|busybox init|systemd|
|Size|few MB|several GB|
|Purpose|embedded/minimal|general-purpose OS|
## Big picture

There are basically **two ecosystems of userspace**:

```
Embedded Linux → BusyBox world  
Desktop/Server Linux → GNU world
```

## Final answer

> In Ubuntu and most distributions, userspace is mainly GNU tools (bash + coreutils + glibc + systemd), not BusyBox.