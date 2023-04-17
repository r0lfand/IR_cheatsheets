# Modify Authentication Process: Pluggable Authentication Modules T1556.003

## Malicious PAM module playbook
In my opinion it is a good idea to keep some kind of list of standard PAM modules for your system. Some kind of integrity monitoring software will make the task of finding malicious modules easier.
In general if you suspect a malicious PAM module is taking part in the authentication process you should:
1. Check PAM configs for modifications (modification date, suspicious new contents, some module you haven't seen before)
2. Check standard module directories for modifications (modification dates of the modules)
3. Even though the modules themselves are compiled binary objects, it is still possible to look for something suspicious in their contents (run strings/YARA).

## MITRE ATT&CK
Adversaries may modify pluggable authentication modules (PAM) to access user credentials or enable otherwise unwarranted access to accounts. PAM is a modular system of configuration files, libraries, and executable files which guide authentication for many services. The most common authentication module is pam_unix.so, which retrieves, sets, and verifies account authentication information in **/etc/passwd** and **/etc/shadow**.

Adversaries may modify components of the PAM system to create backdoors. PAM components, such as **pam_unix.so**, can be patched to accept arbitrary adversary supplied values as legitimate credentials.

Malicious modifications to the PAM system may also be abused to steal credentials. Adversaries may infect PAM resources with code to harvest user credentials, since the values exchanged with PAM components may be plain-text since PAM does not store passwords.


## General
Linux-PAM is a system of libraries that handle the authentication tasks of applications (services) on the system. The library provides a stable general interface (Application Programming Interface - API) that privilege granting programs (such as **login(1)** and **su(1)**) defer to to perform standard authentication tasks. 

The principal feature of the PAM approach is that the nature of the authentication is dynamically configurable. In other words, the system administrator is free to choose how individual service-providing applications will authenticate users. This dynamic configuration is set by the contents of the single Linux-PAM configuration file **/etc/pam.conf**. Alternatively, the configuration can be set by individual configuration files located in the **/etc/pam.d/** directory. The presence of this directory will cause Linux-PAM to ignore **/etc/pam.conf**. 

Typical module paths for RHEL derivatives are:
- **/usr/lib/security/**
- **/usr/lib64/security**

Typical module path for Debian derivatives is:
- **/usr/lib/x86_64/-linux-gnu/security/**
