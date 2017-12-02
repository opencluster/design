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

In some environments where full systems are disposable, scaled and built on demand, you want your locks authentication to be ethereal also.  The locks service can handle that by establishing temporary certificates that have a very finite lifetime.  In these cases, when the service is built, it is given a lifetime, and temporary accounts and certificates are built, and provided to the Locks clients before they attempt to connect.  This means that EVERYTHING on the built service has a finite lifetime (which may be only 24 hours).  If the machine is snapshotted, it will have no data or accounts on it that can be used to connect to important services.

This kind of system involves a complete deplyment pipeline.

### Multi-Tenant Capable
The Locks system has the concept of walled-off zones.  Therefore multiple unrelated entities can use the Locks service without impacting or even being aware of each other.  This means that the Locks service can be sold as a service to multiple companies, or used by multiple seperate departments in the same company.

Each 'zone' can have its own set of certificates, authentication services, and logging outputs.   The only thing each 'zone' shares is the Locks servers themselves.  And the 'zones' do not have any visibility or management of the servers, other than being told which servers to connect to.

When a client connects, it indicates what zone it is wanting to use.   The certificate that it provided would be validated against that particular zone.  
Multiple zones might use the same certificate and authentication systems.

### How it clusters.

It is very similar to the OpenCluster-Data service, but only records the state of a lock.   Also, each server needs to be kept in sync with the Cluster.  Each server should have exactly the same dataset.

### How locks are set, released (and how does it prevent unauthorised release)

To set a lock, the caller simply provide a textual name for the lock, along with some optional parameters such as the expiry time of the lock, and whether the lock should survive a disconnect state (and how long it should allow for timeout).

When a lock is set, the Service responds with an unlock-key.  In order to unlock it, the client must provide this unlock-key.  The unlock key will be a random 64-bit number.

Locks can also have a time-limit.  This means that if the lock is not released after that time, then it will be released automatically.  Care should be taken that a released lock from a timeout does not cause problems for your application.

Locks must continue to work even if significant changes in the cluster occur (ie, a site goes offline).

If Configured so, Locks servers must maintain a Quorum.  This requires that an odd number of Locks servers exist (some servers can be setup to only be a quorum member and not actually handle client connections).

The Locks servers will maintain a copy of shared config.  If the config is changed on any of the Locks servers, the change is replicated to all the others.   Each config change is tracked with a change ID, and as part of the heartbeat, each server indicates what config version they are using.   This will be used to ensure that split-brain situations are detected and handled.   This will also assist with synchronisation when a server goes offline, and when it comes back online, has invalid configuration.

Locks servers should also be able to be geographically isolated.  When a client connects to a Locks server, it is given a list of all the other Locks servers, and can choose the one that gives the quickest response.   This will become its primary Locks server.

When a client connects to the server, it establishes an identifier, and it will use that same identifier on the other Locks servers.  This allows them to move around to the fastest repsonding Locks server.

There are multiple kinds of Locks.

- Connection Lock : These locks are automatically released if the caller loses socket connectivity with the Locks server (although it is possible for the calling application to have connectivity to more than one Locks server, and can maintain a connection with any of the Locks servers to maintain the Lock.)

- Static Lock : These locks are used for situations where something connects, sets a lock, disconnects, and later re-connects and removes the lock.


## Discovery.

There are multiple ways that clients can find and continue connected to the Locks services.   A round-robin DNS entry could point to mulitple servers.  When the client gets connected to one server, it can request and receive connectivity details for other locks servers in the network.  The service can be configured to only provide this information if the client has connected with a valid client certificate, or it could be left open where any connected client can request the information.

This means that the client connects to one of the servers, and then can use the connection information returned for all the other servers to determine which one is the most appropriate one.  The client can send a request and determine the response time, for each of the servers, to determine which one gives the quickest response.   It is therefore up the client to determine which locks server is the correct one to connect to.  

The clients can also request to receive notifications when other locks servers move around (become available, or are removed).

The service can also be configured to not provide any information about the servers to the clients.   In this case, the service might sit behind a load-balancer (or multiple load-balancers), and the client will always connect to a specific connection.

## Location Domains

NOTE: Due to the complexity of Location Domains, and the limited use-cases for it, currently Location Domains will have limited supported in the Locks system.  All nodes in the cluster will need to be active, and free to communicate with each other.   Location Domains will purely be used to assist the clients in connecting to servers that are most local to them.  There will be no effort to isolate or co-ordinate communications between nodes in different location domains.

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

If we have a situation where there are a large number of locks servers needed, then ensuring that all servers are in sync could mean locks take a while to set, plus every time a lock is set, a flurry of network activity would occur between all the nodes.   To ensure it is done as fast as possible, then you would typically use only a small number of locks servers.  I cant really think of a reason where you would have more than a few servers, but depending on your design, it may be more convenient to have a large number of servers (eg, you may prefer to have a locks node on each compute server, and direct requests to that node).  

If you have geo-located systems, then you would typically have one or more locks servers at each location.    You would certainly not need a pool of hundreds of locks servers.  Most organisations would only likely have 2 or 3.

If, however, the time it takes for locks to sync is a problem, an alternative design is to use a hash similar to the OpenCluster-Data service.  Data uses a hash of the key to determine which server is responsible for that hash (and its backups), and directs requests and changes there.  In that case, there is a specific master for an operation and therefore requests do not have be sync'd amongst the entire cluster. That master node (for that particular operation) is responsible. 


## OpenCluster Arbiter.

For free (or a small yearly fee), OpenCluster services could provide an arbiter node.  This node contains no data, but instead is used to verify connectivity between nodes.  It is used as a 'third-site' for quorum purposes.   Since it will live on the internet as an external resource, it does not need to be on-premise.   Some people may want to run their own arbiter node in the cloud somewhere.   We might therefore have a number of arbiter nodes in various locations.  

The arbiter nodes themselves will not contain data, and clients will not connect to them (clients wont even know they exist).

The requirement for the arbiter nodes, is that they can receive connections from all members of the cluster, although it should be noted that the arbiter nodes will not attempt to establish connections.   A single arbiter node could service many many different clusters.  


## Quorum

In any clustered solution, a big concern is a 'split-brain' scenario.  Essentially this occurs if you have a situation where the clients can talk to the nodes, but something has happened that is stopping the nodes from talking to each other.  In this case, the nodes assume that the other nodes have simply stopped working.  You then end up with two independantly working clusters.   When the problem is resolved, and those nodes are able to talk to each other again, it will be difficult to work out which state is correct, because both have been servicing clients.   

This is especially a problem in a locking service.  As the point of the locking service is to ensure that certain things do not happen by multiple clients at the same time.  But in this 'split-brain' scenario, that is exactly what is happening.

To stop this from happening, you want to ensure that if either site goes offline, you are able to determine which is the appropriate one that keeps running.   To do this, you should have an odd number of sites (but this doesn't help if most of the sites go offline at the same time).

Another method is to have (one or more) arbiter nodes in other sites.  Especially sites that you are not using yourself.  Eg, if you have two physical data-center locations, you might have the arbiter nodes in one or more Amazon zones, and maybe other 'cloud' environments as well.

*****

When a lock is requested by a client, the server receiving the request will check the lock locally.  If the lock is currently unset locally, then it will assume (for efficiency) that the rest of the cluster also does not have the lock set.  It will send a message to all other servers adding the lock to the queues of each server.  Since it is the first one in the queue, the other nodes will indicate that they have accepted the lock, and are provissionally ready to activate it.

When a lock is requested, the server will send the lock request to all other nodes.  

If the same lock is being requested by more than one server, then it becomes a race to the one which has the majority.

Question:
	What happens if the lock requests are being driven by more than TWO servers, and none of the requests are able to reach a majority?   
Answer:
	Once all the servers have responded, if the majority is not met, then the originating servers will compare their 64-bit server ID against the serverID of the other node that is in contention.   To do this comparison, the server whose number is less than the other will be declared the winner.   The other node will revert their request, and therefore a winner will result.  Even if this happens out of order (ie, server A and B compare, and server C and B compare), once one starts reverting their locks, the deadlock will clear.
	
	NOTE, the servers know they are in a deadlock, because they have received all the responces from the other servers. 

Question:
	Is there an advantage in not making the lock active on the servers after the majority have accepted?
Answer:
	The majority is really only needed before returning the positive result to the client.   It doesn't really matter if the server has activated a lock pre-maturely.  Actually, it would be more work to re-concile an accepted but inactive lock.

Question:
	What happens if an originating node that is co-ordinating the set of a Lock goes offline in the middle of it?   The other nodes will need to cancel the request, or continue on with it if the client is also connected to other nodes.
Answer:
	There is no magic answer here that I have found so far.   It could be that during the lock process, since that node was the one making the Lock, if it fails, the other nodes may be able to tell that it has failed.  The same will happen when unlocking a lock.   Can we do it in a way that one server is not the one co-ordinating the lock?  

Question:
	When a node has accepted a lock, it should send a message to all other nodes indicating that it has accepted the lock?  That way ALL the nodes are aware of the quorum being met.  When setting a lock, maybe there needs to be a timeout?  When the originating node has sent a message back to the client that it has completed the lock, maybe it should send another message back to the cluster to indicate such.   That way, when a server fails mid-lock, and the client is not informed, but the client is also connected to another locks server, the other locks server can take it upon itself to continue the locking process, and inform the client.   Alternatively we can leave it to the client to follow-up if it doesn't get a responce, or if it loses heartbeat with a node.   What if a client is only connected to one server?  What then?   If it loses connetivity with the node mid-lock, and the server cannot return a valid response?   Then the lock needs to be removed.  Which server initiates the removal?  What would be the ramifications if multiple servers initiate the removal.
  
Question:
	What happens if a lock is set, unlocked quickly, and another node sets teh same lock right after.  This means that the sync process will need to be able to catch up.  System should be able to handle very rapid locking and unlocking.
  
Question:
	When a lock is set, and other clients are waiting for the lock, then they essentially queue up.  Should we sync the queue amongst all the nodes? or essentially the server that initially received the request waits for the lock to be unset, and then a free-for-all happens to be the next in the queue?   It should be predictable which clients receive the lock.  The only way to do that, is to sync the request queue.  Maybe the locking process, and the queuing can be in the same operation.   The various syncing modes will play a part in how effective either method is.  If you have a system where you have a large number of nodes, as well as a rapid lock/unlock process, then you want the sync and the queue to be quick.  The quorum method will also be better in situations where some nodes are fast, but others are either slower or have significantly higher latency and take time to catch up.  Ensuring that all nodes are in sync before granting the lock means that the slowest node to respond is the best you will get.

	
## Handling non-quorum clusters.

Since a quorum typically requires the ability to obtain a majority, it is difficult to do when there are less than 3 server nodes.   So in clusters that have less than 3 nodes, it will behave slightly different.   It will essentially go into active/passive mode.     Clients will be informed that the cluster is in active/passive mode, and that they should connect to both nodes, but send all requests to one (although they dont have to).  if the 2 nodes are able to talk to each other, then they will remain in the same state.  Clients taht are able to talk to the passive, but not to the active one, will be able to send requests to the passive.  Clients will also inform the server that it is unable to communicate with the active.  If the passive is unable to talk to the active, and the clients are saying they cannot talk to the active, then the passive will become the active node.   The clients will be informed that the passive node has now become active.   And if any clients (who are connected to both), are still able to talk to the active, they will be told that the other server is now active, and they will inform the previously-active server that the now-active server has taken over the role.  The now-passive server will be in a 'fault' status, and when it manages to connect to the other node, it will have to go in catch-up mode.  When everything is in sync, the preferred active (that is currently passive, and faulted) will try and promote itself back to normal running state.

  
  
## Integrity Checks.

Each lock will keep track of their age, and certain other settings.   Each locks server that has locks, will need to validate against the other servers that they all have the same details.

In addition, it should keep track of the number of locks each server has, and during a moment of stability, should provide stats to the other servers to indicate how many locks they each have.  If the servers have different lock counts, then something is wrong.  The hard part here, is doing that while the service is heavily in use, as many servers might be legitimately out of sync as new locks are propogating.

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



### Client to Server Commands
```
// 1 byte integer - 0x0000 to 0x0fff         0 000 xxxx xxxx
// 2 byte integer - 0x1000 to 0x1fff         0 001 xxxx xxxx

0x1000	CAPABILITY <command-id>
		This command asks the service if it supports a specific command.  It's parameter is a 16-bit command.   Can be used by both server and client.
		Responds with:
			OK_COMMAND <command-id>
			INVALID_COMMAND <command-id>
			
0x1001	OK_COMMAND <command-id>
		When a client or server wants to know if particular commands are supported, it will ask 'CAPABILITY <command-id>'.  If the command is supported by the server, it will respond with this OK_COMMAND.   
		
0x1002	INVALID_COMMAND <command-id>
		If an invalid command is received by the client or servver, it will respond with this command.  Both client and server need to ensure that services are handled correctly if this result is received.  It typically means that either protocol is out-of-date.   To verify the capability of the opposite connection to be able to handle the commands expected prior to actually using them, see the CAPABILITY command.


// 4 byte integer - 0x2000 to 0x2fff         0 010 xxxx xxxx

0x2000	EXPIRE_SECONDS <seconds>
		Used when a lock is being set an expiry can be set.  This is a failsafe, and is recommended not to use.

// 8 byte integer - 0x3000 to 0x3fff         0 011 xxxx xxxx

0x3000	UNLOCK_CODE <code>
		This is a random 64-bit number that is generated by the server and provided to the client when a Lock is set.   In order to unlock the lock, the same Unlock-Code must be supplied.
		
0x3000	LOCK_WAIT_TIMEOUT <seconds>
		When LOCK_WAIT is used, it will wait for 'seconds' before returning a failure.   This feature can be used in situations where a single-threaded client needs to try for the lock, but if it cant get it within a certain time, needs to provide some other functionality before trying again.   This value will remain for the session, not just the single attempt.  Setting 'seconds' to zero means that it will wait forever.

// 16 byte integer - 0x4000 to 0x4fff        0 100 xxxx xxxx
// 32 byte integer - 0x5000 to 0x5fff        0 101 xxxx xxxx
// 64 byte integer - 0x6000 to 0x6fff        0 110 xxxx xxxx
// No Parameters - 0x7000 to 0x7fff          0 111 xxxx xxxx

0x7000	SET_LOCK
		Will use the NAME and EXPIRE_SECONDS values to determine what to do.
		
0x7001	SET_LOCK_WAIT
		Will attempt to set a Lock, and will wait until LOCK_WAIT_TIMEOUT seconds before exiting.
		
0x7002	LOCKED
		Will provide the LOCK_NAME, EXPIRE_SECONDS, UNLOCK_CODE of a lock that has been set.
		
0x7003	LOCK_FAILED
		Will provide the LOCK_NAME of the failed lock.   Lock can fail immediately if SET_LOCK was used, or after the timeout period if SET_LOCK_WAIT was used.

// 1 byte-length string - 0x8000 to 0x8fff   1 000 xxxx xxxx
// 2 byte-length string - 0x9000 to 0x9fff   1 001 xxxx xxxx

0x9000	USERNAME <username>
		Used for authentication components.  Normally is used to establish a client certificate.
		
0x9001	PASSWORD <password>
		Used for authentication componenets.
		
0x9002	CSR <csr>
		Client can generate a CSR and ask it to be signed by the CA for the Locks service (zone specific).
		
0x9003	ZONE <zone>
		The zone name is a specific isolated tenented zone.  The user must be authenticated for the zone to allow access to anything.
		
0x9004	LOCATION_DOMAIN <>
		The location domain indicates physical or logical properties that can allow grouping of services.  For example, the client can indicate what country and city it is in, and the Locks service can provide appropriate connection details to a service that is more appropriate for that location.

0x9005	LOCK_NAME <>
		The lock name.
		
0x9006	SERVER_NAME <>
		When servers are connecting to each other, they can provide their configured names.  This can make it easier to debug and manage.
		
0x9007	CLIENT_NAME <>
		When a client connects, it can provide a client name.  Server can be configured in multiple ways, to restrict the connections, or allow them.

		
		
// 4 byte-length string - 0xa000 to 0xafff   1 010 xxxx xxxx
// 8 byte-length string - 0xb000 to 0xbfff   1 011 xxxx xxxx
// No Parameters - 0xc000 to 0xffff          1 1xx xxxx xxxx

```
