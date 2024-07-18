4.3.2 Ensure sudo commands use pty (Automated)
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation
Description:
sudo can be configured to run only from a pseudo terminal (pseudo-pty).
Rationale:
Attackers can run a malicious program using sudo which would fork a background 
process that remains even when the main program has finished executing.
Impact:
WARNING: Editing the sudo configuration incorrectly can cause sudo to stop 
functioning. Always use visudo to modify sudo configuration files.
Audit:
Verify that sudo can only run other commands from a pseudo terminal.
Run the following command:
# grep -rPi '^\h*Defaults\h+([^#\n\r]+,)?use_pty(,\h*\h+\h*)*\h*(#.*)?$' 
/etc/sudoers*
/etc/sudoers:Defaults use_pty
Remediation:
Edit the file /etc/sudoers with visudo or a file in /etc/sudoers.d/ with visudo -f
<PATH_TO_FILE> and add the following line:
Defaults use_pty
Note:
• sudo will read each file in /etc/sudoers.d, skipping file names that end in ~ or 
contain a . character to avoid causing problems with package manager or editor 
temporary/backup files.
• Files are parsed in sorted lexical order. That is, /etc/sudoers.d/01_first will be 
parsed before /etc/sudoers.d/10_second.
• Be aware that because the sorting is lexical, not numeric, 
/etc/sudoers.d/1_whoops would be loaded after /etc/sudoers.d/10_second.
• Using a consistent number of leading zeroes in the file names can be used to 
avoid such problems