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

# Running

When a host has been added to the cluster, recieved its [Role](Roles.md) information, and has everything running, it can continue to run, even reboot, and still perform it's services without the need of the central cluster.  All connectivity and access should remain, using DNS, or local host files. When the Host is setup, services are enabled that will ensure the components that should be running on the server, are running.  The cluster controller only needs to make changes when the environment itself changes.  For optimal environments, they should be setup in a way that doesn't require constant restructuring.  Hosts should ideally be pretty static.  Active/Passive hosts should be co-ordinated within the hosts and services themselves.

A central distributed data service itself can help co-ordinate this, without the need of the controller doing anything.


## Host Configuration

When a Host is first initialised several keys is stored on the server.  However, the [node](../Nodes/Nodes.md) config comes from the controller.   Some config can be marked as secure and volatile, which means that if the server is restarted, it must get the information from the controller.  These nodes cannot reboot and get all the information they need to continue working.  If the Host is compromised, there should not be any files on the system that have security keys.  

It may be more convenient to have the passwords on the server, so that when it restarts, it can automatically continue to function, but that reduces it security somewhat.  


## Host Reboots

When a host reboots, the system should provide an alert to the admins.  The Admin should then acknowledge the alert, which resets its config ready for the next time.  In these examples, the Host only requests its passwords once and keeps them in memory.  If a host requests the passwords again without first detecting a reboot, then alarms should go off.  If the server reboots multiple times, it should be disabled, until an admin acknowledges it.


## Questions.

1. What happens if a host is cloned and it joins the cluster?
* The cluster should identify that it has received connections from two hosts with the same Host ID, because when it connects they also have different IP's and MAC addresses and some other identifying components.  When new hosts are cloned, they should have certain things done to automatically generate a new Host ID.  Alternatively, they can connect without a Host ID and the system can give them one.