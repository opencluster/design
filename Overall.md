# Overall OpenCluster design.


**OpenCluster** is meant to be a suite of products that can be used individually or combined to provide a highly secure, scalable and maintainable environment.

## Trellis
Container and Service management interface and infrastructure.  This is the primary component that most people will call 'OpenCluster'.  It allows an environment to be described, and it will maintain that state.  It is functionally a combination of Kubernetes, Puppet and Monitoring.

[See More information about Trellis](Trellis.md)

## Locks
OpenCluster-Locks are a clustered external locking system for assisting applications that run on multiple instances to co-ordinate the work that they do.  For example, if your applications is designed so that it can only process a certain action one at a time, but has multiple instances of the application running (for HA, etc), then it one of those instances can first set a lock, do the action, and release the lock.  The other instances, when they try to set a lock are blocked if the lock is already set.

[See More information about Locks](OpenCluster-Locks.md)

----

Common features of all the products.

## Security

The entirety of the OpenCluster products need to take security at a very high level.  Security needs to be implemented in a way that doesn't 