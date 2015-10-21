# netmap

### VALE
- fast virtual ethernet

### netmap pipes
- shared memory packet transport channel

### details
- for both userspace & kernel clients
- vale - in-kernel
- faster than sockets, bpf, tun/tap, native switches, pipes
- 14.88 million packets per second with 1 core on a 10 Gbit NIC
- 20 Mpps per core for VALE ports
- 100 Mpps for netmap pipes
- userspace clients can send/receive raw packets through memory mapped buffers
- supports non-blocking IO via ioctls
- supports sync & blocking IO via file descriptor
- standard OS mechanisms e.g. select, poll, epool, kqueue
- port are ring buffers in a mmapped region
- one ring for transmit/receive of virtual port
- bind a fd to a port & netmap client can send or receive packets in batches via the ring
- can implement zero-copy forwarding between ports
- All NICS in netmap mode use same memory region
- region is accessible to all processes who own the /dev/netmap fds bound to NICs
- independent VALE & netmap pipe ports by default use separate memory regions but can be configured otherwise

### VALE & netmap pipes can be created dynamically 
- providing high speed packet IO between processes
- providing high speed packet IO between VMs
- providing high speed packet IO between NICs
- providing high speed packet IO between host stack

For best performance, netmap requires explicit support in device drivers

netmap <-> port <-> NIC
netmap <-> port <-> host stack
netmap <-> port <-> VALE switch

fd = open("/dev/netmap");
ioctl(fd, NIOCREGIF, (struct nmreq *)arg);

- above will create port , ring, vale dynamically

What happens behind the scenes:
While a NIC is in netmap mode, OS will believe the interface is up & running.
OS-generated packets for that NIC end up into a netmap ring & another ring is used to send packets to the OS network stack.
A close on the fd removes the binding & returns the NIC to normal mode i.e.
- reconnect the data path to the host stack
- destroys the virtual port
