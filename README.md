# check_aix_filesystems
nagios check to confirm all AIX filesystems are mounted correctly
This checks for both "normal" filesystems, as well as NFS-mounted filesystems.
NFS filesystems can be tricky - this will tell us if a filesystem goes away / hangs / etc.  
This is important to know, as some commands (df, etc) will hang indefinitely while waiting for a response from an NFS filesystem.


# Requirements
ksh, ssh, sudo on nagios server

# Configuration

This script is executed remotely on a monitored system by the NRPE or check_by_ssh methods available in nagios.  

If you are using the check_by_ssh method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
    define service {
       use                             generic-service
       hostgroup_name                  all_aix
       service_description             AIX filesystems
       check_command                   check_by_ssh!/usr/local/nagios/libexec/check_aix_filesystems
       }
```

Alternatively, if you are using the check_nrpe method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
   define service{
      use                             generic-service
      hostgroup_name                  all_aix
      service_description             AIX filesystems
      check_command                   check_nrpe!check_aix_filesystems -t 30
      notification_options            c,r                     ; Send notifications about critical and recovery events
      }
```

If you are using the NRPE method, you will also need a command definition similar to the following on each monitored host in the /usr/local/nagios/nrpe/nrpe.cfg file:
```
    command[check_aix_users]=/usr/local/nagios/libexec/check_aix_users
```


This script executes as the nagios user, but there are a few steps that need to be executed with root privileges.
We handle this by using sudo to execute certain commands, so you will need to ensure that sudo is installed, and entries similar to the following exist in the /etc/sudoers file:
```
    User_Alias      NAGIOS_USER = nagios
    Cmnd_Alias      LS = /usr/sbin/ls
    NAGIOS_USER ALL = (root) NOPASSWD: LS
```
