# Ensure rpcbind services are not in use (Automated)
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation

## Description:
The rpcbind utility maps RPC services to the ports on which they listen. RPC processes 
notify rpcbind when they start, registering the ports they are listening on and the RPC 
program numbers they expect to serve. The client system then contacts rpcbind on the 
server with a particular RPC program number. The rpcbind.service redirects the client 
to the proper port number so it can communicate with the requested service.
Portmapper is an RPC service, which always listens on tcp and udp 111, and is used to 
map other RPC services (such as nfs, nlockmgr, quotad, mountd, etc.) to their 
corresponding port number on the server. When a remote host makes an RPC call to 
that server, it first consults with portmap to determine where the RPC server is listening.

## Rationale:
A small request (~82 bytes via UDP) sent to the Portmapper generates a large 
response (7x to 28x amplification), which makes it a suitable tool for DDoS attacks. If 
rpcbind is not required, it is recommended to remove rpcbind package to reduce the 
potential attack surface.

## Impact:
Many of the libvirt packages used by Enterprise Linux virtualization, and the nfs-utils
package used for The Network File System (NFS), are dependent on the rpcbind
package. If the rpcbind package is removed, these dependent packages will be 
removed as well. Before removing the rpcbind package, review any dependent 
packages to determine if they are required on the system.
-IF- a dependent package is required: stop and mask the rpcbind.socket and 
rpcbind.service leaving the rpcbind package installed.
Page 290

## Audit:
Run the following command to verify rpcbind package is not installed:

`rpm -q rpcbind`

package rpcbind is not installed

-OR-

-IF- the rpcbind package is required as a dependency:

Run the following command to verify rpcbind.socket and rpcbind.service are not 
enabled:

`systemctl is-enabled rpcbind.socket rpcbind.service 2>/dev/null | grep 'enabled'`

Nothing should be returned

Run the following command to verify rpcbind.socket and rpcbind.service are not 
active:

`systemctl is-active rpcbind.socket rpcbind.service 2>/dev/null | grep '^active'`

Nothing should be returned

Note: If the package is required for a dependency

• Ensure the dependent package is approved by local site policy
• Ensure stopping and masking the service and/or socket meets local site policy

## Remediation:
Run the following commands to stop rpcbind.socket and rpcbind.service, and remove the rpcbind package:
```
systemctl stop rpcbind.socket rpcbind.service
dnf remove rpcbind
```
-OR-

-IF- the rpcbind package is required as a dependency:
Run the following commands to stop and mask the rpcbind.socket and rpcbind.service:
```
systemctl stop rpcbind.socket rpcbind.service
systemctl mask rpcbind.socket rpcbind.service
```