# Deployment of non-secure files

When having a container running on a host, there may be a requirement that a particular file be on the host, and passed to the container.
The file (or folder) on the container should be listed as a resource, and the config should reference the resource, not the actual path, etc.
Dependencies of containers and resources would mean that if one or the other is changed, then the other can respond.
Dependencies can be automatically determined if resources are used in config rather than specific paths, etc.
An example of a file might be a security certificate to access a resource.

For folders, could move the folder to specific hosts that are needed.  read-only could be rsync'd between hosts.

Another example of a folder might be a data folder.  If we want to migrate a service to another host, we could handle it that way.  Even better would be to migrate the block device to the other node.

