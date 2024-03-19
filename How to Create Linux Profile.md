# Linux profile creation steps.....


![image](https://github.com/Sandesh028/Universe/assets/123413262/da9fff0e-a64a-4b2e-b361-adc28cf8f489)


I had downloaded and installed Ubuntu 22.04.36LTS which is pretty much stable.
## Next step is :
### Install Go 
sudo apt update
sudo apt install golang-go

To get the debug symbols for this OS:
### Clone the dwarf2json repository and build it
```sh
git clone https://github.com/volatilityfoundation/dwarf2json.git
```
```sh
cd dwarf2json/
```
```sh
go build
```
```sh
git clone https://github.com/volatilityfoundation/dwarf2json.git
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/41043bf4-1182-40ac-a8bd-eac06c7fbab7)




Next step: 
### Add the debug symbol repository
```sh
echo "deb http://ddebs.ubuntu.com $(lsb_release -cs) main restricted universe multiverse
```
```sh
deb http://ddebs.ubuntu.com $(lsb_release -cs)-updates main restricted universe multiverse
```
```sh
deb http://ddebs.ubuntu.com $(lsb_release -cs)-proposed main restricted universe multiverse" | \
```
```sh
sudo tee -a /etc/apt/sources.list.d/ddebs.list
```
### Install the debug symbol keyring
```sh
$ sudo apt install ubuntu-dbgsym-keyring
```

### Update the package list
```sh
$ sudo apt update
```

### Install the debug symbols for your currently running kernel
```sh
$ sudo apt install linux-image-$(uname -r)-dbgsym
```

Saving the debug symbol to json file format: 
```sh
$ sudo ./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-$(uname -r) > linux-image-$(uname -r)-amd64.json
```

As extra I am also creating system map json file:
```sh
$ sudo ./dwarf2json linux --elf /usr/lib/debug/boot/vmlinux-$(uname -r) --system-map /boot/System.map-$(uname -r) > linux-image-$(uname -r)-amd64-SystemMap.json
 ```

![image](https://github.com/Sandesh028/Universe/assets/123413262/cb1f68ab-0cfc-45a3-bd9c-5e5ee6300624)




### Next Step:
```sh
$ sudo apt update
$ sudo apt install git build-essential kernel-headers-$(uname -r) dkms
```
### Download and Install Lime tool to take the dump:
```sh
$ git clone https://github.com/504ensicsLabs/LiME.git
```
```sh
$ cd LiME/src
```
```sh
$ make
```
### This command is used to create the memory dump file with the lime extension: 
```sh
$ sudo insmod /home/kali/LiME/src/lime-6.2.0-36-generic.ko path=/home/kali/memdump.lime format=lime
```

### Copying the symbol debug tables to the volatility3 symbols directory:
```sh
$ cp /home/kali/dwarf2json/linux-image-6.2.0-36-generic-amd64.json /home/kali/volatility3/volatility3/symbols/
linux-image-6.2.0-36-generic-amd64.json
```
```sh
$ cp /home/kali/dwarf2json/linux-image-6.2.0-36-generic-amd64-SystemMap.json /home/kali/volatility3/volatility3/symbols/
linux-image-6.2.0-36-generic-amd64-SystemMap.json
 ```

![image](https://github.com/Sandesh028/Universe/assets/123413262/158dd3fb-fb39-476c-ad8a-fdfbfc295837)


After this our linux profile is created for this ubuntu 22.04 version.









### Once Memory dump is saved as memorydump.lime in the Ubuntu system, I tested it with the volatility3 with the following commands:
* PsList: Lists active processes in the memory image.
* PsScan: Scans for processes in the memory image by walking the process list.
* PsTree: Displays active processes in a parent-child relationship tree structure.
* Banners: Identifies and prints the operating system banner information from the memory image.
* Capabilities: Lists the Linux capabilities for each process.
* Check Modules: Compares the loaded modules list against the module list obtained from sysfs.
* Check Syscall: Checks the system call table for unexpected modifications (hooks).
* Elfs: Lists ELF executables and shared libraries mapped into process address spaces.
* Envvars: Lists environment variables for each process.
* IOMem: Provides information similar to what is available in /proc/iomem on a live Linux system.
* Keyboard_Notifiers: Analyzes keyboard notifier call chains for hooks.
* KMSG: Reads the kernel log buffer messages.
* Lsmod: Lists currently loaded kernel modules.
* Lsof: Lists open file descriptors across all processes.
* Malfind: Searches for memory regions within processes that may contain injected code.
* Mountinfo: Lists mount points and mount namespaces for processes.
* Proc.Maps: Lists all memory-mapped files for each process.
* PsAux: Lists processes along with their command-line arguments.
* Sockstat: Lists network connections and sockets for each process.
* tty_check: Checks tty devices for hooks or manipulations.
* FrameworkInfo: Provides details about the Volatility framework's components and configuration.
* IsfInfo: Displays information about the available Intermediate Symbol Format (ISF) files.
* LayerWriter: Writes out the data from a specified memory layer (used for debugging and analysis).
* Check_afinfo: Verifies the operation function pointers for network protocols to check for rootkits.
* Check_creds: Looks for processes that are sharing credential structures, which could indicate credential reuse or theft.
* Check_idt: Checks the Interrupt Descriptor Table (IDT) for unexpected modifications, which could indicate rootkit activity.





### 1] PsList
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.pslist
```
![image](https://github.com/Sandesh028/Universe/assets/123413262/f83be3ff-929e-416b-84ab-7582ac36cf8d)
 

### 2] PsScan
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.psscan
```
 ![image](https://github.com/Sandesh028/Universe/assets/123413262/791b906e-2fdc-4aed-85fc-887e5e7d0975)






3] PsTree [Didn’t worked ]
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.pstree
```
 ![image](https://github.com/Sandesh028/Universe/assets/123413262/27d26cd3-29d2-4f9c-9efb-a0f4f878bff7)

4] Banners
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime banners.Banners
```
 ![image](https://github.com/Sandesh028/Universe/assets/123413262/25fccdaa-6083-4201-8856-27d266301a85)

5] Capabilities [Didn’t worked]
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.capabilities.Capabilities
```
 ![image](https://github.com/Sandesh028/Universe/assets/123413262/197268e0-ebaa-4045-8892-f1bdb0a52d9d)

6] Check Modules
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.check_modules.Check_modules
 ```

![image](https://github.com/Sandesh028/Universe/assets/123413262/1732a608-db37-4de6-b1cf-f93cfbe53753)



7] Check syscall
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.check_syscall.Check_syscall
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/0ec88d8d-7e4d-4069-a0f2-56fe3cd7ed4b)

8] Elfs
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.elfs.Elfs
``` 


![image](https://github.com/Sandesh028/Universe/assets/123413262/4ad44064-2bba-4b80-8429-9cbbea654165)




9] Envvars
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.envvars.Envvars
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/3e177535-aab2-4d60-a437-697f3cf9cc77)

10] IOMem
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.iomem.IOMem
```
 ![image](https://github.com/Sandesh028/Universe/assets/123413262/5dbec2c3-2f45-47d7-8133-06d415598bbb)


11] Keyboard_Notifiers
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.keyboard_notifiers.Keyboard_notifiers
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/871adbb5-ac34-463b-be67-42c9da1089cf)






12] KMSG
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.kmsg.Kmsg
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/9c090b84-f654-4eff-ac40-8fa7462a9d18)


13] Lsmod
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.lsmod.Lsmod
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/7191867d-0211-4548-93ad-fa0b297044fb)






14] Lsof
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.lsof.Lsof 
```
![image](https://github.com/Sandesh028/Universe/assets/123413262/1686c62d-6ed3-4da5-9277-2def150e02f0)

15] Malfind
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.malfind.Malfind
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/333f37e5-8b4b-4951-94e2-502cf861c223)








16] Mountinfo
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.mountinfo.MountInfo
```
![image](https://github.com/Sandesh028/Universe/assets/123413262/39bc6414-d832-4847-9eff-6f1819c0e1c0)
 

17] Proc.Maps
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.proc.Maps
``` 


![image](https://github.com/Sandesh028/Universe/assets/123413262/bb709059-1319-49a8-9a4f-da6fa43f577e)










18] Psaux
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.psaux.PsAux
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/5a8e59d2-abf5-4950-ac8b-3fc16c887314)


19] Sockstat
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.sockstat.Sockstat
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/0c0706b8-25d1-4e93-ade6-3e9de763da4a)


20] tty_check
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.tty_check.tty_check
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/b3e3611c-6e6d-4ed6-af1a-0c38de8bd22a)


21] frameworkinfo.FrameworkInfo
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime frameworkinfo.FrameworkInfo
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/de090f73-bd06-4d68-8daa-158a9b92690c)


22] isfinfo.IsfInfo
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime isfinfo.IsfInfo
```

 ![image](https://github.com/Sandesh028/Universe/assets/123413262/b18d1626-4518-4fd2-931e-4d38d854afe9)


23] layerwriter.LayerWriter
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime layerwriter.LayerWriter
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/76a32ce6-c2cd-4820-b150-f9fdf76eb1e7)


24] Check_afinfo [Didn’t worked]
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.check_afinfo.Check_afinfo
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/d23a7e59-bcc9-4a82-ab08-8a57a39fd055)


25] Check_creds
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.check_creds.Check_creds
```

![image](https://github.com/Sandesh028/Universe/assets/123413262/6ad7992c-8c29-4b58-97d7-022f4f3b23e8)


26] Check_idt
```sh
$ sudo python3 vol.py -f /home/kali/memdump.lime linux.check_idt.Check_idt
``` 

![image](https://github.com/Sandesh028/Universe/assets/123413262/f797d31a-1b8c-46ea-b002-65fda67f98fe)


## References:
- https://beguier.eu/nicolas/articles/security-tips-3-volatility-linux-profiles.html
- https://medium.com/@alirezataghikhani1998/build-a-custom-linux-profile-for-volatility3-640afdaf161b
- https://volatility3.readthedocs.io/en/latest/getting-started-linux-tutorial.html
- https://chat.openai.com/



