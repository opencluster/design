# Hosts

Hosts can be added to Roles.  The actual host themselves do not have to be exclusive or dedicated to Trellis, but obviously anything on there should not conflict with it.  In other words, it is perfectly acceptable to have trellis hosts that have additional things installed on them.  Quite possibly monitoring services, or management and maintenance tools, logging services, etc.

Hosts can be added to the system either manually, or automatic.  

## Automatic Host Provisioning
[Providers](Providers.md) can be utilized, and linked to metrics and other triggers to automically add capacity to an environment.


## Manual Host Provisioning
Hosts can come in all sizes and from different sources.  Also, some environments may be large and expensing, or small and inexpensive.  The trellis system shouldn't force people to use a much larger environment than they need, so it needs to be fairly flexible in the hosts that it can use.   Essentially, you can add a host to the system in a couple of convenient ways.

OpenCluster.org provides a service where config can be uploaded (automatically), and made available to new clients so that they know how to connect to the environment.

You login to the new host, and download a script from opencluster and run it.  You enter in a particular code, and it will pull down the appropriate config, and configure the host to have the minimum services on it that it needs.

The things it will do:
* Inventory and ensure minmum computing resources (CPU, OS Kernel, RAM, Storage, etc)
* Setup accounts and SSH authorized keys.
* Generate a random pool.
* Generate a unique Host Identifier.
* Ensure docker is installed, and of a suitable version.
* Setup the cluster services.
* Connect to the cluster.





# Questions.

1. What happens if a host is cloned and it joins the cluster?
* The cluster should identify that it has received connections from two hosts with the same Host ID, because when it connects they also have different IP's and MAC addresses and some other identifying components.  When new hosts are cloned, they should have certain things done to automatically generate a new Host ID.  Alternatively, they can connect without a Host ID and the system can give them one.