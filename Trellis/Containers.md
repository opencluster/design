# Containers

Normally Docker containers that are associated with a Role.
Service details can be imported (the imported config may indicate a preferred role with a generic name, or a specific role with  a specified ID).  The implementor might overrule some of the security considerations, but by default, imported config that should be automatically associated with particular roles, should be signed by a certificate that has been approved to add those services to a particular role.
Note that Role's should not specify environments.   You should not have a 'Staging' and 'Production' role.  Those are environments.   Roles should be logically considered to be within the scope of environments but identical.

## Container Config

The container config will be based on a core config, and then multiple versions that remove or add some of that config.  It will have some defaults for when it is integrated into a Role (active/passive, ports, storage requirments, etc)



## Updating Containers.

In the trellis config, Each container would be listed as a resource, and it would include information about all the different versions of that container, and will provide the instructions on how to migrate from older versions to new.

If you have a container called "MainWebsite", and you have multiple versions of it, there would be some config for each version.  

When a container is referenced in a [Role](Roles.md), it indicates which version should be used.  When the change is submitted, the [Controller](Controller.md) will see that the Running Info and the Config do not match.  It will see that MainWebsite container should be running at a particular version.  It will see that older versions of that Container are running, so will follow the instructions on updating from the older version.  The instructions would reference a script that runs.

