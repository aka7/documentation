# quick lvm commands

rescan a device after disk size has been increased and allocate the size to lvm vol

Remember it depends on which disk, example is for /dev/sdd
```
echo 1 > /sys/block/sdd/device/rescan
pvresize /dev/sdd
lvresize -r -l+100%FREE /dev/mapper/dataVG01-optVol01 (uses entire volume)

#or to extend by say 10GB only:
lvresize -rL +10G /dev/mapper/dataVG01-optVol01

```
run reszie4fs to see the allocated size on vol

```
resize4fs /dev/mapper/dataVG01-optVol01
df -h /dev/mapper/dataVG01-optVol01
```

# create a new vol
```
lvcreate -L 20G -n lv_data02 dataVG01
mkfs.xfs /dev/mapper/dataVG01-lv_data02 (for ext4, use mkfs.ext4)

#mount it

```

scan-scsi-lun

needed if a new disk has been added for vm.

```
# ls  -l /sys/class/scsi_host/host*/scan

# --w------- 1 root root 0 Nov  2 15:04 /sys/class/scsi_host/host0/scan

# --w------- 1 root root 0 Nov  2 15:04 /sys/class/scsi_host/host1/scan

# echo "- - -" > /sys/class/scsi_host/host1/scan
# echo "- - -" > /sys/class/scsi_host/host2/scan

fdisk -l

```

# add new disk to VG (sde and sdf)

```
fdisk -l
vgextend dataVG01 /dev/sde
vgextend dataVG01 /dev/sdf
pvs
```

