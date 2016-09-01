# sample example for using ldap and sssd with PAM

# Install sssd and authconfig if they aren’t already. 
The packages are:
```
sssd-client
sssd-common
sssd-common-pac
sssd-ldap
sssd-proxy
python-sssdconfig
authconfig
authconfig-gtk
```


#config sssd.conf
```
cat /etc/sssd/sssd.conf 
[domain/LDAP]
auth_provider = ldap
cache_credentials = true
chpass_provider = ldap
debug_level = 4
entry_cache_timeout = 60
enumerate = false
id_provider = ldap
ldap_access_filter = (&(objectClass=posixAccount)(!(employeeType=disabled))(ou=IT))
ldap_default_authtok = sfh23RIGV
ldap_default_authtok_type = password
ldap_default_bind_dn = uid=nssuser,ou=LdapAdmins,dc=example,dc=com
ldap_id_use_start_tls = true
ldap_schema = rfc2307
ldap_search_base = dc=exmaple,dc=com
ldap_sudo_search_base = ou=SUDOers,dc=example,dc=com
ldap_sudorule_runasuser = sudoRunAs
ldap_tls_cacertdir = /etc/openldap/cacerts/
ldap_tls_reqcert = demand
ldap_uri = ldaps://ldap.exmaple.com/
ldap_user_search_base = ou=People,dc=example,dc=com?sub?(&(objectClass=posixAccount)(!(employeeType=disabled))(ou=IT))
min_id = 1
```

# openldap.conf
cat /etc/openldap/ldap.conf
``` 
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

#BASE	dc=example,dc=com
#URI	ldap://ldap.example.com ldap://ldap-master.example.com:666

#SIZELIMIT	12
#TIMELIMIT	15
#DEREF		never

TLS_CACERTDIR /etc/openldap/cacerts/
URI ldaps://ldap.exmaple.com/
BASE dc=example,dc=com

sudo_provider = ldap

[nss]

[pam]

[sssd]
config_file_version = 2
domains = LDAP
services = nss,pam,sudo
```


# PAM
in /etc/pam.d/login and /etc/pam.d/sshd add line 
```
account    required     pam_access.so
```

# Control user/group access using pam_access

in /etc/security/access.conf 

```
#

# allow only the groups listed
+ : admin : ALL
+ : root : ALL
+ : group1 : ALL

# default deny
- : ALL : ALL
```

## change nsswitch to use sss for everything you want to  go to ldap
````
$ cat /etc/nsswitch.conf
passwd:     files sss
shadow:     files sss
group:      files sss
sudoers:    files sss
hosts:      files dns
bootparams: files
ethers:     files
netmasks:   files
networks:   files
protocols:  files
rpc:        files
services:   files
netgroup:   files
publickey:  files
automount:  files sss
aliases:    files
````

# Debuging sssd
run in foreground in debug mode

```
sssd -i -d 7
```
