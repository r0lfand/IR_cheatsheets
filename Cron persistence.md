# Scheduled Task/Job: Cron T1053.003
## Malicious Cron entry playbook
1. ### Locate the malicious entry.
Paths to check are:
`/etc/anacrontab`, `/var/spool/anacron` and `/home/$USER/.anacron/` - if anacron is installed 
`/var/spool/cron` - all users' jobs excluding root (**RHEL-derived**)
`/etc/crontab` and `/etc/cron.d` - system-wide cronjobs. Requires privileged access to edit.
In most Linux distros it is also possible to put files inside `/etc/cron.{hourly|daily|weekly|monthly}` directories.
On **debian-based** distros user crontabs are stored in `/var/spool/cron/crontabs`.

2. ### Remove the job.
You can use any text editor you like to edit the job files. (nano/vi)

## Useful commands
`crontab -l` - list current user's cron job file
`crontab -r` - remove current user's job file
`crontab -e` - edit current user's job file

## MITRE ATT&CK Information
Adversaries may abuse the `cron` utility to perform task scheduling for initial or recurring execution of malicious code. The `cron` utility is a time-based job scheduler for Unix-like operating systems. The `crontab` file contains the schedule of cron entries to be run and the specified times for execution. Any `crontab` files are stored in operating system-specific file paths.

An adversary may use `cron` in Linux or Unix environments to execute programs at system startup or on a scheduled basis for persistence. `cron` can also be abused to conduct remote Execution as part of Lateral Movement and or to run a process under the context of a specified account.

## General
Although you can edit crontab files manually, it is recommended to use the `crontab` command.
In most Linux distros it is also possible to put files inside `/etc/cron.{hourly|daily|weekly|monthly}` directories.
On debian-based distros user crontabs are stored in `/var/spool/cron/crontabs`.

```txt
* * * * * command(s)
- - - - -
| | | | |
| | | | ----- Day of week (0 - 7) (Sunday=0 or 7)
| | | ------- Month (1 - 12)
| | --------- Day of month (1 - 31)
| ----------- Hour (0 - 23)
------------- Minute (0 - 59)
```

### RHEL
`/var/spool/cron` - user crontabs (contains all of the configured crontabs of all users, except the root).
`/etc/crontab` and `/etc/cron.d` contain system-wide crontabs. Editing requires privileges.

### System-wide Crontab files
```txt
* * * * * <username> command(s)
```
The syntax of system-wide crontab file is slightly different than the user crontab. It contains an additional mandatory user field that specifies which user will run the cron job.

### Predefined Macros

There are several special Cron schedule macros used to specify common intervals. You can use these shortcuts in place of the five-column date specification.

-   `@yearly` (or `@annually`) - Run the specified task once a year at midnight (12:00 am) of the 1st of January. Equivalent to `0 0 1 1 *`.
-   `@monthly` - Run the specified task once a month at midnight on the first day of the month. Equivalent to `0 0 1 * *`.
-   `@weekly` - Run the specified task once a week at midnight on Sunday. Equivalent to `0 0 * * 0`.
-   `@daily` - Run the specified task once a day at midnight. Equivalent to `0 0 * * *`.
-   `@hourly` - Run the specified task once an hour at the beginning of the hour. Equivalent to `0 * * * *`.
-   `@reboot` - Run the specified task at the system startup (boot-time).

### Crontab Variables

The cron daemon automatically sets several environment variables.

-   The default path is set to `PATH=/usr/bin:/bin`. If the command you are executing is not present in the cron specified path, you can either use the absolute path to the command or change the cron `$PATH` variable. You can’t implicitly append `:$PATH` as you would do with a regular script.
-   The default shell is set to `/bin/sh`. To change the different shell, use the `SHELL` variable.
-   Cron invokes the command from the user’s home directory. The `HOME` variable can be set in the crontab.
-   The email notification is sent to the owner of the crontab. To overwrite the default behavior, you can use the `MAILTO` environment variable with a list (comma separated) of all the email addresses you want to receive the email notifications. When `MAILTO` is defined but empty (`MAILTO=""`), no mail is sent

### Crontab Restrictions

The `/etc/cron.deny` and `/etc/cron.allow` files allows you to control which users have access to the `crontab` command. The files consist of a list of usernames, one user name per line.

By default, only the `/etc/cron.deny` file exists and is empty, which means that all users can use the crontab command. If you want to deny access to the crontab commands to a specific user, add the username to this file.

If the `/etc/cron.allow` file exists only the users who are listed in this file can use the `crontab` command.

If neither of the files exists, only the users with administrative privileges can use the `crontab` command.

### Anacron
**anacron** is a program that performs periodic command scheduling, which is traditionally done by cron, but without assuming that the system is running continuously.
If the machine is off when a scheduled job is supposed to run it will execute it once the machine is powered on. 
The program stores and reads its timestamp files in `/var/spool/anacron` by default but a different directory can be specified with `-S` option.

An anacrontab entry contains the following parts:
`period delay job-id command`
- period: A number giving the execution interval in days
- delay: A number that lets anacron wait for the specified amount of minutes before starting the job
- job-id: A string, used to identify the job. The timestamp files for each jobs are named after the job-id
- command: Any shell command to be executed

The relevant configuration file for anacron is `/etc/anacrontab`, but you also can find jobs in `/var/spool/anacron` and in user's home directory under `/home/$USER/.anacron`.

