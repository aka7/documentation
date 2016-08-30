
## Attach new interface
````
brctl show
bridge name     bridge id               STP enabled     interfaces
br0             8000.002128e7817b       no              bond0
                                                        vnet0
                                                        vnet1
                                                        vnet2
br1             8000.002128e7817a       no              eth0
virbr0          8000.52540030247a       yes             virbr0-nic
````

I want to attach an interface for br1 for kvm host aktest, aktest already has an interface on br0, to see which interface, run domiflist

````
virsh domiflist aktest
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet2      bridge     br0        virtio      52:54:00:8d:45:70

````
vnet2 is attached to aktest. 

So to add another interface on aktest host, on br1 bridge, first generate a mac address, then run attach-interface with required details.

````
virsh attach-interface aktest --type bridge --source br1 --model virtio --mac 00:16:3e:40:dc:7f --config
````

After its attached.
````
virsh domiflist aktest
Interface  Type       Source     Model       MAC
-------------------------------------------------------
vnet2      bridge     br0        virtio      52:54:00:8d:45:70
vnet3      bridge     br1        virtio      00:16:3e:40:dc:7f

````
now configure the interface on vm.

For new macvtap interface types, the attach-interface doesn't understand --type direct. so you can create a interface like above and run virsh edit and change it settings to type direct, like so.



macvtap interface for aktest01 below

````
virsh domiflist aktest01
Interface  Type       Source     Model       MAC
-------------------------------------------------------
macvtap2   direct     br250      virtio      00:16:3e:58:7f:49

````

Add a vnet interface like below.
`````
#virsh attach-interface aktest01  --source br250 --model virtio --type bridge --mac 00:16:3e:1b:c9:c7 --config
Interface attached successfully
`````

now, config and interface will look like this,  which will be wrong.
````
# virsh domiflist aktest01
Interface  Type       Source     Model       MAC
-------------------------------------------------------
macvtap2   direct     br250      virtio      00:16:3e:58:7f:49
vnet1      bridge     br250      virtio      00:16:3e:1b:c9:c7
````

Edit  and change the config to macvtap
````
root@nippi# virsh edit aktest01.nme.bllon.sns 

````

for direct macvtap driver, the config should look like this

````
    <interface type='direct'>
      <mac address='00:16:3e:58:7f:49'/>
      <source dev='br250' mode='vepa'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
    </interface>
    <interface type='direct'>
      <mac address='00:16:3e:1b:c9:c7'/>
      <source dev='br250' mode='vepa'/>
      <model type='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x08' function='0x0'/>
    </interface>
````

power off and on the vm aktest01. 

once the config is applied, the output of domiflist will look like this.

````
# virsh domiflist aktest01
Interface  Type       Source     Model       MAC
-------------------------------------------------------
macvtap2   direct     br250      virtio      00:16:3e:58:7f:49
macvtap3   direct     br250      virtio      00:16:3e:1b:c9:c7

````
Configure the interface as normal on the VM.

To remove the interface

````
# virsh detach-interface aktest01 --type direct --mac 00:16:3e:1b:c9:c7
Interface detached successfully
````
