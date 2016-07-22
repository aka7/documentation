# set to pxe boot at next logon
```
ipmitool -H <ip> -U <user> -P <password> chassis bootdev pxe
```
Can also be run from OS if ipmitool is installed 

#change ilom password using ipmitool for oracle hardware ( worked on Oracle X4-2L)
# first get get user id
````
ipmitool -I lanplus -H <ip> -U <username> user list 1
```
# assuming the user you want change password for has id of 2, run  (worked on Oracle X4-2L)
```
ipmitool -I lanplus -H i<ip> -U <username> user set password 2 <newpassword>
```
