# Ensure password history remember is configured 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation

## Description:
The /etc/security/opasswd file stores the users' old passwords and can be checked to 
ensure that users are not recycling recent passwords.

• remember=<N> 

- <N> is the number of old passwords to remember

## Rationale:
Requiring users not to reuse their passwords make it less likely that an attacker will be 
able to guess the password or use a compromised password.
Note: These change only apply to accounts configured on the local system.

## Audit:
Run the following command and verify that the remember option is set to 24 or more 
and meets local site policy in /etc/security/pwhistory.conf:

`grep -Pi -- '^\h*remember\h*=\h*(2[4-9]|[3-9][0-9]|[1-9][0-9]{2,})\b' /etc/security/pwhistory.conf`

remember = 24

Run the following command to verify that the remember option is not set to less than 24 on the pam_pwhistory.so module in /etc/pam.d/password-auth and /etc/pam.d/system-auth:

`grep -Pi --'^\h*password\h+(requisite|required|sufficient)\h+pam_pwhistory\.so\h+([^#\n\r]+\h+)?remember=(2[0-3]|1[0-9]|[0-9])\b' /etc/pam.d/system-auth /etc/pam.d/password-auth`

Nothing should be returned
Page 594

## Remediation:
Edit or add the following line in /etc/security/pwhistory.conf:

remember = 24

Run the following script to remove the remember argument from the pam_pwhistory.so module in /etc/pam.d/system-auth and /etc/pam.d/password-auth:
```
#!/usr/bin/env bash
{
 for l_pam_file in system-auth password-auth; do
 l_authselect_file="/etc/authselect/$(head -1 
/etc/authselect/authselect.conf | grep 'custom/')/$l_pam_file"
 sed -ri 
's/(^\s*password\s+(requisite|required|sufficient)\s+pam_pwhistory\.so.*)(\s+
remember\s*=\s*\S+)(.*$)/\1\4/' "$l_authselect_file"
 done
 authselect apply-changes
}
```