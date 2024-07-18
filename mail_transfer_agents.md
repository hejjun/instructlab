# Ensure mail transfer agents are configured for local-only 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation

## Description:
Mail Transfer Agents (MTA), such as sendmail and Postfix, are used to listen for 
incoming mail and transfer the messages to the appropriate user or mail server. If the 
system is not intended to be a mail server, it is recommended that the MTA be 
configured to only process local mail.

## Rationale:
The software for all Mail Transfer Agents is complex and most have a long history of 
security issues. While it is important to ensure that the system can process local mail 
messages, it is not necessary to have the MTA's daemon listening on a port unless the 
server is intended to be a mail server that receives and processes mail from other 
systems.

## Audit:
Run the following commands to verify that the MTA is not listening on any non-loopback address ( 127.0.0.1 or ::1 )
```
ss -plntu | grep -P -- ':25\b' | grep -Pv --'\h+(127\.0\.0\.1|\[?::1\]?):25\b'
ss -plntu | grep -P -- ':465\b' | grep -Pv --'\h+(127\.0\.0\.1|\[?::1\]?):465\b'
ss -plntu | grep -P -- ':587\b' | grep -Pv --'\h+(127\.0\.0\.1|\[?::1\]?):587\b'
```
Nothing should be returned

## Remediation:
Edit /etc/postfix/main.cf and add the following line to the RECEIVING MAIL section. 
If the line already exists, change it to look like the line below:

`inet_interfaces = loopback-only`

Run the following command to restart postfix:

`systemctl restart postfix`

Page 316

## Note:
• This remediation is designed around the postfix mail server.
• Depending on your environment you may have an alternative MTA installed such 
as sendmail. If this is the case consult the documentation for your installed MTA 
to configure the recommended state