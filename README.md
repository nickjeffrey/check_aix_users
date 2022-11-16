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


# Hints
If you are running this script in a low-security environment that does not care about  things like maximum password age, you may be tempted to just not use this script at all.
Don't do that, as you will miss out on things like sanity checks for the root account.
Instead, just comment out the check_minlen, check_maxage subroutines at the very end of the script.

# Sample Output


```
AIX users OK - all user accounts within normal parameters
```

```
AIX users WARN - janedoe,johnsmith user account(s) locked out.  Potential breakin attempt or legitimate user error.  Please investigate. When done, reset failed login count with: chsec -f /etc/security/lastlog -a unsuccessful_login_count=0 -s userid 
```

```
AIX users WARN - root user can login remotely.  This is considered a security risk.  You should force users to login with their own userid, then su or sudo to root.  Fix up with: chuser rlogin=false root
```

```
AIX users WARN - janedoe,johnsmithaccount(s) have a maximum password age that is too long, so the passwords are not prompted to be regularly changed. This is a security risk.  Please fix up with: chuser maxage=## userid
```

```
AIX users WARN - janedoe,johnsmith account(s) do not get locked out after a reasonable number of bad password attempts.  This is a security risk, because it does not protect the system against brute force password attacks.  Please fix up with: chuser loginretries=## userid
```

```
AIX users WARN - janedoe,johnsmith account(s) have a minimum password length that is too short.  This is a security risk.  Please fix up with: chuser minlen=## userid
```

