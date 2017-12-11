# OpenCluster-Locks

## Overview

The OpenCluster-Locks service allows different processes a method to perform functionality that requires other components do not interfere at the same time.    For example, you might have 4 instances of a service that performs account management, but you only want one instance making changes to an account at the same time.  This service allows that co-ordination.

The Locking service is used similar to Mutex's and thread-locking in single-process applications, but has been expanded to a cluster of applications.

It is important to note that the Locking service does not have any way to enforce the lock.  The applications accessing the resource must comply with the Locking agreements.

Important Warning.
Using Locks should be avoided where possible in the architecture.  Using a lock can cause problems during a Disaster-Recovery or High-Availability event.  If the system that is holding a lock has a failure, then recovery of that Lock can be consequential.   Systems should be designed where a lock is unnecessary.  Locks are typically only used for very sparse situations where it cannot be avoided.

An alternative to using locks is to use a Message Queing system that only delivers requests to an active node.

## Specific Feature.

### Authentication

Locks services should be authenticatable via multiple means.  The most common authentication protocol is LDAP.  This can be used to integrate authentication across multiple products.  The Locks service will specify specific groups that have particular roles and zones.  Each zone can therefore be treated seperately.

Authentication is done through username/password or certificates.

If a client connects to the locks server, but does not include a client certificate, it cannot know who the client is.   It will still accept the connect, and allow a certain number of operations to proceed.

#### Ethereal Authentication.

In some environments where full systems are disposable, scaled and built on demand, you want your locks authentication to be ethereal also.  The locks service can handle that by establishing temporary certificates that have a very finite lifetime.  In these cases, when the service is built, it is given a lifetime, and temporary accounts and certificates are built, and provided to the Locks clients before they attempt to connect.  This means that EVERYTHING on the built service has a finite lifetime (which may be only 24 hours as an example).  If the machine is snapshotted, it will have no data or accounts on it that can be used to connect to important services.

This kind of system involves a complete deplyment pipeline.

### Multi-Tenant Capable
The Locks system has the concept of walled-off zones.  Therefore multiple unrelated entities can use the Locks service without impacting or even being aware of each other.  This means that the Locks service can be sold as a service to multiple companies, or used by multiple seperate departments in the same company.

Each 'zone' can have its own set of certificates, authentication services, and logging outputs.   The only thing each 'zone' shares is the Locks servers themselves.  And the 'zones' do not have any visibility or management of the servers, other than being told which servers to connect to.

When a client connects, it indicates what zone it is wanting to use.   The certificate that it provided would be validated against that particular zone.  
Multiple zones might use the same certificate and authentication systems.

### How it clusters.

It is very similar to the OpenCluster-Data service, but only records the state of a lock.   Also, each server needs to be kept in sync with the Cluster.  Each server should have exactly the same dataset.   

When a lock is set, it is based on a name.   The name can be quite large, should be considered a string of characters, and should be human readable, however, it doesn't have to be.   The actual name is a binary string.    The 'name' is hashed to return a 64-bit integer.  The integer should have a low possibility of clashing (multiple names resulting in the same integer), however, it is possible, therefore the integer is used to provide a fast index, but is not required to be unique (although it is expected that it will still be mostly-unique).

The 64-bit hash (referred to as IDX), will be used to determine which server will be primarily responsible for it.   Depending on the number of servers in the cluster, a number of buckets will be allocated.  Each server in the cluster will take ownership of particular buckets.  It will continuously attempt to keep the bucket responsibilities  shared evenly amongst the cluster members.

Unlike the Opencluster-Data product, the locks service will ensure that each cluster member maintains a copy of the buckets that it is not responsible for.   However, each bucket will have a primary, and a secondary allocated.  This means that if a server stops responding, the entire cluster is aware of the node that is expected to take over.   

If mutliple nodes go down at the same time, then the cluster will go into recovery mode and responsibilities will be transferred.

Every time a lock is set, the request is directed to the cluster node that is responsible for that lock (based on the IDX hash).   When the lock is accepted, the primary node sends a message to all other nodes to tell them about the lock (so that they can keep their records up-to-date).

NOTE: When using LocationDomains, it can be configured that a copy of the lock data is kept within each LocationDomain, but the number of copies within each LocationDomain can be limited (to a specific number of copies).   This can help reduce the amount of network traffic, and resources required to maintain the multiple copies of the Lock data.


### How locks are set, released (and how does it prevent unauthorised release)

To set a lock, the caller simply provide a textual name for the lock, along with some optional parameters such as the expiry time of the lock, and whether the lock should survive a disconnect state (and how long it should allow for timeout).

When a lock is set, the Service responds with an unlock-key.  In order to unlock it, the client must provide this unlock-key.  The unlock key will be a random 64-bit number.

Locks can also have a time-limit.  This means that if the lock is not released after that time, then it will be released automatically.  Care should be taken that a released lock from a timeout does not cause problems for your application.

Locks must continue to work even if significant changes in the cluster occur (ie, a site goes offline).

The Locks servers will maintain a copy of shared config.  If the config is changed on any of the Locks servers, the change is replicated to all the others.   Each config change is tracked with a change ID, and as part of the heartbeat, each server indicates what config version they are using.   This will be used to ensure that split-brain situations are detected and handled.   This will also assist with synchronisation when a server goes offline, and when it comes back online, has invalid configuration.

Locks servers should also be able to be geographically isolated (through LocationDomains).  When a client connects to a Locks server, it is given a list of all the other Locks servers, and can choose the one that gives the quickest response.   This will become its primary Locks server.

When a client connects to the server, it establishes an identifier, and it will use that same identifier on the other Locks servers.  This allows them to move around to the fastest repsonding Locks server.

There are multiple kinds of Locks.

- Connection Lock : These locks are automatically released if the caller loses socket connectivity with the Locks server (although it is possible for the calling application to have connectivity to more than one Locks server, and can maintain a connection with any of the Locks servers to maintain the Lock.)

- Static Lock : These locks are used for situations where something connects, sets a lock, disconnects, and later re-connects and removes the lock.


## Discovery.

There are multiple ways that clients can find and continue connected to the Locks services.   A round-robin DNS entry could point to mulitple servers.  When the client gets connected to one server, it can request and receive connectivity details for other locks servers in the network.  The service can be configured to only provide this information if the client has connected with a valid client certificate, or it could be left open where any connected client can request the information.

This means that the client connects to one of the servers, and then can use the connection information returned for all the other servers to determine which one is the most appropriate one.  

To determine the appropriate server to connect to, the client can specify its LocationDomain and the server it connected to, will return a number of servers to connect to.

The client can also send a request and determine the response time, for each of the servers, to determine which one gives the quickest response.   It is therefore up the client to determine which locks server is the correct one to connect to.  

The clients can also request to receive notifications when other locks servers move around (become available, or are removed).

The service can also be configured to not provide any information about the servers to the clients.   In this case, the service might sit behind a load-balancer (or multiple load-balancers), or a Locks-Proxy, and the client will always connect to a specific connection.

## Location Domains

NOTE: Due to the complexity of Location Domains, and the limited use-cases for it, currently Location Domains will have limited support in the Locks system.  All nodes in the cluster will need to be active, and free to communicate with each other.   Location Domains will purely be used to assist the clients in connecting to servers that are most local to them, and to co-ordinate the number of copies of the LockData are kept within each LocationDomain.  There will be no effort to isolate or co-ordinate communications between nodes in different location domains.

Most OpenCluster products support [Location Domains](../LocationDomains.md), and the Locks product does as well.   

All the Locks servers in all locations are part of the same over-all cluster. And locks are kept in sync over the entire organisation, but efforts are in place to try and reduce the amount of traffic that occurs between those locations.

Depending on how you configure the services, you might restrict clients to only connect and use Locks services that exist in the same location domain.  For example, the clients in perth might be configured to use locks servers that exist in "au.perth".  This means they can connect to any of the locks servers in both 'au.perth.dh1' and 'au.perth.dh4'.  Those same clients wont try to connect to Locks servers in 'au.sydney'.

However, it should be noted that the default configuration will still allow for clients to connect to other locations if none exist in theirs.   It will bubble up.

If the client has a location domain for "au.perth.dh4.rack3.srv5", it will try to connect to clients in the following order.
* "au.perth.dh4.rack3.srv5"
* "au.perth.dh4.rack3.srv5.*"
* "au.perth.dh4.rack3"
* "au.perth.dh4.rack3.*"
* "au.perth.dh4"
* "au.perth.dh4.*"
* "au.perth"
* "au.perth.*"
* "au"
* "au.*"
* "*"


## Locks Performance Design.

If we have a situation where there are a large number of locks servers needed, then ensuring that all servers are in sync could mean locks take a while to set, plus every time a lock is set, a flurry of network activity would occur between all the nodes.   To ensure it is done as fast as possible, then you would typically use only a small number of locks servers.  I cant really think of a reason where you would need more than a few servers, but depending on your design, it may be more convenient to have a large number of servers (eg, you may prefer to have a locks node on each compute server, and direct requests to that node).  

If you have geo-located systems, then you would typically have one or more locks servers at each location.    You would certainly not need a pool of hundreds of locks servers.  Most organisations would only likely have 2 or 3.

OpenCluster Locks use a hash similar to the OpenCluster-Data service.  Data uses a hash of the key to determine which server is responsible for that hash (and its backups), and directs requests and changes there.  In that case, there is a specific master for an operation and therefore requests do not have be sync'd amongst the entire cluster. That master node (for that particular operation) is responsible. 

To ensure that the time it takes to set a lock is as predictable as possible, the actual requirement to set a lock is the time it takes for the master to process the request, to send the details to the secondary, and get confirmation from the secondary that it has received and accepted the request. 


## Cascading secondaries.

For every Bucket, there is a master node, and a cascading set of secondaries (depending on the config).

When a new node comes online, or the cluster-config changes regarding the number and method of secondaries, the nodes that are deficient will contact the master, and ask if they can be a standby for the bucket.  The master will respond with details of the node that it should cascade from, this will be the node at the end of the list.   The requesting node will then ask the node that the master indicated, if it can be a standby.  If that node does not already have a tail node, then it will accept.  If another node has snuck in, it will tell the standby to check that node... and so on, until it eventually finds one that isn't already linked.

When a node has accepted a secondary, it will begin pushing all its current data to the secondary.  It will keep sending data to its secondary until all the data is complete.  So that a node does not need to keep track of every change since the beginning of time, it will do some shortcuts when soing the initial sync.  Once a node is sync'd, it needs to confirm that each operation is being performed correctly.

While it is syncing its secondary, the node will keep track of new changes to the bucket that occur since, and it will queue those changes until the starting-sync has completed.  Once it is completed, it will tell the node what the current hash is.   The secondary will then compare the next change against that hash, which will allow it to know that it is in sync.

When something changes in a master bucket, it sends a message of the change to the designated secondary.  The change includes a chained hash, which allows the secondary to verify that the information it has is the same as the master has at the end of the transaction.  It is up to the secondary to ensure that it has a valid copy, so any discrepancies will result in a recovery situation.   This cascades through the secondaries.

When a change is received by a secondary, it marks that notation in its memory structures, and when it gets confirmation that its secondary has processed it, it will cascade up.  This also means that while a new node is syncing a bucket, and new changes to that bucket will not be confirmed until that sync is complete.

Since updates to the locks are cascaded, it means it is slow to send the info to the entire cluster.  This is not a problem because the master node is the most important, and it will always be updated first.  It is the one that sets and checks locks.  The cascading sync is only for redundancy, and the further down the tree, the less important it is.   It should also be noted that when setting a lock, you dont need the lock to be cascaded to all nodes for them to take effect.   It should wait until the first secondary has responded that it has received the lock data, and then tell the client that it can proceed.

If a node detects a discrepancy in its hash of the latest change, it will report the problem, then recover.  To recover, they essentially dump their copy, ask to be dropped as a secondary.   Node then begins the process of asking to be a standby for the bucket, which will be directed to the last node in the list.  It will then do the normal process as if it was a new node.   When nodes go out of sync, it should be investigated.  It could be from a myriad of unusual problems.  Including memory corruption, lost packets, bugs, etc.  It is a serious situation.

If a master node that is co-ordinating the set of a Lock goes offline, then a number of things for that lock need to be checked by the secondary when it becomes the master.  Since there is a master node for each lock bucket, if that node goes offline, the secondary server will need to take over.  Depending on what it knows about the lock, is how it would respond.  Basically the master server will have handled the lock/unlock operation.  The rest of the cluster would return information about the status of that operation.   The master server will then redirect taht information to the secondaries, etc.   This means that the secondary should know what state the lock sync was in, and can re-send any lock details to the other servers, that it hasn't recieved full confirmation on.    Every bucket has a cascading list of secondaries... so the updates should flow from one to the other.   A secondary should have no problem whatsoever if it has to take over for a missing primary.

When anything happens on any node that is a concern, then that alert should be sent to all nodes.    The nodes themselves might not do anything about it, but can report it to monitoring solutions.

## Preferred Secondary.

LocationDomains are used to logically determine where the server nodes are.  It does not have to be geographically based, but that generally makes the most sense.  When the nodes are determining which should be the secondary of the master, there is logic that needs to be determined.  This depends on your actual use case.  This can be done using a mask, or specific LUA code that makes the determination.

If a mask is used (Note, LUA code is used to process the mask).   It gives preference to a node that fits within the mask, but does not much the locationDomain of the master node.

If the master node has a location domain of 'au.perth.dc4.rack3.srv25', and a secondary already exists that has 'au.perth.dc4.rack2.srv08', and another node comes online with a location domain of 'au.sydney.dc1.rack2.srv85'.   The new one will do the normal process of becoming a backup bucket node.  Once the sync is complete, it will have a complete picture of the master and secondaries for each bucket.   At the send of the sync, the master will be told by the new node, and the node it is cascaded from, this it is a new backup bucket.   The master will then determine if the new node is better than the current secondary.  If the master thinks it is a better match, it will promote the new node to be the first secondary.

By default the logic will return a number that is the number of location domains that are different.  The higher the level that is different, the higher the number.  In this case, the existing secondary will return 2.  The new secondary will return 4, which is higher.     

NOTE that the master will always be the one that determines if a secondary should be promoted or not.  It is not up to the secondaries to ask to be promoted, otherwise we will have lots of secondaries constantly asking to be promoted.  The master will only promote a secondary if it has a higher score.  Using the mask method, anything that does



## Metrics

As each cascading node pushes the change to its standby, it will wait until it gets a sync responce back, before sending a responce itself.  This means that when the master sends an update to its secondary, it wont get a completion result until all the cascading nodes have received the sync.  It can store that metric and report it, so that the governing system have some metrics on how quickly changes get synced down the entire chain.

It can get metrics on:
* time to sync an update
* number of updates performed per second
* time it takes for master to process an update.
* number of locks (per bucket, and total) that are active per second.
* average time of locks lifetime before it is unlocked.
* average ping time between nodes, from each node.
* number of bytes received, number of bytes sent
* number of clients connected to a node.


## Quorum

In any clustered solution, a big concern is a 'split-brain' scenario.  Essentially this occurs if you have a situation where the clients can talk to the nodes, but something has happened that is stopping the nodes from talking to each other.  In this case, the nodes assume that the other nodes have simply stopped working.  You then end up with two independantly working clusters.   When the problem is resolved, and those nodes are able to talk to each other again, it will be difficult to work out which state is correct, because both have been servicing clients.

This is especially a problem in a locking service.  As the point of the locking service is to ensure that certain things do not happen by multiple clients at the same time.  But in this 'split-brain' scenario, that is exactly what is happening.

To stop this from happening, you want to ensure that if either site goes offline, you are able to determine which is the appropriate one that keeps running.   To do this, you should have an odd number of sites (but this doesn't help if most of the sites go offline at the same time).

Another method is to have (one or more) arbiter nodes in other sites.  Especially sites that you are not using yourself.  Eg, if you have two physical data-center locations, you might have the arbiter nodes in one or more Amazon zones, and maybe other 'cloud' environments as well.



## OpenCluster Arbiter.

For free (or a small yearly fee), OpenCluster services could provide an arbiter node.  This node contains no data, but instead is used to verify connectivity between nodes.  It is used as a 'third-site' for quorum purposes.   Since it will live on the internet as an external resource, it does not need to be on-premise.   Some people may want to run their own arbiter node in the cloud somewhere.   We might therefore have a number of arbiter nodes in various locations.  

The arbiter nodes themselves will not contain data, and clients will not connect to them (clients wont even know they exist).

The requirement for the arbiter nodes, is that they can receive connections from all members of the cluster, although it should be noted that the arbiter nodes will not attempt to establish connections.   A single arbiter node could service many many different clusters.  


	
## Handling non-quorum clusters.

Since a quorum typically requires the ability to obtain a majority, it is difficult to do when there are less than 3 server nodes.   So in clusters that have less than 3 nodes, it will behave slightly different.   It will essentially go into active/passive mode.     Clients will be informed that the cluster is in active/passive mode, and that they should connect to both nodes, but send all requests to one (although they dont have to).  if the 2 nodes are able to talk to each other, then they will remain in the same state.  Clients that are able to talk to the passive, but not to the active one, will be able to send requests to the passive.  Clients will also inform the server that it is unable to communicate with the active.  If the passive is unable to talk to the active, and the clients are saying they cannot talk to the active, then the passive will become the active node.   The clients will be informed that the passive node has now become active.   And if any clients (who are connected to both), are still able to talk to the active, they will be told that the other server is now active, and they will inform the previously-active server that the now-active server has taken over the role.  The now-passive server will be in a 'fault' status, and when it manages to connect to the other node, it will have to go in catch-up mode.  When everything is in sync, the preferred active (that is currently passive, and faulted) will try and promote itself back to normal running state.

  
  
## Integrity Checks.

Each lock will keep track of their age, and certain other settings.   Each locks server that has locks, will need to validate against the other servers that they all have the same details.

In addition, it should keep track of the number of locks each server has, and during a moment of stability, should provide stats to the other servers to indicate how many locks they each have.  If the servers have different lock counts, then something is wrong.  The hard part here, is doing that while the service is heavily in use, as many servers might be legitimately out of sync as new locks are propogating.

The Buckets themselves need to verified amongst all the secondaries.   As each lock is updated, it will update a rolling hash.  The master node will periodically sort all its keys for that bucket in binary order, and generate a quick hash on that data.  It will then send that check to all the secondaries.  If the secondaries do not match, then they will need to do a more deep verification.


## Starting a New Locks Cluster.

When the services start, and there is some config that is stored on disk, then it should know how to connect to the cluster, or start a new one.  This config may have a DNS entry, and the locks server can try all the IP's that are returned for that DNS entry, and if it cant connect to any of them (and it's IP appears to be one of them in the list), then it can assume that it is the first member of the cluster.

The config should also indicate the minimum quorum (not just a percentage).  This means that if the config indicates that there should be a minimum of 3 locks servers running, then it will not open up client connectivity until that many servers are talking with each other.

If there is no config however.  The server is being started for the first time.  Then it will need to be specifically indicated that it is the first node being created in the cluster.

A command-line tool will be provided that establishes everything needed for a cluster to start.   All the parameters could be supplied on the command-line.  Anything needed that is not supplied, will be asked.


## Tools and Binaries

OpenCluster Locks will include some command-line tools to set local config, and to interact with the locks themselves.  The configuration tools will be installed with the daemon, but the tools to interact with locks themselves will likely be in a different package that can be installed anywhere.

### Opencluster Locks Daemon

`oclocks-daemon`

This is the daemon service that will communicate with other daemons to co-ordinate the setting and clearing of locks.

### OpenCluster Locks Cluster Startup Tool

`oclocks-startup`

When normal Locks services startup, they try to connect to a cluster that is already running.  Special steps are required to start a new cluster.
This tool can be either interactive or through command-line parameters.

### OpenCluster Locks Command Line Tool

`oclocks-cli`

This command-line tool is used to interact with a running locks service.  It allows you to connect to the cluster, and depending on authentication levels and rights, view details of the zones and locks within the system.

### OpenCluster Locks config tool

`oclocks-config`

Any clustered product can be complicated to configure and test connectivity.  This command-line tool will assist with that.   It will allow for servers to be setup manually, and even through automatic solutions.

Note that the main config file, should be considered to be the same for the entire cluster.   It should even be possible to make a change and have that change pushed to the config for every node in the cluster.  However, it should also be possible to lock down the main config so that remote changes to the config are not possible.  This would be the case where the main config is controlled by some other deployment process.

When making config changes, most of them would actually be within other suplimental config files that are unique for that node.  For example, authentication.  The main config file will point to another file that contains all the config.

It should also be noted that other standard environemental factors can allow for a completely config-free locks service.  If there is no config, it has defaults for everything, including the address to use to connect to the other locks servers.  This will only work if the environment has been setup specifically to allow this though.

```
oclocks-config set auth type certificate
oclocks-config set auth certfile /opt/opencluster/locks/config/certs/certfile
oclocks-config set auth keyfile /opt/opencluster/locks/config/certs/certfile
oclocks-config verify
```

## Requirements

1. When the service starts, it loads the config that tells it what other locks servers to connect to (this can be either a list of IP's of actual locks servers, or it can be the address of a load-balancer).
1. The config file will point to an identifier file.  The unique server identifier is not kept in the config file, because it is possible, the config file would be desired to be kept the same on all systems.  Therefore, the configfile should point to other files that contain unique information for the server.   As much actual config as possible, should be kept in the clustered service itself.
1. If the startup option indicates that it is known that this server is the first node in the cluster, it will start the cluster.
1. If not specifically indicated that this is the first instance of the Locks service, The service connects to at least one other locks server that is part of the cluster.
1. Server can be configured to not provide Locks server details to clients.   
1. If the client connects without a client certificate, only a subset of operations are available until the client authenticates.  They can authenticate with a username and password.  The client can also generate a CSR and get a certificate signed, which it can then use to connect in the future.


## Config File (for Server) requirements

The config file should contain information that can be identical on all locks servers (which means that it should be possible to deploy the exact same config file to all locks servers).   The locks service does need to store data that is unique to it, so the config file should point to other files that contain this data.

Note that the main config files does not have to be the same on all boxes, but it is desired.

### Identifier

When a locks service first starts up, if the file that 'identifier' is referring to doesn't exist, it will create a random number, and use that number to identify itself. 
Config inside the Locks system can add friendly names to those identifiers.
Location domains are applied to the identified systems, but again, those details are stored in the cluster itself, not in local config on the box.

No two servers should exist in the cluster with the same identifier.  The identifier file should NOT be deployed to systems unless it can be guaranteed to be unique.

### Authentication

The config file will point to an authentication file.  The authentication file will be a JSON structure which can be built using the command-line tools.  The JSON file does not need to be modified manually, but can be.

An example:

```
oclocks-config set auth type certificate
oclocks-config set auth certfile /opt/opencluster/locks/config/certs/certfile
oclocks-config set auth keyfile /opt/opencluster/locks/config/certs/certfile
oclocks-config verify
```

If the main config does not specify an auth file, the tool will use the default location `/opt/opencluster/locks/config/auth.json`, but will not modify the main config file.



## Protocol

* [Locks Server Protocol](LocksServerProtocol.md)
* [Locks Client Protocol](LocksClientProtocol.md)

