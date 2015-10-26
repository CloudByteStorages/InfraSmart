### All About SCSI

#### related scsi transport technologies
- scsi fibre channel protocol 	(FCP)
- serial storage architecture 	(ssa)
- serial bus protocol 			(sbp)
- scsi over infiniband
- scsi over TCP/IP 				(iSCSI)

#### scsi over tcp alternatives
- ethernet
- ip
- udp
- sctp

#### tcp features
- automatic ack
- retransmission of lost & corrupted packets
- guaranteed in-order delivery
- congestion control

#### ip-family features
- ipsec		security
- slp		discovery
- dhcp		configuration

#### Layered Packet Format
- ethernet header	14
- ip header			20
- tcp header		20
- iscsi header		48
- data			...

#### drawbacks of using TCP
- limited by TCP window size
- lost TCP packets cause delay in delivery of subsequent packets
- TCP checksum not sufficient for storage data integrity
- usually involves multiple copying of data

#### iscsi sessions
- collection of tcp connections between an initiator & target
	- overcome bandwidth limitations imposed by TCP window size
	- utilize multiple CPUs in an SMP
- connections of a session may traverse different physical interconnects
	- aggregate bandwidth from multiple interconnects
- must coordinate between multiple TCP connections

#### Data Transfer Model
- asymmetric
	- single control channel
		- control channel used to transfer commands, status, task management 
	- multiple data channels
- symmetric
	- all channels identical


#### iscsi locations @ freebsd
- sys/modules/iscsi_initiator	-> just the Makefile
- sys/modules/iscsi				-> just the Makefile
- sys/dev/iscsi/
- sys/dev/iscsi_initiator
- usr.bin/iscsictl
- usr.bin/iscsid
- etc/rc.d/iscsictl
- etc/rc.d/iscsid

#### iscsi details @ freebsd
- freebsd/sys/dev/iscsi/icl.c	
	- common layer used by initiator & target
	- representation in terms of module
	- module name, priority, connection, limits


#### AHS
- Additional Header Segment
- AHS_type + AHS_length + AHS_data
- AHS_type			1 byte length
- AHS_length		1 byte length
- AHS_data			2-1018 bytes in length

#### AHS_type
- 0th bit				this PDU must be dropped if rx does not understand to interpret AHS
- 1st bit				reserved
- 2nd till 7th bits	any value from 0 to 63
	- 0-62 				reserved for iscsi assignment
	- 63 				implies this is not iscsi defined PDU


#### References
- https://www.research.ibm.com/haifa/Workshops/Storage2002/papers/iscsiDesign.pdf
