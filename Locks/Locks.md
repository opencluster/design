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

## Questions 

* Should the server-server connectivity be on a seperate socket than the client connectivity?    
** Multiple ports mean more 

## Protocol

### Requirements

1. When the service starts, it loads the config that tells it what other locks servers to connect to (this can be either a list of IP's of actual locks servers, or it can be the address of a load-balancer).
1. If the startup option indicates that it is known that this server is the first node in the cluster, it will start the cluster.
1. If not specifically indicated that this is the first instance of the Locks service, The service connects to at least one other locks server that is part of the cluster.
1. When the service starts it listens on a particular socket (accepting TLS1.3 connections)
1. Once the configuration has loaded, and it has a quoram, it will accept client connections.
1. Server attempts to connect to other Locks services (stored locally or in config).
1. If config dictates, server connects to Load-Balancer to inform them it is ready to receive traffic, and the zone it is in.
1. Server expects client connections to use a client certificate for authentication.
1. Server 

### Commands
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
