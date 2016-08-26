##To find users with privileges related reportsserver group:
````
ldapsearch -h ldap.mydomain.com -D 'uid=abdul.karim,ou=People,dc=mydomain,dc=com' -W -x -b 'ou=groups,ou=reportserver,ou=Applications,dc=mydomain,dc=com'
````
##To find users in www group:
````
ldapsearch -h ldap.mydomain.com -D 'uid=abdul.karim,ou=People,dc=mydomain,dc=com' -W -x -b 'ou=Group,dc=mydomain,dc=com' 'cn=www'
`````

To find users with SUDO privileges:
````
ldapsearch -h ldap.mydomain.com -D 'uid=abdul.karim,ou=People,dc=mydomain,dc=com' -W -x -b 'ou=SUDOers,dc=mydomain,dc=com' 
````
