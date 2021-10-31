# Linux Terminal
## Commands
### Systemd
- `systemctl`
- `systemctl <start|stop|status|enable|disable|is-enabled|restart|condrestart|reload|> <service>`
- `journalctl -f`
- `hostnamectl --static set-hostname <hostname>` (Edit `/etc/hosts` also)
- `hostname`
- `timedatectl`
- `systemd-analyze time` : Boot time
### Runlevel
- `who -r` (SysV)
- `init <#>` (SysV)
- `systemctl get-default`
- `systemctl list-units --type=target`
- `systemctl isolate <multi-user|graphical|...>.target`
- `systemctl set-default <multi-user|graphical|...>.target`
### Super user
- `passwd root`
- `adduser <username> sudo` (`/etc/sudoers`)
- `sudo -i`
- `sudo sh -c "<command>"`
### File system
- `ls -a -i -l [folder]`
- `ln [-s] <file> <link>`
- `stat <file>`
- `df -T`
- `du --human-readable --summarize <folder>`
- `lsblk`
- `mount`
- `fdisk --list [img-file]`
- `dd if=<input-file> of=<output-file>`
- `mount <>`, `umount <>`
### Users and groups
- `groups [user]`
- `adduser <user>`
- `groupadd <new-group>`
- `usermod -a -G <group1,group2,...>`
- `su <user>`
- `deluser --remove-home <user>`
- `chgrp <group> <file>`
- `chown <user> <file>`
- `chmod <u=rwx,g=rwx,o=rwx> <file>`
### Search
- `find <folder> -name <file>`
- `grep -rn <folder> -e "<regex>"`
- `whereis <binary>`
### Pipes 
- `|`
- `tee`
### Filter
- `sort`, `wc`, `head`, `tail`, `grep`, `xargs`
### Echo
- `echo -e "<string-with-escape-characters>"`
- `echo $?` : Exit status
### Diff
- `diff -y -W<#> <file-1> <file-2>` 
### Tar
- `tar cvf<z|j|J> <archive-file>.tar.<gz|bz2|xz> <in-folder>`
- `tar xvf <archive> [file]` : Extract
- `tar rvf <archive> <file>` : Add file to archive
### Process
- `ps [aux]`
- `Ctrl + C` or `kill <PID#>` or `pkill <name>` : Kill process
- `Ctrl + Z`, `bg` or `<command> &` : Run in background 
- `fg`
## Shortcuts
- `Ctrl + L` : Clear terminal
- `Ctrl + A` : Go to start
- `Ctrl + E` : Go to end
## Output Redirection
- `>` : Write
- `>>` : Append