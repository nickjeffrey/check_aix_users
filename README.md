# check_aix_users
nagios check for AIX user accounts (locked out, expired passwords)

# Requirements
perl, ssh, sudo on nagios server

# Configuration

This script is executed remotely on a monitored system by the NRPE or check_by_ssh methods available in nagios.  

If you are using the check_by_ssh method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
    define service {
       use                             generic-service
       hostgroup_name                  all_aix
       service_description             AIX users
       normal_check_interval           15                       ; Check every 15 minutes under normal conditions
       check_command                   check_by_ssh!/usr/local/nagios/libexec/check_aix_users
       }
```

Alternatively, if you are using the check_nrpe method, you will need to add a section similar to the following to the services.cfg file on the nagios server.  
This assumes you already have ssh key pairs configured.
```
   define service{
      use                             generic-service
      hostgroup_name                  all_aix
      service_description             AIX users
      normal_check_interval           15                       ; Check every 15 minutes under normal conditions
      check_command                   check_nrpe!check_aix_users -t 30
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
    Cmnd_Alias      LSUSER = /usr/sbin/lsuser
    NAGIOS_USER ALL = (root) NOPASSWD: LSUSER
```


# HINTS
If you are running this script in a low-security environment that does not care about  things like maximum password age, you may be tempted to just not use this script at all.
Don't do that, as you will miss out on things like sanity checks for the root account.
Instead, just comment out the check_minlen, check_maxage subroutines at the very end of the script.

