# LiME In-Class Demo  (Ubuntu 22.04)
 
 
Every command below was run start-to-finish on this exact machine
(Ubuntu 22.04.3 LTS, kernel `6.8.0-138-generic`, VMware VM) right before class. Substitute
your own username/kernel version if demoing on a different box   everywhere below assumes
user `sandesh`; students will see `whoami`'s own username instead.
 
---
## Glossary: Terms and Abbreviations

- **LiME (Linux Memory Extractor):** A Linux kernel module that captures a computer’s physical memory for later forensic analysis.
- **Ubuntu:** A Linux-based operating system used in this demo.
- **LTS (Long-Term Support):** A release supported with updates for an extended period.
- **VM (Virtual Machine):** A software-based computer that runs an operating system inside another computer.
- **Kernel:** The core part of an operating system that manages hardware and system resources.
- **HWE (Hardware Enablement):** Ubuntu updates that provide support for newer hardware and kernels.
- **GCC (GNU Compiler Collection):** A set of tools that includes the C compiler used to build LiME.
- **`build-essential`:** An Ubuntu package that installs common tools needed to compile software.
- **Git:** A version-control tool used here to download the LiME source code.
- **`make`:** A build tool that follows instructions in a project’s makefile to compile software.
- **`.ko` (Kernel Object):** A file containing a Linux kernel module that can be loaded into the running kernel.
- **`insmod` (Insert Module):** A command that loads a kernel module into the running kernel.
- **`rmmod` (Remove Module):** A command that unloads a kernel module from the running kernel.
- **`lsmod` (List Modules):** A command that shows kernel modules currently loaded in Linux.
- **`dmesg` (Display Message):** A command that displays messages recorded by the Linux kernel.
- **Root:** The administrator account with permission to perform privileged system operations.
- **RAM (Random-Access Memory):** Temporary working memory used by the computer while it is running.
- **GB (Gigabyte):** A unit of data size equal to one billion bytes.
- **GiB (Gibibyte):** A unit of data size equal to 1,073,741,824 bytes.
- **Physical address space:** The set of memory addresses available to the computer’s physical or virtual hardware.
- **Kernel ring buffer:** A temporary area where Linux stores messages from the kernel and hardware.
- **Out-of-tree module:** A kernel module developed separately from the Linux kernel’s built-in source tree.
- **Kernel taint:** A status flag indicating that a condition, such as loading an unsigned external module, may affect kernel support or debugging.
- **Module signature:** A digital signature used to verify who created a kernel module and whether it has been altered.
- **Secure Boot:** A startup security feature that can block unsigned boot components or kernel modules.
- **MOK (Machine Owner Key):** A key that can be enrolled to allow Secure Boot to trust certain third-party software.
- **EFI (Extensible Firmware Interface):** Firmware used to start a computer and hand control to its operating system.
- **Dual-boot:** A setup that lets a computer start one of two or more installed operating systems.
- **Volatility 3:** A memory-forensics framework used to analyze captured memory images.
- **Debug symbols:** Extra build information that helps forensic tools connect kernel data to names and structures.
- **EDR (Endpoint Detection and Response):** Security software that monitors computers and helps detect and investigate threats.
- **`format=lime`:** An option that tells LiME to save the memory capture in its segmented LiME format.
- **`format=raw`:** An option that tells LiME to save the capture as a raw sequence of bytes.
- **`format=padded`:** An option that tells LiME to save the capture with padding between memory regions.

 ---
## 0. Check your kernel version first
 
```bash
uname -r
```
 
Whatever this prints, the `.ko` file you build later will be named
`lime-<that-version>.ko`. Point this out to students so the filename in step 3 doesn't
confuse them   it's not a typo, it's *their* kernel version.
 
## 1. Install build tools
 
```bash
sudo apt update
sudo apt install -y git build-essential
```
 
This pulls in `git`, `gcc`, `make`, and friends. Took ~10 seconds on the classroom VM.
 
## 2. Match the compiler to the kernel's compiler (the one gotcha)
 
Ubuntu's HWE kernel (`6.8.0-138-generic`) was built with **gcc-12**, but
`build-essential` on 22.04 installs **gcc-11** by default. If you skip this step, `make`
fails with:
 
```
/bin/sh: 1: gcc-12: not found
make[3]: *** [scripts/Makefile.build:243: .../tcp.o] Error 127
```
 
Fix   install the matching version:
 
```bash
sudo apt install -y gcc-12
```
 
**Tell students:** run `uname -r` and check the running kernel's build info if unsure
which gcc it wants; on 22.04 HWE kernels it's almost always `gcc-12`. If `make` complains
about a different missing `gcc-N`, install that N instead.
 
## 3. Clone and build LiME
 
```bash
cd ~
git clone https://github.com/504ensicsLabs/LiME.git
cd LiME/src
make
```
 
Expect a `warning: the compiler differs...` line   that's fine as long as it's immediately
followed by `You are using: gcc-12 ...` matching the kernel's compiler. A successful build
ends with:
 
```
strip --strip-unneeded lime.ko
mv lime.ko lime-6.8.0-138-generic.ko
```
 
Confirm the module exists:
 
```bash
ls -la lime-*.ko
```
 
## 4. Load the module and capture a memory dump
 
```bash
sudo insmod /home/sandesh/LiME/src/lime-6.8.0-138-generic.ko path=/home/sandesh/memdump.lime format=lime
```
 
This blocks for a few seconds while it dumps physical memory, then returns. Verify:
 
```bash
lsmod | grep lime
ls -lh /home/sandesh/memdump.lime
```
 
On the classroom VM (3.8 GiB RAM per `free -h`) this produced a **4.0 GB** dump file
owned by `root` with `r--r--r--` permissions   that's expected; the dump size reflects the
physical address space, not just "used" memory, and it will differ on other machines/VMs.
 
**Sanity check with dmesg** (needs `sudo` to read the kernel ring buffer as a
non-root user):
 
```bash
sudo dmesg | tail -5
```
 
You should see two lines like:
```
lime: loading out-of-tree module taints kernel.
lime: module verification failed: signature and/or required key missing - tainting kernel
```
 
**Tell students this is expected, not an error.** Any unsigned, out-of-tree kernel module
taints the kernel and triggers this warning   it's exactly what you want to see, it
confirms LiME loaded.
 
## 5. Unload the module when done
 
```bash
sudo rmmod lime
lsmod | grep lime
```
 
The second command should print nothing   module is gone. The dump file
(`memdump.lime`) stays on disk; unloading the module just removes it from the running
kernel, it doesn't touch the file you already captured.
 
---
 
## Talking points while demoing
 
- `format=lime` (LiME's own segmented format) vs `format=raw`/`format=padded`   mention
  these exist as options but you're using `lime` since that's what the Volatility3
  workflow in this repo expects.
- Why root/insmod at all: LiME is a loadable kernel module, so capturing memory this way
  requires kernel module privileges   this is the same reason antivirus/EDR kernel
  drivers need elevated install rights.
- The dump is only useful for analysis (Volatility3, etc.) if you also generate matching
  debug symbols for the exact running kernel   that's covered in the other doc in this
  repo (`How to Create Linux Profile(Volatility 3).md`).
## If something goes wrong mid-demo
 
- `make` error about a missing `gcc-N` → install that exact gcc version (step 2).
- `insmod: ERROR: could not insert module ...: Operation not permitted` → check
  Secure Boot is disabled (`mokutil --sb-state`); this VM reported
  "EFI variables are not supported on this system" so it was never an issue here, but a
  student's laptop dual-booting with Secure Boot enabled will block unsigned modules.
- `insmod: ERROR: could not insert module ...: File exists` → the module is already
  loaded from a previous run; `sudo rmmod lime` first, then retry.
- Dump file already exists from a previous take → `insmod` will fail or refuse to
  overwrite; `rm` the old `memdump.lime` (or dump to a new path) before re-running step 4.
