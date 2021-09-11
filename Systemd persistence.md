# Create or Modify System Process: Systemd Service T1543.002
## Malicious systemd unit playbook
1.  ### Locate the malicious unit:
Use these commands:
`find / -path "*/systemd/system/*.service" -exec grep -H -E "ExecStart|ExecStop|ExecReload" {} \; 2>/dev/null`
`find / -path "*/systemd/user/*.service" -exec grep -H -E "ExecStart|ExecStop|ExecReload" {} \; 2>/dev/null`
These commands will enumerate all the commands specified by system and user systemd services on a host.Pay particular attention to service files corresponding to services you don’t recognize and execution commands that contain `bash`, `python`, `sh` or anything else that can be used to execute commands.
Manual service analysis:
`systemctl` -> [list of services]  
`systemctl --type=service --state=active` -> [list of active services] - if there is something suspicious found use `systemctl status [name_of_service]`. It should drop additional information on the use of the service.

2. ### Stop the malicious unit
`sudo systemctl stop zlyden`

3. ### Analyze the unit for useful artifacts
Use `cat` or open it with an editor of your choice. 
`cat unit_file_to_open` - this will spit out contents of the file to your terminal.
In some cases it might be useful to save the file until further analysis. 
You can use SCP to download the file:
`scp user@server:/path/to/remotefile.zip /Local/Target/Destination`
Or just use scp client with GUI.

4. ### Remove the malicious unit
`rm -f <unit_file>` - remove the malicious unit file

## MITRE ATT&CK Information
Adversaries may create or modify systemd services to repeatedly execute malicious payloads as part of persistence. The systemd service manager is commonly used for managing background daemon processes (also known as services) and other system resources. Systemd is the default initialization (init) system on many Linux distributions starting with Debian 8, Ubuntu 15.04, CentOS 7, RHEL 7, Fedora 15, and replaces legacy init systems including SysVinit and Upstart while remaining backwards compatible with the aforementioned init systems. Systemd utilizes configuration files known as service units to control how services boot and under what conditions.

## General
**Systemd** is a system and service manager for Linux operating systems. It is designed to be backwards compatible with SysV init scripts, and provides a number of features such as parallel startup of system services at boot time, on-demand activation of daemons, or dependency-based service control logic. In Red Hat Enterprise Linux 7, systemd replaces Upstart as the default init system.

Systemd introduces the concept of **systemd units**. These units are represented by unit configuration files located in one of the directories, and encapsulate information about system services, listening sockets, and other objects that are relevant to the init system.

Systemd daemon usually has a PID = 1.

**Systemd system-level unit directories (executed as root or a specified user):**
`/usr/lib/systemd/system` - contains unit files that came with packages (RPM/DEB) that were installed on the system (`/lib/systemd/system` for Debian based systems)
`/run/systemd/system` - runtime location for unit files
`/etc/systemd/system` - unit files that were added by administrator or a privileged user

**Systemd user-level unit directories:**
`/etc/systemd/user`  
`~/.config/systemd/user`  
`~/.local/share/systemd/user`  
`/run/systemd/user`  
`/usr/lib/systemd/user`
There also might be a symlink inside these folders, allowing the unit file to be anywhere on the system.

The default configuration of systemd is defined during the compilation and it can be found in systemd configuration file at `/etc/systemd/system.conf`.

Minimal structure of a unit file:
```text
[Unit]
Description=Foo

[Service]
ExecStart=/usr/sbin/foo-daemon

[Install]
WantedBy=multi-user.target #similar to runlevel=3 - no GUI normal startup
```
Other example:
```text
[Unit]  
Description=Atomic Red Team Service

[Service]  
Type=simple  
ExecStart=/bin/touch /tmp/art-systemd-execstart-marker

[Install]  
WantedBy=default.target
```

## Hunting for malicious systemd services

As mentioned before, systemd service files can live in multiple places on disk. When hunting with EDR technology, you can check out suspicious file modifications of those folders. By default, the processes that typically write these files are Python because of package management via Yum on CentOS/RHEL, dpkg because of package management via apt on Debian derivatives, and systemctl itself to enable and disable services. Bash shell scripts and processes other than those mentioned touch these files less often, so they’ll be more suspect while hunting.

If you don’t have EDR tools and you want to hunt for malicious persistence via the Linux command line, you can do so with these commands:

`find / -path "*/systemd/system/*.service" -exec grep -H -E "ExecStart|ExecStop|ExecReload" {} \; 2>/dev/null`

`find / -path "*/systemd/user/*.service" -exec grep -H -E "ExecStart|ExecStop|ExecReload" {} \; 2>/dev/null`

These commands will enumerate all the commands specified by system and user systemd services on a host. If you’re daring, you might combine these commands with `ssh` commands to execute them on remote hosts, saving the output to files for hunting and processing later. Pay particular attention to service files corresponding to services you don’t recognize and execution commands that contain `bash`, `python`, or `sh`.

When hunting on EDR platforms, be aware of process ancestry involving systemd. On modern Linux systems, the systemd process should start all services, operating as the root of all process trees. Most of the time, systemd will spawn daemon processes, such as httpd or mysqld. When it spawns shell processes such as bash, this definitively shows that bash was specified as a service execution. Legitimate user instances of bash shells that occur after logon do not spawn as direct children of systemd, and they exist further down the ancestry chain. While a systemd -> bash ancestry is not certainly evil, it’s a good place to start when hunting in your own environment.


## journalctl commands to view logs
`journalctl` displays logs from oldest to newest. To view newest logs first the command should be `journalctl -r`.

Filter logs by date-time:
`journalctl --since "2018-08-30 14:10:10" --until "2018-09-02 12:05:50"`
`--since` and `--until` options can be used separately.

Show logs for the last boot:`journalctl -b`

List all previous boots: `journalctl --list-boots`

Check logs from previous boot: `journalctl -b a09dce7b2c1c458d861d7d0f0a7c8c65`

Show logs for a specific systemd service:
`journalctl -u ssh`

Different log formats can be selected:
`journalctl -o {short(default)|verbose|json|json-pretty|cat}`
- `short` - The default option, displays logs in the traditional syslog format.
- `verbose` - Displays all information in the log record structure.
- `json` - Displays logs in JSON format, with one log per line.
- `json-pretty` - Displays logs in JSON format across multiple lines for better readability.
- `cat` - Displays only the message from each log without any other metadata.

systemd-journald can be configured to persist your systemd logs on disk, and it also provides controls to manage the total size of your archived logs. These settings are defined in `/etc/systemd/journald.conf`. 
Do not forget to restart the daemon! `sudo systemctl restart systemd-journald`
