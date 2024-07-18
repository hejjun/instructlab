Ensure SELinux is not disabled in bootloader configuration 
Profile Applicability:
• Level 1 - Server
• Level 1 - Workstation
Description:
Configure SELINUX to be enabled at boot time and verify that it has not been 
overwritten by the grub boot parameters.
Rationale:
SELinux must be enabled at boot time in your grub configuration to ensure that the 
controls it provides are not overridden.
Impact:
Files created while SELinux is disabled are not labeled at all. This behavior causes 
problems when changing to enforcing mode because files are labeled incorrectly or are 
not labeled at all. To prevent incorrectly labeled and unlabeled files from causing 
problems, file systems are automatically relabeled when changing from the disabled 
state to permissive or enforcing mode. This can be a long running process that should 
be accounted for as it may extend downtime during initial re-boot.
Audit:
Run the following command to verify that neither the selinux=0 or enforcing=0
parameters have been set:
# grubby --info=ALL | grep -Po '(selinux|enforcing)=0\b'
Nothing should be returned
Remediation
Remediation:
Run the following command to remove the selinux=0 and enforcing=0 parameters:
grubby --update-kernel ALL --remove-args "selinux=0 enforcing=0"
Run the following command to remove the selinux=0 and enforcing=0 parameters if 
they were created by the deprecated grub2-mkconfig command:
# grep -Prsq --
'\h*([^#\n\r]+\h+)?kernelopts=([^#\n\r]+\h+)?(selinux|enforcing)=0\b' 
/boot/grub2 /boot/efi && grub2-mkconfig -o "$(grep -Prl --
'\h*([^#\n\r]+\h+)?kernelopts=([^#\n\r]+\h+)?(selinux|enforcing)=0\b' 
/boot/grub2 /boot/efi)"
References:
1. NIST SP 800-53 Rev. 5: AC-3, MP-2
Additional Information:
This recommendation is designed around the grub 2 bootloader, if another bootloader is 
in use in your environment enact equivalent settings.
grubby is a command line tool for updating and displaying information about the 
configuration files for the grub2 and zipl boot loaders. It is primarily designed to be used 
from scripts which install new kernels and need to find information about the current 
boot environment.
• All bootloaders define the boot entries as individual configuration fragments that 
are stored by default in /boot/loader/entries. The format for the config files is 
specified at https://systemd.io/BOOT_LOADER_SPECIFICATION. The grubby 
tool is used to update and display the configuration defined in the 
BootLoaderSpec fragment files.
• There are a number of ways to specify the kernel used for --info, --removekernel, and --update-kernel. Specifying DEFAULT or ALL selects the de‐fault 
entry and all of the entries, respectively. Also, the title of a boot entry may be 
specified by using TITLE=title as the argument; all entries with that title are used