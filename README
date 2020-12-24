dpdk-dynamic_vhost is a modified DPDK version and the difference only 
appears in the `librte_vhost`. We modified the memory sharing methods in 
vhost integrated vSwitch to dynamically map VM memory pages, which is 
supposed to reduce the risks of illegal access to VM memory by vSwitch.

It's based on DPDK 17.05. All of these modifications are unaware to users 
and are easy to conduct on the latest versions.

## How does it work?

As the packet delivery from vSwitch buffer to VM memory is implemented in 
the vhost integrated vSwitch, the vSwitch is granted to access VM memory 
for memory copying. In DPDK `librte_vhost`, the VM's whole memory is shared
to the vSwitch by calling `mmap` function. That brings secure risks of 
illegal memory access.

So the goal of dpdk-dynamic_vhost is to reduce the memory size that VM 
needs to share with vSwitch. As the drivers in VM are unpredictable, it is 
impossible to separately share a block of I/O memory with vSwitch. That's
why we propose the dynamically memory mapping in dpdk-dynamic_vhost.

In dpdk-dynamic_vhost, only when the packet buffer in VM that vSwitch needs 
to access for memory copying has not been mapped, vSwitch will then only 
map one VM memory page on demand.

Thus, the VM memory that shared with vSwitch is greatly reduced, which reduces
the possible attack surface. Due to only limited number of VM memory pages 
need to be mapped and only mapped once when delivering the first packet, our 
modifications will not reduce performance.

## What's the difference compared with original vhost-user in `librte_vhost`?

The difference includes two points: the socket channel and memory mapping.

In original vhost-user, before the datapath established, there is an socket 
based channel to negotiate some configurations. The procedures are shown as 
the following figure:

<p align="left">
  <img src="./media/socket_channel_vhostuser.png"/>
<p/>

The most important step is to process signal `VHOST_USER_SET_MEM_TABLE`, which 
is to call `mmap` function to map VM's whole memory into vSwitch's memory 
address. And then vSwitch stores the relationship between VVA (vSwitch virtual 
address) and GPA (guest physical address) for future memory copying.

In dynamic_vhost, we modified the processing function of `VHOST_USER_SET_MEM_TABLE`
as below:

<p align="left">
  <img src="./media/socket_channel_dynamic.png"/>
<p/>

We do not call `mmap` function to map VM memory immediately, and only store the 
FD (file description) and address offset for futere map only when on demand. For 
example, as shown in the figure below, if vSwitch need to access a packet in an
unmapped page, it will call the `mmap` function to only map the particular page.

<p align="left">
  <img src="./media/memory_map.png"/>
<p/>

One flaw of dynamic_vhost is that it is only effective when using 2MB hugepages. 
If VM memory is created with 1GB hugepages, the shared memory size that 
dynamic_vhost can reduce is very limited.
