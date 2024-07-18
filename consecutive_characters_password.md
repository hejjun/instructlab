Ensure password same consecutive characters is configured 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation
Description:
The pwquality maxrepeat option sets the maximum number of allowed same 
consecutive characters in a new password.
Rationale:
Use of a complex password helps to increase the time and resources required to 
compromise the password. Password complexity, or strength, is a measure of the 
effectiveness of a password in resisting attempts at guessing and brute-force attacks.
Password complexity is one factor of several that determines how long it takes to crack 
a password. The more complex the password, the greater the number of possible 
combinations that need to be tested before the password is compromised.
Audit:
Run the following command to verify that the maxrepeat option is set to 3 or less, not 0, 
and follows local site policy:
# grep -Psi -- '^\h*maxrepeat\h*=\h*[1-3]\b' /etc/security/pwquality.conf 
/etc/security/pwquality.conf.d/*.conf
Example output:
/etc/security/pwquality.conf.d/50-pwrepeat.conf:maxrepeat = 3
Verify returned value(s) are 3 or less, not 0, and meet local site policy
Run the following command to verify that maxrepeat is not set, is 3 or less, not 0, and 
conforms to local site policy:
grep -Psi --
'^\h*password\h+(requisite|required|sufficient)\h+pam_pwquality\.so\h+([^#\n\
r]+\h+)?maxrepeat\h*=\h*(0|[4-9]|[1-9][0-9]+)\b' /etc/pam.d/system-auth 
/etc/pam.d/password-auth
Nothing should be returned