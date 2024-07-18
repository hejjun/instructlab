# Ensure nis server services are not in use (Automated)
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation

## Description:
The Network Information Service (NIS), formerly known as Yellow Pages, is a clientserver directory service protocol used to distribute system configuration files. The NIS 
client ( ypbind ) was used to bind a machine to an NIS server and receive the 
distributed configuration files.

## Rationale:
The NIS service is inherently an insecure system that has been vulnerable to DOS 
attacks, buffer overflows and has poor authentication for querying NIS maps. NIS 
generally has been replaced by such protocols as Lightweight Directory Access 
Protocol (LDAP). It is recommended that the service be removed.

## Impact:
There may be packages that are dependent on the ypserv package. If the ypserv
package is removed, these dependent packages will be removed as well. Before 
removing the ypserv package, review any dependent packages to determine if they are 
required on the system.
-IF- a dependent package is required: stop and mask the ypserv.service leaving the 
ypserv package installed.
Page 284

## Audit:
Run the following command to verify ypserv is not installed:

`rpm -q ypserv`

package ypserv is not installed

-OR-

-IF- the package is required for dependencies:
Run the following command to verify ypserv.service is not enabled:

`systemctl is-enabled ypserv.service 2>/dev/null | grep 'enabled'`

Nothing should be returned

Run the following command to verify ypserv.service is not active:

`systemctl is-active ypserv.service 2>/dev/null | grep '^active'`

Nothing should be returned

Note: If the package is required for a dependency
• Ensure the dependent package is approved by local site policy
• Ensure stopping and masking the service and/or socket meets local site policy

## Remediation:
Run the following commands to stop ypserv.service and remove ypserv package:
```
systemctl stop ypserv.service
dnf remove ypserv
```
-OR-

-IF- the ypserv package is required as a dependency:

Run the following commands to stop and mask ypserv.service:
```
systemctl stop ypserv.service
systemctl mask ypserv.service
```