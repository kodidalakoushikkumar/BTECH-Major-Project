## HARDWARE
When you first turn on your computer it tests itself to make sure everything is in working order. This is called the ``Power on self test`` . Then a program called the bootstrap loader, located in the ROM BIOS, looks for a boot sector. A boot sector is the first sector of a disk and has a small program that can load an operating system. Boot sectors are marked with a magic number `0xAA55 = 43603` at byte `0x1FE = 510`. That's the last two bytes of the sector. This is how the hardware can tell whether the sector is a boot sector or not. The bootstrap loader has a list of places to look for a boot sector. My old machine looks in the primary floppy drive, then the primary hard drive. More modern machines can also look for a boot sector on a CD−ROM. If it finds a boot sector, it loads it into memory and passes control to the program that loads the operating system. On a typical Linux system, this program will be LILO's first stage boot loader. There are many different ways of setting your system up to boot though. See the LILO User's Guide for details. See section LILO for a URL.
### Configuration 
The machine stores some information about itself in its `CMOS`. This includes what disks and RAM are in the system. The machine's BIOS contains a program to let you modify these settings. Check the messages on your screen as the machine is turned on to see how to access it. On my machine, you press the delete key before it begins loading its operating system.
## LILO
When the computer loads a boot sector on a normal Linux system, what it loads is actually a part of lilo, called the ``first stage boot loader``. This is a tiny program who's only job in life is to load and run the ``second stage boot loader``. 
The second stage loader gives you a prompt (if it was installed that way) and loads the operating system you choose.
When your system is up and running, and you run lilo, what you are actually running is the ``map installer``. This reads the configuration file /etc/lilo.conf and writes the boot loaders, and information about the operating systems it can load, to the hard disk. 
There are lots of different ways to set your system up to boot. What I have just explained is the most obvious and ``normal`` way, at least for a system who's main operating system is Linux.
### Configuration 
The configuration file for lilo is /etc/lilo.conf. There is a manual page for it: type man lilo.conf into a shell to see it. The main thing in lilo.conf is one entry for each thing that lilo is set up to boot. For a Linux entry, this includes where the kernel is, and what disk partition to mount as the root file system. For other operating systems, the main piece of information is which partition to boot from.
## The Linux Kernel 
The kernel does quite a lot really. I think a fair way of summing it up is that it makes the hardware do what the programs want, fairly and efficiently. 
The processor can only execute one instruction at a time, but Linux systems appear to be running lots of things simultaneously. The kernel achieves this by switching from task to task really quickly. It makes the best use of the processor by keeping track of which processes are ready to go, and which ones are waiting for something like a record from a hard disk file, or some keyboard input. This kernel task is called scheduling. 
If a program isn't doing anything, then it doesn't need to be in RAM. Even a program that is doing something, might have parts that aren't doing anything. The address space of each process is divided into pages. The Kernel keeps track of which pages of which processes are being used the most. The pages that aren't used so much can be moved out to the swap partition. When they are needed again, another unused page can be paged out to make way for it. This is virtual memory management. 
If you have ever compiled your own Kernel, you will have noticed that there are many many options for specific devices. The kernel contains a lot of specific code to talk to diverse kinds of hardware, and present it all in a nice uniform way to the application programs. 
The Kernel also manages the file system, inter-process communication, and a lot of networking stuff. 
Once the kernel is loaded, the first thing it does is look for an init program to run.
### Configuration
Most of the configuration of the kernel is done when you build it, using make `menuconfig`, or make `xconfig` in `/usr/src/linux/` (or wherever your Linux kernel source is). You can reset the default video mode, root file system, swap device and RAM disk size using `rdev`. These parameters and more can also be passed to the kernel from lilo. You can give lilo parameters to pass to the kernel either in lilo.conf, or at the lilo prompt. For example if you wanted to use `hda3` as your root file system instead of `hda2`, you might type `LILO: linux root=/dev/hda3`
If you are building a system from source, you can make life a lot simpler by creating a ``monolithic`` kernel. That is one with no modules. Then you don't have to copy kernel modules to the target system. 
NOTE: The System.map file is used by the kernel logger to determine the module names generating messages. The program top also uses this information. When you copy the kernel to the target system, copy System.map too.
### Key idea
The kernel is stored **on disk**, but when running it exists **only in RAM**.
So:
- `/boot` → where the kernel file is **kept**
- RAM → where the kernel actually **lives and runs**
### Why `/boot` doesn’t need a separate partition
Because `/boot` is only important **before Linux exists**.
Once boot finishes:
- kernel never reads itself again
- deleting `/boot` won’t crash running system
- `/boot` is just storage

Separate partition is only needed when firmware/bootloader cannot read your main filesystem.
## The GNU C Library 
The next thing that happens as your computer starts up is that init is loaded and run. However, init, like almost all programs, uses functions from libraries. 
You may have seen an example C program like this: 
```
main() {
printf("Hello World!\n");
}
``` 
The program contains no definition of `printf`, so where does it come from? It comes from the standard C libraries, on a GNU/Linux system, `glibc`. If you compile it under Visual C++, then it comes from a Microsoft implementation of the same standard functions. There are zillions of these standard functions, for `math`, string, dates/times memory allocation and so on. Everything in Unix (including Linux) is either written in C or has to try hard to pretend it is, so everything uses these functions. 
If you look in /lib on your Linux system you will see lots of files called `libsomething.so` or `libsomething.a ` etc. They are libraries of these functions. `Glibc` is just the GNU implementation of these functions. 
There are two ways programs can use these library functions. If you statically link a program, these library functions are copied into the executable that gets created. This is what the `libsomething.a` libraries are for. If you dynamically link a program (and this is the default), then when the program is running and needs the library code, it is called from the `libsomething.so `file. 
The command `ldd` is your friend when you want to work out which libraries are needed by a particular program. For example, here are the libraries that bash uses: 
```
[greg@Curry power2bash]$ ldd /bin/bash 
libtermcap.so.2 => /lib/libtermcap.so.2 (0x40019000) 
libc.so.6 => /lib/libc.so.6 (0x4001d000)
/lib/ld−linux.so.2 => /lib/ld−linux.so.2 (0x40000000)
```
### Configuration 
Some of the functions in the libraries depend on where you are. For example, in Australia we write dates as `dd/mm/yy`, but Americans write `mm/dd/yy`. There is a program that comes with the `glibc` distribution called `localedef` which enables you to set this up.

| Component | Role              |
| --------- | ----------------- |
| Kernel    | talks to hardware |
| glibc     | talks to kernel   |
| Program   | talks to glibc    |
### Why Alpine Linux is tiny
Because it uses musl instead of glibc.
musl = minimal implementation  
glibc = huge compatibility ecosystem
That’s why many binaries compiled on Ubuntu don’t run on Alpine.\
## Init
I will only talk about the ``System V`` style of init that Linux systems mostly use. There are alternatives. In fact, you can put any program you like in `/sbin/init`, and the kernel will run it when it has finished loading. 
It is init's job to get everything running the way it should be. It checks that the file systems are OK and mounts them. It starts up ``daemons`` to log system messages, do networking, serve web pages, listen to your mouse and so on. It also starts the Getty processes that put the login prompts on your virtual terminals. 
There is a whole complicated story about switching ``run−levels``, but I'm going to mostly skip that, and just talk about system start up.
Init reads the file `/etc/inittab`, which tells it what to do. Typically, the first thing it is told to do is to run an initialisation script. The program that executes (or interprets) this script is bash, the same program that gives you a command prompt. In Debian systems, the initialisation script is `/etc/init.d/rcS`, on Red Hat, `/etc/rc.d/rc.sysinit`. This is where the file systems get checked and mounted, the clock set, swap space enabled, host name gets set etc.
Next, another script is called to take us into the default run−level. This just means a set of subsystems to start up. There is a set of directories `/etc/rc.d/rc0.d`, `/etc/rc.d/rc1.d`, ..., `/etc/rc.d/rc6.d` in Red Hat, or `/etc/rc0.d`, `/etc/rc1.d`, ..., `/etc/rc6.d `in Debian, which correspond to the run−levels. If we are going into `runlevel` 3 on a Debian system, then the script runs all the scripts in `/etc/rc3.d` that start with `S` (for start). These scripts are really just links to scripts in another directory usually called init.d.
So our run−level script was called by init, and it is looking in a directory for scripts starting with `S`. It might find `S10syslog` first. The numbers tell the run−level script which order to run them in. So in this case `S10syslog` gets run first, since there were no scripts starting with `S00 ... S09`. But `S10syslog` is really a link to `/etc/init.d/syslog` which is a script to start and stop the system logger. Because the link starts with an `S`, the run−level script knows to execute the `syslog` script with a ``start`` parameter. There are corresponding links starting with `K` (for kill), which specify what to shut down and in what order when leaving the run−level. 
To change what subsystems start up by default, you must set up these links in the `rcN.d `directory, where N is the default run level set in your `inittab`. 
The last important thing that init does is to start some Getty's. These are ``respawned`` which means that if they stop, init just starts them again. Most distributions come with six virtual terminals. You may want less than this to save memory, or more so you can leave lots of things running and quickly flick to them as you need them. You may also want to run a Getty for a text terminal or a dial in modem. In this case you will need to edit the `inittab` file.
## The `Filesystem` 
Using the word ``filesystem`` in two different ways. There are file systems on disk partitions and other devices, and there is the file system as it is presented to you by a running Linux system. In Linux, you ``mount`` a disk file system onto the system's file system. 
In the previous section I mentioned that init scripts check and mount the file systems. The commands that do this are `fsck` and mount respectively. A hard disk is just a big space that you can write ones and zeros on. A `filesystem` imposes some structure on this, and makes it look like files within directories within directories... Each file is represented by an `inode`, which says who's file it is, when it was created and where to find its contents. Directories are also represented by `inodes`, but these say where to find the `inodes` of the files that are in the directory. If the system wants to read `/home/greg/bigboobs.jpeg`, it first finds the `inode` for the root directory / in the ``superblock``, then finds the `inode` for the directory home in the contents of /, then finds the `inode` for the directory `greg` in the contents of /home, then the `inode` for `bigboobs.jpeg` which will tell it which disk blocks to read.
If we add some data to the end of a file, it could happen that the data is written before the `inode` is updated to say that the new blocks belong to the file, or vice versa. If the power cuts out at this point, the file system will be broken. It is this kind of thing that `fsck` attempts to detect and repair. 
The mount command takes a file system on a device, and adds it to the hierarchy that you see when you use your system. Usually, the kernel mounts its root file system read−only. The mount command is used to remount it read−write after `fsck` has checked that it is OK. Linux supports other kinds of file system too: `msdos`, `vfat`, `minix` and so on. The details of the specific kind of file system are abstracted away by the virtual file system (`VFS`). 
A completely different kind of file system gets mounted on /proc. It is really a representation of things in the kernel. There is a directory there for each process running on the system, with the process number as the directory name. There are also files such as interrupts, and `meminfo` which tell you about how the hardware is being used. You can learn a lot by exploring /proc.
### Configuration
There are parameters to the command `mke2fs` which creates `ext2` file systems. These control the size of blocks, the number of `inodes` and so on. Check the `mke2fs` man page for details.
What gets mounted where on your file system is controlled by the `/etc/fstab` file. It also has a man page
## Kernel Daemons
If you issue the `ps aux` command, you will see something like the following:
```
USER PID %CPU %MEM SIZE RSS TTY STAT START TIME COMMAND
root 1   0.1  8.0  1284 536 ?   S    07:37 0:04 init [2]
root 2   0.0  0.0  0    0   ?   SW   07:37 0:00 (kflushd)
root 3   0.0  0.0  0    0   ?   SW   07:37 0:00 (kupdate)
root 4   0.0  0.0  0    0   ?   SW   07:37 0:00 (kpiod)
root 5   0.0  0.0  0    0   ?   SW   07:37 0:00 (kswapd)
root 52  0.0  10.7 1552 716 ?   S    07:38 0:01 syslogd −m 0
root 54  0.0  7.1  1276 480 ?   S    07:38 0:00 klogd
root 56  0.3  17.3 2232 11561   S    07:38 0:13 −bash
root 57  0.0  7.1  1272 480 2   S    07:38 0:01 /sbin/agetty 38400 tt
root 64  0.1  7.2  1272 484 S1  S    08:16 0:01 /sbin/agetty −L ttyS1
root 70  0.0  10.6 1472 708 1   R    Sep 11 0:01 ps aux
```

This is a list of the processes running on the system. The information comes from the /proc file system that I mentioned in the previous section. Note that init is process number one. Processes 2, 3, 4 and 5 are `kflushd`, `kupdate`, `kpiod` and `kswapd`. There is something strange here though: notice that in both the virtual storage size (SIZE) and the Real Storage Size (`RSS`) columns, these processes have zeroes. How can a process use no memory? 
These processes are the kernel daemons. Most of the kernel does not show up on process lists at all, and you can only work out what memory it is using by subtracting the memory available from the amount on your system. The kernel daemons are started after init, so they get process numbers like normal processes do. But their code and data lives in the kernel's part of the memory.
There are brackets around the entries in the command column because the /proc filesystem does not contain command line information for these processes. 
So what are these kernel daemons for? Previous versions of this document had a plea for help, as I didn't know much about the kernel daemons. The following partial story has been patched together from various replies to that plea, for which I am most grateful. Further clues, references and corrections are most welcome!
Input and output is done via buffers in memory. This allows things to run faster. What programs write can be kept in memory, in a buffer, then written to disk in larger more efficient chunks. The daemons `kflushd` and `kupdate` handle this work: `kupdate` runs periodically (5 seconds?) to check whether there are any dirty buffers. If there are, it gets `kflushd` to flush them to disk.
Processes often have nothing to do, and ones that are running often don't need all of their code and data in memory. This means we can make better use of our memory, by shifting unused parts of running programs out to the swap partition(s) of the hard disk. Moving this data in and out of memory as needed is done by `kpiod` and `kswapd`. Every second or so, `kswapd` wakes up to check out the memory situation, and if something out on the disk is needed in memory, or there is not enough free memory, `kpiod` is called in. 
There might also be a `kapmd` daemon running on your system if you have configured automatic power management into your kernel.
### Configuration 
The program update allows you to configure `kflushd` and `kswapd`. Try update −h for some information. 
Swap space is turned on by `swapon` and off by `swapoff`. The init script `(/etc/rc.sysinit or /etc/rc.d/rc.sysinit)` usually calls `swapon` as the system is coming up. I'm told that `swapoff` is handy for saving power on laptops
## System Logger 
Init starts the `syslogd` and `klogd` daemons. They write messages to logs. The kernel's messages are handled by `klogd`, while `syslogd` handles log messages from other processes. The main log is /var/log/messages. This is a good place to look if something is going wrong with your system. Often there will be a valuable clue in there.
### Configuration
The file `/etc/syslog.conf` tells the loggers what messages to put where. Messages are identified by which service they come from, and what priority level they are. This configuration file consists of lines that say messages from service x with priority y go to z, where z is a file, `tty`, printer, remote host or whatever. NOTE: `Syslog` requires the /etc/services file to be present. The services file allocates ports. I am not sure whether `syslog` needs a port allocated so that it can do remote logging, or whether even local logging is done through a port, or whether it just uses /etc/services to convert the service names you type `/etc/syslog.conf` into port numbers.
## Getty and Login 
Getty is the program that enables you to log in through a serial device such as a virtual terminal, a text terminal, or a modem. It displays the login prompt. Once you enter your username, `getty` hands this over to login which asks for a password, checks it out and gives you a shell. There are many `getty's` available. Some distributions, including Red Hat use a very small one called `mingetty` that only works with virtual terminals. The login program is part of the util−`linux` package, which also contains a `getty` called `agetty`, which works fine. This package also contains `mkswap`, `fdisk`, `passwd`, kill, `setterm`, mount, `swapon`, `rdev`, `renice`, more (the program) and more (i.e more programs).
### Configuration 
The message that comes on the top of your screen with your login prompt comes from /etc/issue. `Gettys` are usually started in `/etc/inittab`. Login checks user details in `/etc/passwd`, and if you have password shadowing, /etc/shadow.
## Bash
If you give login a valid username and password combination, it will check in `/etc/passwd` to see which shell to give you. In most cases on a Linux system this will be bash. It is bash's job to read your commands and see that they are acted on. It is simultaneously a user interface, and a programming language interpreter.
As a user interface it reads your commands, and executes them itself if they are ``internal`` commands like `cd`, or finds and executes a program if they are ``external`` commands like `cp` or `startx`. It also does groovy stuff like keeping a command history, and completing filenames. 
We have already seen bash in action as a programming language interpreter. The scripts that init runs to start the system up are usually shell scripts, and are executed by bash. Having a proper programming language, along with the usual system utilities available at the command line makes a very powerful combination, if you know what you are doing. For example (smug mode on) I needed to apply a whole stack of ``patches`` to a directory of source code the other day. I was able to do this with the following single command: 
`for f in /home/greg/sh−utils−1.16*.patch; do patch −p0 < $f; done; `
This looks at all the files in my home directory whose names start with sh−utils−1.16 and end with .patch. It then takes each of these in turn, and sets the variable f to it and executes the commands between do and done. In this case there were 11 patch files, but there could just as easily have been 3000.
### Configuration 
The file /etc/profile controls the system−wide behaviour of bash. What you put in here will affect everybody who uses bash on your system. It will do things like add directories to the PATH, set your MAIL directory variable. The default behaviour of the keyboard often leaves a lot to be desired. It is actually `readline` that handles this. `Readline` is a separate package that handles command line interfaces, providing the command history and filename completion, as well as some advanced line editing features. It is compiled into bash. By default, `readline` is configured using the file `.inputrc` in your home directory. The bash variable `INPUTRC` can be used to override this for bash. For example in Red Hat 6, `INPUTRC` is set to `/etc/inputrc` in /etc/profile. This means that backspace, delete, home and end keys work nicely for everyone. Once bash has read the system−wide configuration file, it looks for your personal configuration file. It checks in your home directory for .bash_profile, .bash_login and .profile. It runs the first one of these it finds. If you want to change the way bash behaves for you, without changing the way it works for others, do it here. For example, many applications use environment variables to control how they work. I have the variable EDITOR set to vi so that I can use vi in Midnight Commander (an excellent console based file manager) instead of its editor.
## Commands 
You do most things in bash by issuing commands like `cp`. Most of these commands are small programs, though some, like `cd` are built into the shell. 
The commands come in packages, most of them from the Free Software Foundation (or GNU).