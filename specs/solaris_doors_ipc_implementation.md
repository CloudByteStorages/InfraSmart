# Solaris Doors IPC Implementation

Fast Sockets, An IPC Library to Boost Application Performance

Doors provide a mechanism for processes to issue remote procedure calls to functions in other processes running on the 
same system. The door APIs were developed by Sun Microsystems as a core part of the Spring operating system and were officially
available in Solaris 2.6. They are extensively used in Illumos.

In addition to the Solaris/Illumos port, which is well documented, there is also an outdated implementation for linux that can
serve for comparison. The project would consist in understanding how the existing code works and designing a completely new, 
but compatible, implementation for FreeBSD.

Requirements

- Interest in Inter-process Communications
- Ability to understand code and the existing implementations in Illumos and linux.
- Capacity to run your own tests and benchmarks.

http://www.scn.rain.com/~neighorn/PDF/fast_sockets.pdf
http://www.rampant.org/doors/linux-doors.pdf
