
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
virsh attach-interface aktest --type bridge --source br1 --model virtio --mac 00:16:3e:40:dc:7f
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

For new macvtap interface types, the attach-interface doesn't understand --type direct. so you can create a interface like above and run virsh edit and change it. not tested.
