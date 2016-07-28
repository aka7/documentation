
# partition config for kickstart, on 8 disk system, software raid and lvm 
just for my reference, disk label may change
some examples of katello partition templates for different disks

````
# System bootloader configuration
zerombr
bootloader --location=mbr --driveorder=sda --append="crashkernel=auto audit=1"

clearpart --all --initlabel --drives=sda,sdb,sdc,sdd,sde,sdf,sdg,sdh

part raid.01  --size=250 --ondrive=sda
part raid.02 --size=250 --ondrive=sdb

part raid.11  --size=102400 --ondrive=sda
part raid.12 --size=102400 --ondrive=sdb

part raid.21  --size=10240 --grow --ondrive=sda
part raid.22 --size=10240 --grow --ondrive=sdb

part raid.31 --size=10240 --grow --ondrive=sdc
part raid.32 --size=10240 --grow --ondrive=sdd
part raid.33 --size=10240 --grow --ondrive=sde
part raid.34 --size=10240 --grow --ondrive=sdf
part raid.35 --size=10240 --grow --ondrive=sdg
part raid.36 --size=10240 --grow --ondrive=sdh

raid /boot --fstype ext4 --level=RAID1 --device=md0 raid.01 raid.02

raid pv.02 --device md1  --level=RAID1 raid.11 raid.12

raid pv.03 --device md2  --level=RAID1 raid.21 raid.22

raid pv.04 --device md3  --level=RAID10 raid.31 raid.32 raid.33 raid.34 raid.35 raid.36 


volgroup rootvg pv.02

logvol /     --fstype=ext4 --vgname=rootvg --size=10240 --name=root
logvol swap  --fstype=swap --vgname=rootvg --size=32768 --name=swap
logvol /var  --fstype=ext4 --vgname=rootvg --size=20480 --name=var
logvol /var/log/audit  --fstype=ext4 --vgname=rootvg --size=2048 --name=audit
logvol /tmp  --fstype=ext4 --vgname=rootvg --size=4096  --name=tmp
logvol /home --fstype=ext4 --vgname=rootvg --size=4096  --name=home

volgroup datavg pv.03 pv.04
````

## Two disk part config
````
# System bootloader configuration
clearpart --all --drives=sda,sdb
zerombr
bootloader --location=mbr --driveorder=sda --append="crashkernel=auto audit=1"

part raid.01  --size=250 --ondrive=sda
part raid.02 --size=250 --ondrive=sdb

part raid.11  --size=102400 --ondrive=sda
part raid.12 --size=102400 --ondrive=sdb

part raid.21  --size=10240 --grow --ondrive=sda
part raid.22 --size=10240 --grow --ondrive=sdb

raid /boot --fstype ext4 --level=RAID1 --device=md0 raid.01 raid.02

raid pv.02 --device md1  --level=RAID1 raid.11 raid.12

raid pv.03 --device md2  --level=RAID1 raid.21 raid.22

volgroup rootvg pv.02

logvol /     --fstype=ext4 --vgname=rootvg --size=10240 --name=root
logvol swap  --fstype=swap --vgname=rootvg --size=32768 --name=swap
logvol /var  --fstype=ext4 --vgname=rootvg --size=20480 --name=var
logvol /var/log/audit  --fstype=ext4 --vgname=rootvg --size=2048 --name=audit
logvol /tmp  --fstype=ext4 --vgname=rootvg --size=4096  --name=tmp
logvol /home --fstype=ext4 --vgname=rootvg --size=4096  --name=home

volgroup datavg pv.03
````

## two disk, katello template, for a KVM
````
# System bootloader configuration
zerombr
bootloader --location=mbr --driveorder=vda --append="crashkernel=auto audit=1"

clearpart --all --initlabel --drives=vda,vdb

part /boot --fstype=ext4 --size=250 --ondrive=vda
part pv.01 --size=10240 --grow --ondrive=vda
part pv.02 --size=10240 --grow --ondrive=vdb

volgroup rootvg pv.01

logvol /     --fstype=ext4 --vgname=rootvg --size=4096 --name=root
logvol swap  --vgname=rootvg --size=2048 --name=swap
logvol /var  --fstype=ext4 --vgname=rootvg --size=4096 --grow --name=var
logvol /var/log/audit  --fstype=ext4 --vgname=rootvg --size=2048 --name=audit
logvol /tmp  --fstype=ext4 --vgname=rootvg --size=2048  --name=tmp
logvol /home --fstype=ext4 --vgname=rootvg --size=512  --name=home

volgroup datavg pv.02
<% if @host.params['data_mount_name']-%>
logvol /<%= @host.params['data_mount_name'] -%> --fstype=ext4 --vgname=datavg --name=<%= @host.params['data_mount_name'] -%> --grow --size=1
<%else-%>
logvol /data --fstype=ext4 --vgname=datavg --name=data --grow --size=1024
<% end -%>
````

### 2 disks NO MIRROR
````
# System bootloader configuration
clearpart --all --drives=sda,sdb
zerombr
bootloader --location=mbr --driveorder=sda --append="crashkernel=auto audit=1"

part /boot --fstype=ext4 --size=1024 --ondrive=sda
part pv.01 --size=25600 --ondrive=sda
part pv.02 --size=1 --grow --ondrive=sda
part pv.03 --size=1 --grow --ondrive=sdb

volgroup rootvg pv.01

logvol /     --fstype=ext4 --vgname=rootvg --size=5120 --name=root
logvol swap  --fstype=swap --vgname=rootvg --size=8192 --name=swap
logvol /var  --fstype=ext4 --vgname=rootvg --size=1 --grow --name=var
logvol /var/log/audit  --fstype=ext4 --vgname=rootvg --size=2048 --name=audit
logvol /tmp  --fstype=ext4 --vgname=rootvg --size=2048  --name=tmp
logvol /home --fstype=ext4 --vgname=rootvg --size=512  --name=home

volgroup datavg pv.02 pv.03
````


