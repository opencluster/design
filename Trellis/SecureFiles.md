# Secure files

When a host is started up, it should be fairly clean.  There should not be any critical security files on it.
When the host connects to the system and uses its identity to obtain the node information set.  This information set will include all the keys and secure files (that are encrypted and can only be parsed with the private key).
The secure files can then be put in the filesystem, preferably as piped files, that a service will respond with.  Those decrypted versions of the file will remain in memory only, and all effort should be taken to ensure that it is not be written to disk.
