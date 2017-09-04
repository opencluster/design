# OpenCluster-Locks

Allows different processes a method to perform functionality that requires other components do not interfere at the same time.    For example, you might have 4 instances of a service that performs account management, but you only want one instance making changes to a particular account at the same time...  

The Locking service is used similar to Mutex's and thread-locking in single-process applications, but has been expanded to a cluster of applications.

It is important to note that the Locking service does not have any way to enforce the lock.  The applications accessing the resource must comply with the Locking agreements.

Important Warning.
Using Locks should be avoided where possible in the architecture.  Using a lock can cause problems during a Disaster-Recovery or High-Availability event.  If the system that is holding a lock has a failure, then recovery of that Lock can be consequential.   Systems should be designed where a lock is unnecessary.  Locks are typically only used for very sparse situations where it cannot be avoided.

An alternative to using locks is to use a Message Queing system that only delivers requests to an active node.

The Locks system has the concept of walled-off zones.   
It is very similar to the OpenCluster-Data service,  but only records the state of a lock.  
When a lock is established, the caller generates a private key and signs a record.  In order to unlock the lock, they prove they still have the key.

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

Location Domains are used to provide detail to the servers and clients around the location of the services.  In some organisations this does not matter, but some organisations have seperated data centers, or regional zones, and they want as little traffic as possible to go over the links between those data centers.  This means that clients connect to servers in that zone, and only cross over to the other zones when there is no other choice.   The actual structure of the Location Domain is entirely up to the implementation.   

As an example, lets say that a fairly large organisation has a presence in the US, Australia, and Europe.   In the US, they have two datacenters, one on the west coast, one on the east coast.  In Australia they have two datacenters, one in Perth, and one in Sydney.  In Europe, they have just one datacenter.  In the datacenter in Perth, they have two sets of racks, one set in DataHall1, and one set in another part of the building DataHall4.

You might then have multiple Location Domains.
* au.perth.dh1
* au.perth.dh4
* au.sydney
* us.east
* us.west
* eu

Keep in mind, that this structure is completely up to you to define in your implementation.  There are no hard-configured parts of this structure.

All the Locks servers in those locations are part of the same over-all cluster. And locks are kept in sync over the entire organisation, but efforts are in place to try and reduce the amount of traffic that occurs between those locations.

Depending on how you configure the services, you might restrict clients to only connect and use Locks services that exist in the same location domain.  For example, the clients in perth might be configured to use locks servers that exist in "au.perth".  This means they can connect to any of the locks servers in both 'au.perth.dh1' and 'au.perth.dh4'.  Those same clients wont try to connect to Locks servers in 'au.sydney'.

This also means that the Locks Servers themselves which need to talk to other members of the Cluster can designate one primary node in each domain to pass communications back and forth between locations.  An example, you might have 4 locks servers in au.perth.  Only one of them will be designated to represent all 4 to the other location domains.

You may also want to designate the provider of the services.  For example, if you use Amazon Web Services for compute, as well as DigitalOcean, and Azure.  You could have your Location Domain be represented by provider as the highest level of the structure.
* amazon.sydney
* amazon.east
* amazon.west
* azure.sydney
* azure.east
* do.singapore


Or alternatively, you may actually want the location to be of higher importance.

* au.sydney.amazon
* au.sydney.azure
* us.east.amazon
* us.east.azure
* asia.singapore.do

The Location Domain distinction is completely up to you.  The Locks services will try and use this information to improve responsiveness, and reduce bandwidth.   It is important to remember that the Cluster will be treated as a single entity, even if it is spread all over the globe.   

If you want to have distinct difference between locations, then you would implement multiple clusters (and probably co-ordinate those clusters with another cluster that co-ordinates changes between locations).






## Questions 

* Should the server-server connectivity be on a seperate socket than the client connectivity?    
  * Multiple ports mean more firewall rules needed.
  * Multiple ports mean that server to server communication can be locked down to only locks servers.
  * Server to Server communication can be setup to accept different client certificates.
  * Advantage of a single port is that there is only one protocol.
  * Disadvantage of a single port is that the protocol is more complicated and you need extra code to ensure that clients dont use commands that should be restricted to servers.
  * Yes.

  
  
## Protocol

### Requirements

#### High-Level 

1. When the service starts, it loads the config that tells it what other locks servers to connect to (this can be either a list of IP's of actual locks servers, or it can be the address of a load-balancer).
1. If the startup option indicates that it is known that this server is the first node in the cluster, it will start the cluster.
1. If not specifically indicated that this is the first instance of the Locks service, The service connects to at least one other locks server that is part of the cluster.
1. When the service starts it listens on a particular socket (accepting TLS1.3 connections)
1. Once the configuration has loaded, and it has a quoram, it will accept client connections.
1. Server attempts to connect to other Locks services (stored locally or in config).
1. If config dictates, server connects to Load-Balancer to inform them it is ready to receive traffic, and the zone it is in.
1. Server expects client connections to use a client certificate for authentication.
1. Server can be configured to not provide Locks server details to clients.   
1. Server can be told what location domain it is.  The location domain is very typically used within all of OpenCluster services.
1. Each locks server should be connected to every other locks server within the location domain.  
1. Each locks server keeps track of how many other locks servers are in the cluster.
1. Each locks server maintains a complete picture of what locks have been presented.
1. When the locks server receives connections from clients and have received invalid commands, it can be configured to block connections from that IP for a certain length of time.
1. Administrators can:
  * list all the nodes in the cluster.
  * remove nodes from the cluster.
  * list all the new server requests.
  * list all the locks in the cluster.
  * list all the locks in the cluster, where the client is not connected.
  * list all the clients connected to the cluster.
  * over-ride locks and remove them.


##### Low-Level

1. Before requesting a lock, the client generates a private key pair.  
1. When the client requests a lock, it sends the name of the lock it wants to set, its expiry time (in seconds), and the public key.
1. When releasing a lock, the client signs a peice of standard text with the private key, therefore proving that they have the private key, and therefore own the lock, and can release it.
1. When a server receives a request from a client to set a lock, it will pass that request on to all the other nodes in the cluster.   When it gets confirmation from more than half of the servers, it will know that the lock is successful, and will inform the client.

### Protocol 



### Server to Server Commands

RISP-based Protocols can be either flat, or structured.  For this use-case, a flat protocol is probably more efficient.   This is typically the case for protocols that are fairly static in their implementation and operation.  Since these operations are very much 1 to 1, and mostly simple, then 


```
// 1 byte integer - 0x0000 to 0x0fff         0 000 xxxx xxxx
// 2 byte integer - 0x1000 to 0x1fff         0 001 xxxx xxxx
// 4 byte integer - 0x2000 to 0x2fff         0 010 xxxx xxxx
// 8 byte integer - 0x3000 to 0x3fff         0 011 xxxx xxxx
// 16 byte integer - 0x4000 to 0x4fff        0 100 xxxx xxxx

0x4000	CAPABILITY <command-id>
		This command asks the service if it supports a specific command.  It's parameter is a 16-bit command.   Can be used by both server and client.
		Responds with:
			OK_COMMAND <command-id>
			INVALID_COMMAND <command-id>
			
0x4001	OK_COMMAND <command-id>
		When a client or server wants to know if particular commands are supported, it will ask 'CAPABILITY <command-id>'.  If the command is supported by the server, it will respond with this OK_COMMAND.   
		
0x4002	INVALID_COMMAND <command-id>
		If an invalid command is received by the client or servver, it will respond with this command.  Both client and server need to ensure that services are handled correctly if this result is received.  It typically means that either protocol is out-of-date.   To verify the capability of the opposite connection to be able to handle the commands expected prior to actually using them, see the CAPABILITY command.

// 32 byte integer - 0x5000 to 0x5fff        0 101 xxxx xxxx
// 64 byte integer - 0x6000 to 0x6fff        0 110 xxxx xxxx

0x6000	REQUEST_TOKEN

// No Parameters - 0x7000 to 0x7fff          0 111 xxxx xxxx
// 1 byte-length string - 0x8000 to 0x8fff   1 000 xxxx xxxx
// 2 byte-length string - 0x9000 to 0x9fff   1 001 xxxx xxxx
// 4 byte-length string - 0xa000 to 0xafff   1 010 xxxx xxxx
// 8 byte-length string - 0xb000 to 0xbfff   1 011 xxxx xxxx
// No Parameters - 0xc000 to 0xffff          1 1xx xxxx xxxx

```

### Client to Server Commands
```
// 1 byte integer - 0x0000 to 0x0fff         0 000 xxxx xxxx
// 2 byte integer - 0x1000 to 0x1fff         0 001 xxxx xxxx
// 4 byte integer - 0x2000 to 0x2fff         0 010 xxxx xxxx
// 8 byte integer - 0x3000 to 0x3fff         0 011 xxxx xxxx
// 16 byte integer - 0x4000 to 0x4fff        0 100 xxxx xxxx
// 32 byte integer - 0x5000 to 0x5fff        0 101 xxxx xxxx
// 64 byte integer - 0x6000 to 0x6fff        0 110 xxxx xxxx
// No Parameters - 0x7000 to 0x7fff          0 111 xxxx xxxx
// 1 byte-length string - 0x8000 to 0x8fff   1 000 xxxx xxxx
// 2 byte-length string - 0x9000 to 0x9fff   1 001 xxxx xxxx
// 4 byte-length string - 0xa000 to 0xafff   1 010 xxxx xxxx
// 8 byte-length string - 0xb000 to 0xbfff   1 011 xxxx xxxx
// No Parameters - 0xc000 to 0xffff          1 1xx xxxx xxxx

```
