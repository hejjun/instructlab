Ensure journald is configured to compress large log files 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation
Description:
The journald system includes the capability of compressing overly large files to avoid 
filling up the system with logs or making the logs unmanageably large.
Rationale:
Uncompressed large files may unexpectedly fill a filesystem leading to resource 
unavailability. Compressing logs prior to write can prevent sudden, unexpected 
filesystem impacts.
Audit:
Review /etc/systemd/journald.conf and verify that large files will be compressed:
# grep ^\s*Compress /etc/systemd/journald.conf
Verify the output matches:
Compress=yes
Remediation:
Edit the /etc/systemd/journald.conf file and add the following line:
Compress=yes
Restart the service:
# systemctl restart systemd-journald.service
Additional Information:
The main configuration file /etc/systemd/journald.conf is read before any of the 
custom *.conf files. If there are custom configs present, they override the main 
configuration parameters.
It is possible to change the default threshold of 512 bytes per object before compression 
is used