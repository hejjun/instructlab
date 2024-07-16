# 4.4.2.1 Ensure active authselect profile includes pam modules
A custom profile can be created by copying and customizing one of the default profiles. The default profiles include: sssd, winbind, or the nis. This profile can then be customized to follow site specific requirements.
You can select a profile for the authselect utility for a specific host. The profile will be applied to every user logging into the host.

## Rationale:
A custom profile is required to customize many of the pam options.
Modifications made to a default profile may be overwritten during an update.
When you deploy a profile, the profile is applied to every user logging into the given host

## Impact:
If local site customizations have been made to the authselect template or files in /etc/pam.d these custom entries should be added to the newly created custom profile before it's applied to the system. Please note that the order within the pam stacks is important when adding these entries. Specifically for the password stack, the use_authtok option is important, and should appear on all modules except for the first entry.

Example:

password requisite pam_pwquality.so local_users_only #<-- Top of password stack, doesn't include use_authtok

password required pam_pwhistory.so use_authtok #<-- subsequent entry in password stack, includes use_authtok

## Solution:
Perform the following to create a custom authselect profile, with the modules covered in this Benchmark correctly included in the custom profile template files
Run the following command to create a custom authselect profile:

`authselect create-profile`

Example:

`authselect create-profile custom-profile -b sssd`

Run the following command to select a custom authselect profile:

`authselect select custom/ {with-} {--force}`

Example:

`authselect select custom/custom-profile --backup=PAM_CONFIG_BACKUP --force`

## Note:

The PAM and authselect packages must be versions pam-1.3.1-25 and authselect-1.2.6-1 or newer

The example is based on a custom profile built (copied) from the the SSSD default authselect profile.

The example does not include the symlink option for the PAM or Metadata files. This is due to the fact that by linking the PAM files future updates to authselect may overwrite local site customizations to the custom profile

The --backup=PAM_CONFIG_BACKUP option will create a backup of the current config. The backup will be stored at /var/lib/authselect/backups/PAM_CONFIG_BACKUP

The --force option will force the overwrite of the existing files and automatically backup system files before writing any change unless the --nobackup option is set.

On a new system where authselect has not been configured. In this case, the --force option will force the selected authselect profile to be active and overwrite the existing files with files generated from the selected authselect profile's templates

On an existing system with a custom configuration. The --force option may be used, but ensure that you note the backup location included as your custom files will be overwritten. This will allow you to review the changes and add any necessary customizations to the template files for the authselect profile. After updating the templates, run the command authselect apply-changes to add these custom entries to the files in /etc/pam.d/

- IF - you receive an error ending with a message similar to:

[error] Refusing to activate profile unless those changes are removed or overwrite is requested.
Some unexpected changes to the configuration were detected. Use 'select' command instead.

This error is caused when the previous configuration was not created by authselect but by other tool or by manual changes and the --force option will be required to enable the authselect profile.

See Also: https://workbench.cisecurity.org/benchmarks/15286