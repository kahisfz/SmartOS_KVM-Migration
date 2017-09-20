# SmartOS_KVM-Migration
In SmartOS migrate KVM or zone form one server to another using WAN or LAN connection
Migrating a KVM from one SmartOS host to another in WAN envorinment: A Step by Step Process
The following steps are esential to move or migrate a KVM from one SmartOS to another, In this process it is assumed that both KVMs are on same LAN segment or no firewall involved in highspeed WAN
1. Get the UIID and status of the KVM need migration
source_Server# vmadm list

2. Check for ZFS file system list to that VM 
source_Server#zfs list -o name | grep UUID

3. Check for xml cofiguration file of KVM
source_Server#/etc/zones/UUID.xml
4. Stop the KVM
source_Server#vmadm stop UUID

5. Create the snapshot of every zfs for example
[source_Server ~]# zfs snapshot zones/UUID@tosend
[source_Server ~]# zfs snapshot zones/UUID-disk0@tosend
[source_Server ~]# zfs snapshot zones/UUID-disk1@tosend
[source_Server ~]# zfs snapshot zones/cores/UUID@tosend

6. Send the snapshot to the destination server, it may take a lot of time depending upon the size of kvm, for example
[source_Server ~]# zfs send zones/UUID@tosend | ssh gz2.domain zfs receive -v zones/UUID
[source_Server ~]# zfs send zones/UUID-disk0@tosend | ssh destination.server zfs receive -v zones/UUID-disk0
[source_Server ~]# zfs send zones/UUID-disk1@tosend | ssh destination.server zfs receive -v zones/UUID-disk1
[source_Server ~]# zfs send zones/cores/UUID@tosend | ssh destination.server zfs receive -v zones/cores/UUID
7. Copy the line of VM from inside the /etc/zones/index in the source global zone, and paste it at the end of same file in the destination global zone.
[Source_Server ~]# cat /etc/zones/index | grep UUID
[Destination_Server ~]# echo "UUID:installed:/zones/UUID" >> /etc/zones/index
(PS.or you may simply nano editor)

8.Copy the xml configuration file from the source global zone, to the destination 
[source_Server ~]#scp /etc/zones/UUID.xml destination.server:/etc/zones/UUID.xml
