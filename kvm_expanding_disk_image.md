
Steps taken from https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Administration_Guide/sect-expand-disk-image.html

steps, below increases lvm vol by 10G and allocate all free space to /tmp

````
virt-list-partitions -lh /dev/datavg/script-dev01-root
/dev/sda1 ext4 250.0M
/dev/sda2 pv 14.8G

virt-df -h /dev/datavg/script-dev01-root 
Filesystem                                Size       Used  Available  Use%
script-dev01-root:/dev/sda1               238M        74M       152M   31%
script-dev01-root:/dev/rootvg/audit       1.9G        28M       1.8G    2%
script-dev01-root:/dev/rootvg/home        488M       396K       462M    1%
script-dev01-root:/dev/rootvg/root        3.8G       2.1G       1.5G   56%
script-dev01-root:/dev/rootvg/tmp         1.9G       3.0M       1.8G    1%
script-dev01-root:/dev/rootvg/var         4.1G       433M       3.4G   11%


lvrename /dev/datavg/script-dev01-root /dev/datavg/script-dev01-root.bk
lvcreate -n script-dev01-root --size 25G datavg

virt-resize /dev/datavg/script-dev01-root.bk /dev/datavg/script-dev01-root --expand /dev/sda2 --LV-expand /dev/rootvg/tmp

````

Or allocate size

````
virt-resize /dev/datavg/script-dev01-root.bk /dev/datavg/script-dev01-root  --resize /dev/sda1=500M --expand /dev/sda2 --LV-expand /dev/rootvg/tmp
````
The first two arguments are the input disk and output disk. --resize /dev/sda1=500M resizes the first partition up to 500MB. --expand /dev/sda2 expands the second partition to fill all remaining space. --LV-expand /dev/rootvg/tmp expands the guest virtual machine logical volume to fill the extra space in the second partition.

