# Deployment of non-secure files

When having a container running on a host, there may be a requirement that a particular file be on the host, and passed to the container.
The file (or folder) on the container should be listed as a resource, and the config should reference the resource, not the actual path, etc.
Dependencies of containers and resources would mean that if one or the other is changed, then the other can respond.
Dependencies can be automatically determined if resources are used in config rather than specific paths, etc.
An example of a file might be a security certificate to access a resource.

For folders, could move the folder to specific hosts that are needed.  read-only could be rsync'd between hosts.

Another example of a folder might be a data folder.  If we want to migrate a service to another host, we could handle it that way.  Even better would be to migrate the block device to the other node.

NOTE: this is not referring to a clustered file service, this is for delivering files to a server, instance or container.


* **Static Files** - These are deployed by the controller, and the agent will periodically verify it and its content remains unchanged.  If the contents are different, it will restore the content (and other settings) and report it.
* **Initial Files** - When a service is first deployed to the server, the initial file is deployed.  However, once deployed it will no longer maintain it (although it can be configured to maintain the permissions and ownership, but not the content), or even not change anything at all.
* **Dynamic Files** - These are deployed the same as static files, but the source is not in the opencluster config itself, but a different service generates the file and it is deployed to the endpoints.  The source of the file can change the file, and depending on the config, can only really affect new services that startup, or it can trigger and transition where the new file is deployed and the services that require it restarted.  This would be described as either passive or assertive.  Passive means that when opencluster needs the file content, it will ask the service, and it will provide it.   Assertive means that the service will be active, and at its own determination decide to generate a new file and inform opencluster which will then trigger some functionality (which would likely result in the new file deployed, and the services restarted).

NOTE: Static files when deployed are a complete file, however the various components within OpenCluster config can be used to build the file.  In other words, it can be stored as a complete file... or some modules can use config to generate the file.


Another NOTE: In complex situations, an Initial File may be used, but when a significant change is being applied, it can **translate** a file using some specific script.   This would be unusual, but possible.  For example, if there is a service that used an *initial file* and has modified it, but now a replacement service is being applied that uses the same file, but some content needs to be changed.  In this case a specific script is initiated (pre-transition and post-transition) which can read the contents of the file, and modify it appropriately before starting the service.   The purpose of this is to fully automate every possible scenario.   This wouldn't normally apply to static files, but the pre-transition and post-transition scripts could still be useful in reading the contents of the static file (before and after modification) to do some other task to complete the process.
