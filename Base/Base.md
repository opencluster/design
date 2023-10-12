# OpenCluster-Base

Some of the OpenCluster services use very similar methods to ensure communication between nodes within the cluster.
The MessageQueue services are similar but provide different metrics.

The 'Base' is essentially a library and an optional code framework, that is targetted towards the Server components.  The counter-part for clients is [OpenCluster-Common](../Common/Common.md).

The [OpenCluster-Cache](../Cache/Cache.md), [OpenCluster-Data](../Data/Data.md) and [OpenCluster-Locks](../Locks/Locks.md) services will all use very similar methods in communicating.  What they do with the data communicated is all very different, but the communication method itself is similar.

## Basic Requirements:

* Security Authentication using certificates.
* Only one node can make changes at a time.
* Seperation and sync of communications between sites.

The base will handle the authentication part, with hooks that allow the overlying sytem to utlize the authentication and enhance it.

## Datacenter Connector.

Since datacenters may be regionally disperse, then the cluster in each datacenter should be fairly separate, but still in sync with the 
other cluster.

This means that the clients in one datacenter should not be connecting to the servers in the other datacenter, but the data they access 
should be in sync.

With the SimpleData one, not all communications needs to go to every node.  It is specific.  But changes need to be co-ordinated between 
sites.

