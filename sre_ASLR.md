# Ensure address space layout randomization (ASLR) is enabled"

Address space layout randomization (ASLR) is an exploit mitigation technique which randomly arranges the address space of key data areas of a process.

## Rationale:

Randomly placing virtual memory regions will make it difficult to write memory page exploits as the memory placement will be consistently shifting.

## Solution:
Set the following parameter in /etc/sysctl.conf or a file in /etc/sysctl.d/ ending in .conf:
kernel.randomize_va_space = 2
Example:

`printf 'kernel.randomize_va_space = 2' >> /etc/sysctl.d/60-kernel_sysctl.conf`

Run the following command to set the active kernel parameter:

`sysctl -w kernel.randomize_va_space=2`

Note: If these settings appear in a canonically later file, or later in the same file, these settings will be overwritten
Default Value:
kernel.randomize_va_space = 2
See Also: https://workbench.cisecurity.org/benchmarks/15286