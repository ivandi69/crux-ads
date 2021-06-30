# crux-ads
## CRUX active directory authentication

* **Install krb5 first**
* **Install openldap**
* Optionally install avahi then cups
* Optionally rebuild openssh
* **Install samba and cifs-utils**

## Create or modify the configs below

## Join the directory
```
net ads join -U <your ad user name>
```

Make sure winbindd service is running on boot.
Optionally run samba service for shares.

**At this point you should be able to login as AD user**

# CONFIGURATION
## DHCP
Add --fqdn to OPTS in /etc/rc.d/dhcpcd\
dhcpcd(8): Requests that the DHCP server updates DNS using FQDN instead of just a hostname.

## /etc/samba/smb.conf
```
[global]
    realm = EXAMPLE.NET
    workgroup = EXAMPLE
    security = ADS

    idmap config *:backend = tdb
    idmap config *:range = 3000-7999

    idmap config EXAMPLE.NET:backend = rid
    idmap config EXAMPLE.NET:schema_mode = rfc2307
    idmap config EXAMPLE.NET:range = 70001-80000
    idmap config EXAMPLE.NET:unix_nss_info = yes
    idmap config EXAMPLE.NET:unix_primary_group = yes

    template shell = /bin/bash
    template homedir = /home/%U

    winbind use default domain = yes

    winbind enum users = yes
    winbind enum groups = yes
    winbind nested groups = yes

    winbind refresh tickets = yes
    winbind offline logon = yes

    kerberos method = secrets and keytab
    include system krb5 conf = no

#    usershare path = /var/lib/samba/usershares
#    usershare max shares = 10
#    usershare allow guests = yes
#    usershare owner only = yes

[homes]
    comment = Home Directories
    browseable = no
    writable = yes

[printers]
    comment = All Printers
    path = /var/spool/samba
    browseable = no
    guest ok = no
    writable = no
    printable = yes
```

## /etc/security/pam_winbind.conf
```
[global]

# request a cached login if possible
# (needs "winbind offline logon = yes" in smb.conf)
cached_login = yes

# authenticate using kerberos
krb5_auth = yes

# when using kerberos, request a "FILE" or "DIR" krb5 credential cache type
# (leave empty to just do krb5 authentication but not have a ticket
# afterwards)
krb5_ccache_type = FILE

# password expiry warning period in days
warn_pwd_expire = 14

# create homedirectory on the fly
mkhomedir = yes
```


## /etc/krb5.conf
```
ln -s /var/lock/samba/smb_krb5/krb5.conf.EXAMPLE /etc/krb5.conf
```

## /etc/pam.d/common-auth
```
auth    [success=1 default=ignore]                       pam_localuser.so
auth    [success=1 new_authtok_reqd=done default=die]    pam_winbind.so
auth    required                                         pam_unix.so
auth    required                                         pam_permit.so
```

## /etc/pam.d/common-account
```
account    [success=1 default=ignore]                       pam_localuser.so
account    [success=1 new_authtok_reqd=done default=die]    pam_winbind.so
account    required                                         pam_unix.so
account    required                                         pam_permit.so
```

## /etc/pam.d/common-session
```
session    [success=1 default=ignore]                       pam_localuser.so
session    [success=1 new_authtok_reqd=done default=die]    pam_winbind.so
session    required                                         pam_unix.so
session    required                                         pam_permit.so
```

## /etc/pam.d/common-password
```
password    [success=1 default=ignore]                       pam_localuser.so
password    [success=1 new_authtok_reqd=done default=die]    pam_winbind.so
password    required                                         pam_unix.so
password    required                                         pam_permit.so
```

## /etc/nsswitch.conf
```
passwd:      files winbind
group:       files winbind
shadow:      files

hosts:       files dns
networks:    files

services:    files
protocols:   files
rpc:         files
publickey:   files
ethers:      files
netmasks:    files
netgroup:    files
bootparams:  files

automount:   files
aliases:     files
```
