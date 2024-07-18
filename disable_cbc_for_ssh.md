Ensure system wide crypto policy disables cbc for ssh 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation
Description:
Cypher Block Chaining (CBC) is an algorithm that uses a block cipher.
Rationale:
A vulnerability exists in SSH messages that employ CBC mode that may allow an 
attacker to recover plaintext from a block of ciphertext. If exploited, this attack can 
potentially allow an attacker to recover up to 32 bits of plaintext from an arbitrary block 
of ciphertext from a connection secured using the SSH protocol.
Impact:
CBC ciphers might be the only common cyphers when connecting to older SSH clients 
and servers
Page 190
Audit:
Run the following script to verify CBC is disabled for SSH:
#!/usr/bin/env bash
{
 l_output="" l_output2=""
 if grep -Piq -- '^\h*cipher\h*=\h*([^#\n\r]+)?-CBC\b' /etc/cryptopolicies/state/CURRENT.pol; then
 if grep -Piq -- '^\h*cipher@(lib|open)ssh(-server|-client)?\h*=\h*' 
/etc/crypto-policies/state/CURRENT.pol; then
 if ! grep -Piq -- '^\h*cipher@(lib|open)ssh(-server|-
client)?\h*=\h*([^#\n\r]+)?-CBC\b' /etc/crypto-policies/state/CURRENT.pol; 
then
 l_output="$l_output\n - Cipher Block Chaining (CBC) is disabled 
for SSH"
 else
 l_output2="$l_output2\n - Cipher Block Chaining (CBC) is enabled 
for SSH"
 fi
 else
 l_output2="$l_output2\n - Cipher Block Chaining (CBC) is enabled for 
SSH"
 fi
 else
 l_output=" - Cipher Block Chaining (CBC) is disabled"
 fi
 if [ -z "$l_output2" ]; then # Provide output from checks
 echo -e "\n- Audit Result:\n ** PASS **\n$l_output\n"
 else
 echo -e "\n- Audit Result:\n ** FAIL **\n - Reason(s) for audit 
failure:\n$l_output2\n"
 [ -n "$l_output" ] && echo -e "\n- Correctly set:\n$l_output\n"
 fi
}
Page 191
Remediation:
Note:
• The commands below are written for the included DEFAULT system-wide crypto 
policy. If another policy is in use and follows local site policy, replace DEFAULT
with the name of your system-wide crypto policy.
• Multiple subpolicies may be assigned to a policy as a colon separated list. e.g. 
DEFAULT:NO-SHA1:NO-SSHCBC
• Any subpolicy not included in the update-crypto-policies --set command will 
not be applied to the system wide crypto policy.
• Subpolicies must exist before they can be applied to the system wide crypto 
policy.
Create or edit a file in /etc/crypto-policies/policies/modules/ ending in .pmod and 
add or modify one of the the following lines:
cipher@SSH = -*-CBC # Disables the CBC cipher for SSH
-ORcipher = -*-CBC # Disables the CBC cipher
Example:
# echo -e "# This is a subpolicy to disable all CBC mode ciphers\n# for the 
SSH protocol (libssh and OpenSSH)\ncipher@SSH = -*-CBC" > /etc/cryptopolicies/policies/modules/NO-SSHCBC.pmod
Run the following command to update the system-wide cryptographic policy
# update-crypto-policies --set 
<CRYPTO_POLICY>:<CRYPTO_SUBPOLICY1>:<CRYPTO_SUBPOLICY2>:<SUBPOLICY3>
Example:
update-crypto-policies --set DEFAULT:NO-SHA1:NO-SSHCBC
Run the following command to reboot the system to make your cryptographic settings 
effective for already running services and applications:
# reboot

}
