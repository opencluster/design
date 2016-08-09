# Containers

Normally Docker containers that are associated with a Role.
Service details can be imported (the imported config may indicate a preferred role with a generic name, or a specific role with  a specified ID).  The implementor might overrule some of the security considerations, but by default, imported config that should be automatically associated with particular roles, should be signed by a certificate that has been approved to add those services to a particular role.
Note that Role's should not specify environments.   You should not have a 'Staging' and 'Production' role.  Those are environments.   Roles should be logically considered to be within the scope of environments but identical.
