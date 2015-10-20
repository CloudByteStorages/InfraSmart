# SCSI Passthrough via CTL

CAM Target Layer is a framework to implement virtual SCSI drivers, which can be then "served" to other machines using
e.g. Fibre Channel or iSCSI. The purpose of this project would be to export physical SCSI devices through CTL. 
From the user point of view, this would eg. allow for exporting DVD or tape drive through iSCSI.

Requirements:

Ability to read and understand foreign C code.
Ability to write C code.
Ability to read and understand T10 (SCSI) specifications

ref - https://wiki.freebsd.org/SummerOfCodeIdeas#Add_SCSI_passthrough_to_CTL
