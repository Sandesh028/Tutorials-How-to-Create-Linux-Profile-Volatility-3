# Setting up LiME and Volatility3 for Memory Analysis on Linux

This guide outlines the steps to set up and run `dwarf2json`, `LiME`, and `Volatility3` for Linux memory dump analysis.

---

## Steps to Install and Configure Tools

### 1. Install Git
```bash
sudo apt install git
```
Git is required to clone repositories from GitHub.

### 2. Clone the `dwarf2json` Repository
```bash
sudo git clone https://github.com/volatilityfoundation/dwarf2json.git
```

### 3. Navigate to the `dwarf2json` Directory
```bash
cd dwarf2json/
```

### 4. Install Go Language
```bash
sudo apt install golang-go
```

### 5. Build the `dwarf2json` Tool
```bash
sudo go build -buildvcs=false
```

### 6. Add Debug Symbol Repositories
```bash
sudo echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse
deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse
deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" | \
sudo tee -a /etc/apt/sources.list.d/ddebs.list
```

### 7. Install the Debug Symbols Keyring
```bash
sudo apt install ubuntu-dbgsym-keyring
```

### 8. Update and Upgrade Packages
```bash
sudo apt update
sudo apt upgrade
```

### 9. Install the Linux Image Debug Symbols
```bash
sudo apt install linux-image-$(uname -r)-dbgsym
```

---

## Generate JSON Symbol Files with `dwarf2json`

### 10. Create JSON from Kernel ELF
```bash
sudo ./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-$(uname -r) > linux-image-$(uname -r)-amd64.json
```

### 11. Generate JSON with System Map
```bash
sudo ./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-$(uname -r) --system-map /boot/System.map-$(uname -r) > linux-image-$(uname -r)-amd64-SystemMap.json
```

---

## Set Up LiME for Memory Dumping

### 12. Clone the LiME Repository
```bash
git clone https://github.com/504ensicsLabs/LiME.git
```

### 13. Navigate to the LiME Source Directory
```bash
cd LiME/src
```

### 14. Install Make and GCC
```bash
apt install make
apt install gcc-12
```

### 15. Build the LiME Kernel Module
```bash
make
```

### 16. Insert the LiME Module and Create a Memory Dump
```bash
insmod lime-6.8.0-47-generic.ko path=/home/sam/memdump.lime format=lime
```

### 17. Verify the Memory Dump
```bash
ls -l /home/sam/memdump.lime
```

---

## Set Up Volatility3 for Memory Analysis

### 18. Clone the Volatility3 Repository
```bash
git clone https://github.com/volatilityfoundation/volatility3.git
```

### 19. Create Symbols Directory for Volatility3
```bash
mkdir -p /home/sam/volatility3/volatility3/symbols/linux/
```

### 20. Copy the JSON Symbol Files
```bash
cp /home/sam/Desktop/dwarf2json/linux-image-6.8.0-47-generic-amd64.json /home/sam/volatility3/volatility3/symbols/linux/
cp /home/sam/Desktop/dwarf2json/linux-image-6.8.0-47-generic-amd64-SystemMap.json /home/sam/volatility3/volatility3/symbols/linux/
```

---

## Run Volatility3 Plugins for Memory Analysis

### 21. List Processes Using `linux.pslist`
```bash
sudo python3 vol.py -f /home/sam/memdump.lime linux.pslist.PsList
```

### 22. Scan for Processes Using `linux.psscan`
```bash
sudo python3 vol.py -f /home/sam/memdump.lime linux.psscan
```

### 23. Extract Banners Using `banners.Banners`
```bash
sudo python3 vol.py -f /home/sam/memdump.lime banners.Banners
```

---

