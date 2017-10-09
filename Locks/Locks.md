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

Most OpenCluster products support [Location Domains](../LocationDomains.md), and the Locks product does as well.   

All the Locks servers in all locations are part of the same over-all cluster. And locks are kept in sync over the entire organisation, but efforts are in place to try and reduce the amount of traffic that occurs between those locations.

Depending on how you configure the services, you might restrict clients to only connect and use Locks services that exist in the same location domain.  For example, the clients in perth might be configured to use locks servers that exist in "au.perth".  This means they can connect to any of the locks servers in both 'au.perth.dh1' and 'au.perth.dh4'.  Those same clients wont try to connect to Locks servers in 'au.sydney'.

This also means that the Locks Servers themselves which need to talk to other members of the Cluster can designate one primary node in each domain to pass communications back and forth between locations.  An example, you might have 4 locks servers in au.perth.  Only one of them will be designated to represent all 4 to the other location domains.


## Questions 

* Should the server-server connectivity be on a seperate socket than the client connectivity?    
  * Multiple ports mean more firewall rules needed.
  * Multiple ports mean that server to server communication can be locked down to only locks servers.
  * Server to Server communication can be setup to accept different client certificates.
  * Advantage of a single port is that there is only one protocol.
  * Disadvantage of a single port is that the protocol is more complicated and you need extra code to ensure that clients dont use commands that should be restricted to servers.
  * _Yes._

  
## Starting a New Locks Cluster.

When the services start, and there is some config that is stored on disk, then it should know how to connect to the cluster, or start a new one.  This config may have a DNS entry, and the locks server can try all the IP's that are returned for that DNS entry, and if it cant connect to any of them (and it's IP appears to be one of them in the list), then it can assume that it is the first member of the cluster.

The config should also indicate the minimum quorum (not just a percentage).  This means that if the config indicates that there should be a minimum of 3 locks servers running, then it will not open up client connectivity until that many servers are talking with each other.

If there is no config however.  The server is being started for the first time.  Then it will need to be specifically indicated that it is the first node being created in the cluster.
  
  
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


## Config File requirements

The config file should contain information that can be identical on all locks servers (which means that it should be possible to deploy the exact same config file to all locks servers).   The locks service does need to store data that is unique to it, so the config file should point to other files that contain this data.

Note that the main config files does not have to be the same on all boxes, but it is desired.

### Identifier

When a locks service first starts up, if the file that 'identifier' is referring to doesn't exist, it will create a random number, and use that number to identify itself. 
Config inside the Locks system can add friendly names to those identifiers.
Location domains are applied to the identified systems, but again, those details are stored in the cluster itself, not in local config on the box.

No two servers should exist in the cluster with the same identifier.

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

### Requirements



#### Server to Server

1. When the service starts it listens on a particular socket (accepting TLS1.3 connections)
1. Once the configuration has loaded, and it has a quoram, it will accept client connections.
1. Server attempts to connect to other Locks services (stored locally or in config).
1. If config dictates, server connects to Load-Balancer to inform them it is ready to receive traffic, and the zone it is in.
1. Server expects client connections to use a client certificate for authentication.
1. Server can be told what location domain it is.  The location domain is very typically used within all of OpenCluster services.
1. Each locks server should be connected to every other locks server within the location domain.  
1. Each locks server keeps track of how many other locks servers are in the cluster.
1. Each locks server maintains a complete picture of what locks have been presented.
1. When the locks server receives connections from clients and have received invalid commands, it can be configured to block connections from that IP for a certain length of time.
1. Administrators can list all the nodes in the cluster.
1. Administrators can remove nodes from the cluster.
1. Administrators can list all the new server requests (Each new server must be approved either explicitly or implicity).
1. Administrators can list all the locks in the cluster.
1. Administrators can list all the locks in the cluster, where the client is not connected.
1. Administrators can list all the clients connected to the cluster.
1. Administrators can over-ride locks and remove them.


#### Low-Level

1. When the client requests a lock, it sends the name of the lock it wants to set, its expiry time (in seconds).
1. When releasing a lock, the client signs a peice of standard text with the private key, therefore proving that they have the private key, and therefore own the lock, and can release it.
1. When a server receives a request from a client to set a lock, it will pass that request on to all the other nodes in the cluster.   When it gets confirmation from more than half of the servers, it will know that the lock is successful, and will inform the client.

### Protocol 



### Server to Server Commands

RISP-based Protocols can be either flat, or structured.  For this use-case, a flat protocol is probably more efficient.   This is typically the case for protocols that are fairly static in their implementation and operation.  Since these operations are very much 1 to 1, and mostly simple, then 


```
// 1 byte integer - 0x0000 to 0x0fff         0 000 xxxx xxxx (8-bit)
// 2 byte integer - 0x1000 to 0x1fff         0 001 xxxx xxxx (16-bit)

0x1000	CAPABILITY <command-id>
		This command asks the service if it supports a specific command.  It's parameter is a 16-bit command.   Can be used by both server and client.
		Responds with:
			OK_COMMAND <command-id>
			INVALID_COMMAND <command-id>
			
0x1001	OK_COMMAND <command-id>
		When a client or server wants to know if particular commands are supported, it will ask 'CAPABILITY <command-id>'.  If the command is supported by the server, it will respond with this OK_COMMAND.   
		
0x1002	INVALID_COMMAND <command-id>
		If an invalid command is received by the client or server, it will respond with this command.  Both client and server need to ensure that services are handled correctly if this result is received.  It typically means that either protocol is out-of-date.   To verify the capability of the opposite connection to be able to handle the commands expected prior to actually using them, see the CAPABILITY command.

// 4 byte integer - 0x2000 to 0x2fff         0 010 xxxx xxxx (32-bit)
// 8 byte integer - 0x3000 to 0x3fff         0 011 xxxx xxxx (64-bit)
// 16 byte integer - 0x4000 to 0x4fff        0 100 xxxx xxxx (128-bit)


// 32 byte integer - 0x5000 to 0x5fff        0 101 xxxx xxxx
// 64 byte integer - 0x6000 to 0x6fff        0 110 xxxx xxxx
// No Parameters - 0x7000 to 0x7fff          0 111 xxxx xxxx
// 1 byte-length string - 0x8000 to 0x8fff   1 000 xxxx xxxx (255 byte string)
// 2 byte-length string - 0x9000 to 0x9fff   1 001 xxxx xxxx (64k byte string)

0x9003	ZONE <zone>
		The zone name is a specific isolated tenented zone.  The user must be authenticated for the zone to allow access to anything.
		
0x9004	LOCATION_DOMAIN <>
		The location domain indicates physical or logical properties that can allow grouping of services.  For example, the client can indicate what country and city it is in, and the Locks service can provide appropriate connection details to a service that is more appropriate for that location.

// 4 byte-length string - 0xa000 to 0xafff   1 010 xxxx xxxx ()
// 8 byte-length string - 0xb000 to 0xbfff   1 011 xxxx xxxx
// No Parameters - 0xc000 to 0xffff          1 1xx xxxx xxxx

```

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
		Client can generate a CSR and ask it to be signed by the CA for the Locks service.
		
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
