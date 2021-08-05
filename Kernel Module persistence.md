# Boot or Logon Autostart Execution. Kernel Modules and Extensions T1547.006
## Malicious kernel extension playbook
1. Make sure the malware can't spread. Isolate the affected hosts.
2. Dump memory and filesystem for further analysis. Check directories where kernel modules can be stored.
3. Download and use _rkhunter_ or _chkrootkit_.
4. If suspicious files were found it is recommended to reinstall the operating system.

## MITRE ATT&CK Information
Adversaries may modify the kernel to automatically execute programs on system boot. Loadable Kernel Modules (LKMs) are pieces of code that can be loaded and unloaded into the kernel upon demand. They extend the functionality of the kernel without the need to reboot the system. For example, one type of module is the device driver, which allows the kernel to access hardware connected to the system.

When used maliciously, LKMs can be a type of kernel-mode that run with the highest operating system privilege (Ring 0).  Common features of LKM based rootkits include: hiding itself, selective hiding of files, processes and network activity, as well as log tampering, providing authenticated backdoors and enabling root access to non-privileged users.

Kernel extensions, also called kext, are used for macOS to load functionality onto a system similar to LKMs for Linux. They are loaded and unloaded through `kextload` and `kextunload` commands. Since macOS Catalina 10.15, kernel extensions have been deprecated on macOS systems.

Adversaries can use LKMs and kexts to covertly persist on a system and elevate privileges. Examples have been found in the wild and there are some open source projects.

## General
This technique will only work if the adversary has administrative privileges on the system.

Distributions which use _SysV_ init scripts where _systemd_ is not available used to load modules on init listed in `/etc/modules` or `/etc/modules.conf` (from the kmod job).  
In distributions where _systemd_ is available systemd-modules-load.service reads files from:
-   `/etc/modules-load.d/*.conf`
-  `/run/modules-load.d/*.conf`
-   `/usr/lib/modules-load.d/*.conf`

to load kernel modules during boot in a static list.
A .modules executable scripts can be placed within `/etc/sysconfig/modules`.
https://security.stackexchange.com/questions/175953/how-to-load-a-malicious-lkm-at-startup

## Useful commands
`lsmod` - list all installed kernel modules
`insmod` - load a module
`rmmod` - remove a module

`modprobe -a <name>` - to load a module
`modprobe -r <name>` - to remove a module
`ls -l /etc/modprobe.d/` - list all configuration file for modules

## Detection
Loading, unloading, and manipulating modules on Linux systems can be detected by monitoring for the following commands:`modprobe`, `insmod`, `lsmod`, `rmmod`, or `modinfo`. LKMs are typically loaded into `/lib/modules` and have had the extension .ko ("kernel object") since version 2.6 of the Linux kernel.

If administrative accounts were compromised or there is a possibility that a malicious module was used tools like *rkhunter* or *chkrootkit* can help.
