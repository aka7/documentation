## To import your public key to your RPM DB

````
rpm --import RPM-GPG-KEY-filename
````
## View rpm gpg public key loaded on RPM DB 
````
 rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'
````

## Before the signing, configure your ~/.rpmmacros file to include the following
````
%_signature gpg
%_gpg_name  <name of gpg>
````

## sign your custom RPM package
````
rpm --addsign test.rpm
rpm --checksig test.rpm
````
