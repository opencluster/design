# Secure files

When a host is started up, it should be fairly clean.  There should not be any critical security files on it.
When the host connects to the system and uses its identity to obtain the node information set.  This information set will include all the keys and secure files (that are encrypted and can only be parsed with the private key).
The secure files can then be put in the filesystem, preferably as piped files, that a service will respond with.  Those decrypted versions of the file will remain in memory only, and all effort should be taken to ensure that it is not be written to disk.

Note that it could be implemented in a way that the decryption key is not stored on the actual server itself.  When the server is booted, it has its credentials stored, but has no config stored.  It connects to the config manager, and if its credentials are still valid, then it will get the config and the appropriate keys (combined with the credentials) to decrypt it.

One typical way of doing this is to setup a RAM based virtual file system (TMPFS).  This sets up a file system which is actually stored in RAM.  It can store sensitive files there, and if the server is rebooted, the ram is emptied (although be aware that VirtualMachines can be snapshotted and so the file contents could be obtained... nothing is actually completely secure)

This is to ensure that if a copy of the server is obtained, the amount of critical data on it is limited.

There are advantages and disadvantages to this approach:

Advantages:

* Added security

Disadvantages:

* Server will require connectivity to the Controller to start up
* Services on the node cannot start until the config is obtained and decrypted
* Snapshot and restore of virtual machines might not work if the node data changes regularly (this feature can be enabled or disabled).


## The source of the secure file

There are multiple layers to this functionality.   The OpenCluster core functionality will need to update files on the end-node so that its services will know how to connect to opencluster services.  But the general cluster config can also generate secure files.  This can be done by either storing a secure file, or having a service that can generate the secure file.  This is very flexible.

For example, if we have a custom made service that just generates a unique key for each live server, and that key is then used when that server accesses other services.
