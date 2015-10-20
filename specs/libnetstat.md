# LibNetstat

This project is about creating wrapper libraries to support monitoring and management applications to avoid direct use
of kvm(3) and sysctl(3) interfaces. This approach would allow the kernel implementation to change and monitoring applications 
to be extended without breaking applications and requiring them to be recompiled. For this project, we propose to provide such
a library for the network statistics functions (i.e. libnetstat(3)) as a proof of concept, together with methods for building 
similar libraries for other subsystems.

The initial idea is based on Robert Watson's libmemstat(3) library, where he created an abstract interface for collecting
and providing information on memory allocation statistics. 

Its design and development was inspired by the following needs:
- Robustness for Application Binary Interface (ABI) over time. 
- A library interface, rather than a system call, sysctl(8) variable, or kernel memory interface is the documented interface, 
and is designed to be robust against common changes to kernel data structures. 
- For example, the struct inpcb (for Protocol Control Blocks) grows frequently as TCP gains new features, but the current 
sysctl(3) interface is not robust against that growth.
- Robustness for different word sizes. 
32-bit monitoring tools should be able to work on a 64-bit kernel.
- Improved consistency when multiple sources are available. 
For example, there used to be a number of bugs involving netstat(1) running on crash dumps, where it will once in a while 
show a value from the current running kernel instead because a new statistics for sysctl(8) has been added with no crash dump
use in mind.

# Reference
https://wiki.freebsd.org/LibNetstat
