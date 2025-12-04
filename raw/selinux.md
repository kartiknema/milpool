[200~SELinux
Linux / Unix systems traditionally provided only DAC (Discretionary Access Control).
DAC refers to access control based on user, group and others permissions.
Using DAC we can control what actions each entity can perform on a file, for example read from it, or execute it, or write to it, or a combination of these actions, or none of these actions.

SELinux provides MAC (Mandatory Access Control) which can be used for fine-grained access control. Note, MAC does not replace DAC, it works on top of it to provide an additional layer of security.

On a mandatory access control system (MAC), policies are defined which are administratively enforced, and they regulate access.
As mentioned before MAC works on top of DAC, so even if DAC rules allow some access, it is possible that MAC blocks that access.

The policies, known as SELinux policies can be extremely fine grained, and control an object’s access to:
Files
Processes
Ports
Sockets
Memory
Directories, etc.

A SELinux policy is essentially a set of rules, an example of a rule is one which allows a process to read files in /etc/infobin/custom, for example, without this rule the access will be prohibited. There are two types of policies broadly:
Targeted Policy (or Default Policy): Only targeted processes are protected by SELinux, everything else if unconfined. By targeted processes we refer to the processes which have associated SELinux policies.
Multi-Level Security Policy (or mls): More complex, and hence more powerful.

Some important commands and output:


bash$ cat /etc/selinux/config
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
# enforcing - SELinux security policy is enforced.
# permissive - SELinux prints warnings instead of enforcing.
# disabled - No SELinux policy is loaded.
SELINUX=permissive
# SELINUXTYPE= can take one of these two values:
# default - equivalent to the old strict and targeted policies
# mls     - Multi-Level Security (for military and educational use)
# src     - Custom policy built from source
SELINUXTYPE=default

# SETLOCALDEFS= Check local definition changes
SETLOCALDEFS=0
bash$ 

Note:
SELinux policy type (targeted or mls) is a system wide property.
Here, default indicates the policy type is targeted.
SELinux has 2 states: Enabled or Disabled. When it is enabled, SELinux supports 2 modes: Permissive and Enforcing
As can be seen in the adobe output, in the permissive mode, SELinux just logs warnings, if any access rule is branched, however it does not block the access. In the enforcing mode, the SELinux security policy is actually enforced, i.e. illegal accesses are blocked. 


sh-5.2# getenforce
Enforcing
sh-5.2#

Note:
Get SELinux operating mode.
SELinux mode is a system-wide property.


How SELinux works
SELinux uses two important concepts:
Labelling
Type Enforcement

Labelling: Each object or entity is labelled with a SELinux Context (also known as SELinux Label). Object could be: file, directory, process, port, socket etc.
For files and directories these labels are stored as extended attributes, in the file system itself.
For processes and ports, these labels are managed by the Kernel.

Format of the label:
user:role:type:level
To view a file or directory label, use the command:
Files: ls -Z <path_to_file>
Directories: ls -Zd <path_to_file>
As mentioned before for files and directories the label (or SELinux context) is stored as part of extended attributes , on the filesystem.

Checking some labels:

Executable file:
sh-5.2# ls -Z nft
system_u:object_r:iptables_exec_t:s0 nft
sh-5.2#

In the above label (SELinux context), system_u is the user, object_r is the role, the type is: iptables_exec_t, the level is 0.

To view process labels, use the “ps axZ” command:
system_u:system_r:syslogd_t:s0     1757 ?        Ssl   11:52 /usr/sbin/rsyslogd

To view directory label: use the “ls -Zd <>” command.

We have seen labels, above, the “type” field in the label, the next important concept used by SELinux is: Type Enforcement.

Type Enforcement: Take an example, a process running in the httpd_t context, probably has a good reason to interact with a file, labelled httpd_config_t.
On the contrary it does not make sense for a process running in the httpd_t context to interact with a file labelled with the shadow_t context (password related file).

Type Enforcement rules are part of the policy, for example, a type enforcement rule could state that “a process running in the httpd_t context, can read from a file labelled httpd_config_t”.

Note on the Label / Content
The context is set when the file is created, this context is based on the parent directory’s context, i.e. it is inherited.
We can use commands, like chcon or restorecon to change the context of a file.

Booleans: Represent on / off settings in SELinux, for example if the NTP server should be allowed to access the home directory.

