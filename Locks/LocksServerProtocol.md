# Locks Server Protocol

This document describes the protocol and methods used for all Server to Server communications.

## Questions 

* Should the server-server connectivity be on a seperate socket than the client connectivity?    
  * Multiple ports mean more firewall rules needed.
  * Multiple ports mean that server to server communication can be locked down to only locks servers.
  * Server to Server communication can be setup to accept different client certificates.
  * Advantage of a single port is that there is only one protocol.
  * Disadvantage of a single port is that the protocol is more complicated and you need extra code to ensure that clients dont use commands that should be restricted to servers.
  * _Yes._

* Should the server-server connectivity be allowed to be configured to use the same socket as client connectivity?
  * _No._

  
## Requirements

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


## Low-Level

1. When the client requests a lock, it sends the name of the lock it wants to set, its expiry time (in seconds).
1. When releasing a lock, the client signs a peice of standard text with the private key, therefore proving that they have the private key, and therefore own the lock, and can release it.
1. When a server receives a request from a client to set a lock, it will pass that request on to all the other nodes in the cluster.   When it gets confirmation from more than half of the servers, it will know that the lock is successful, and will inform the client.


## Protocol 

### Setting initial config and authentication.

When a server is connecting to another server it can either already have a client certificate that is used for authentication, or one can be created after authentication from an administrator account.

```
oclocks-addserver -a -o -n <servername> -h <IP:port> -u <username> -p<password>
```

The `oclocks-addserver` command will authenticate to the specified existing locks server.  It will generate a new private key and CSR, and send it to the server to sign.  If no servername is provided it will just use the current hostname.  If the specified servername already exists in the cluster, unless `-a` or `-o` is specified on the command-line, it will report that the server already exists.  `-o` option will cause it to overwrite the existing server (ie, remove the existing server from the cluster and replace it with this one).  `-a` option will allow it to add the new server and not remove the existing, allowing you to have multiple servers with the same name.

When connecting, it will not have a client certificate, so it will be connecting without one.  Its options will be limited, but can provide a username/password of an administrator.  If that is valid, it will create a private key, and ask for the CSR credentials that it should create (the Option).  It will generate a CSR and submit it to the server.  The server will sign it with the Certificate specifically used to authenticate server nodes, and send back the signed certificate.

1. Check for existing config on the server.
2. Generate a new private key.
3. Generate a random 64-bit serverID.
2. Connect to the specified Locks server without a client-certificate.
4. Authenticate and ask for the CSR defaults to use.

```
Send:
  USERNAME <username>
  PASSWORD <password>
  REQ_CSRDEFAULTS
  
Receive (only if username and password were valid):
  CSR_DEFAULTS <defaults>
  
Receive (only if username and password were invalid):
  INVALID_AUTH
```

1. Generate a CSR based on the defaults provided.
2. Send the CSR to be signed, and provide the SERVERID and SERVERNAME.

```
Send:
  SERVERID <serverid>
  SERVERNAME <name> (optional)
  OVERWRITE_SERVER (optional)
  ALLOW_DUP_NAME (optional)
  SIGN_CSR <csr>

Receive (only if previously authenticated correctly, and all options are valid):
  ACCEPTED
  
Receive (if either previous authentication failed, or options are unable to be accepted):
  REJECTED  
```

### Authenticating

Under normal circumstances, the connection should be established with a client-certificate.   When this occurs, then the connection is already authenticated, and nothing needs to be done.   However, if the client connects without a client certificate, they can provide a USERNAME and PASSWORD, along with the AUTHENTICATE command, and it will check those credentials, and either reply with ACCEPTED or REJECTED.

```
Send:
  USERNAME <username>
  PASSWORD <password>
  AUTHENTICATE

Receive (only if previously authenticated correctly, and all options are valid):
  ACCEPTED
  
Receive (if either previous authentication failed, or options are unable to be accepted):
  INVALID_AUTH
```

### Config sync

Once a server is connected to one of the other servers in the cluster, it needs to get all the config of the other servers.  Config includes the list of servers that are in the cluster, and the authentication details (user accounts, certificates, etc).

When a server is first connected, its list of changes is empty, so it indicates that the last change it has is 0.   The receiving server will then send all newer config changes since that one.

The changes are essentially a list of commands that do something.  They might add a new server to the list of servers, remove a server, change some details of the server, add a user, apply certificate details for a user, or server, etc.... 

Whenever an action is recieved on one server, that particular server will send out a message to all other servers with the new config details.   The new details are not actually considered active, until the majority of other nodes have also acknowledged that they have the config.  

When a server receives a new config item, it sends a short message to all other nodes stating that its new config count has changed.  This means that a server might actually receive those config notifications prior to receiving the actual config update itself.  That is ok, it will simply wait for the config update.

```
Send:
  REQ_CONFIG_SYNC <last change>




### Locks Sync

Once the session is authenticated, it needs to establish all the data that is currently in the cluster.


*******************************************



## Server to Server RISP Commands

RISP-based Protocols can be either flat, or structured.  For this use-case, a flat protocol is probably more efficient.   This is typically the case for protocols that are fairly static in their implementation and operation.  Since these operations are very much 1 to 1, and mostly simple, then 

```
// 1 byte integer - 0x0000 to 0x0fff         0 000 xxxx xxxx (8-bit)
// 2 byte integer - 0x1000 to 0x1fff         0 001 xxxx xxxx (16-bit)

0x1000	CAPABILITY <command-id>
		This command asks the service if it supports a specific command.  It's parameter is a 16-bit command.   Can be used by both server and client.
		Responds with either:
			OK_COMMAND <command-id>
			INVALID_COMMAND <command-id>
			
0x1001	OK_COMMAND <command-id>
		When a client or server wants to know if particular commands are supported, it will ask 'CAPABILITY <command-id>'.  If the command is supported by the server, it will respond with this OK_COMMAND.   
		
0x1002	INVALID_COMMAND <command-id>
		If an invalid command is received by the client or server, it will respond with this command.  Both client and server need to ensure that services are handled correctly if this result is received.  It typically means that either protocol is out-of-date.   To verify the capability of the opposite connection to be able to handle the commands expected prior to actually using them, see the CAPABILITY command.

// 4 byte integer - 0x2000 to 0x2fff         0 010 xxxx xxxx (32-bit)
// 8 byte integer - 0x3000 to 0x3fff         0 011 xxxx xxxx (64-bit)

0x3000	SERVERID <serverid>
		Used to represent a server.

// 16 byte integer - 0x4000 to 0x4fff        0 100 xxxx xxxx (128-bit)


// 32 byte integer - 0x5000 to 0x5fff        0 101 xxxx xxxx
// 64 byte integer - 0x6000 to 0x6fff        0 110 xxxx xxxx
// No Parameters - 0x7000 to 0x7fff          0 111 xxxx xxxx

0x7000	AUTHENTICATE
		Authenticate via the USERNAME and PASSWORD parameters supplied (might also need to provide ZONE)

0x7001	REQ_CSRDEFAULTS
		This command will request the CSR subject defaults.

// 1 byte-length string - 0x8000 to 0x8fff   1 000 xxxx xxxx (255 byte string)
// 2 byte-length string - 0x9000 to 0x9fff   1 001 xxxx xxxx (64k byte string)

0x9000	USERNAME <username>
		Used for authentication components.  Normally is used to establish a client certificate.
		
0x9001	PASSWORD <password>
		Used for authentication componenets.
		
0x9002  SERVERNAME <servername>
		Used to identify specific locks servers.  Not mandatory, but useful.  Within the system, servers will be identified by SERVERID, but SERVERNAME provides a human-friendly way to represent the server.

0x9003	ZONE <zone>
		The zone name is a specific isolated tenented zone.  The user must be authenticated for the zone to allow access to anything.
		
0x9004	LOCATION_DOMAIN <>
		The location domain indicates physical or logical properties that can allow grouping of services.  For example, the client can indicate what country and city it is in, and the Locks service can provide appropriate connection details to a service that is more appropriate for that location.
		
0x9005	CSR_DEFAULTS <>
		This is used to tell clients (and newly connected servers) what CSR creds they should put in when requesting a CSR to be signed.  This is not necessary for the operation of the certificate, but it allows the clients to create certificates that are identifiable as being for this service (for operational consistancy)  It will be a string that looks like "C=AU, ST=WA, O=Sample Locks Server"


		
// 4 byte-length string - 0xa000 to 0xafff   1 010 xxxx xxxx (4gb string)
// 8 byte-length string - 0xb000 to 0xbfff   1 011 xxxx xxxx (1.844674407×10^19 byte string)
// No Parameters - 0xc000 to 0xffff          1 1xx xxxx xxxx

0xc000	ACCEPTED
		Generic responce when a more specific responce is not required.

0xc001  REJECTED
		Generic responce when a more specific responce is not required.

0xc002	INVALID_AUTH
		This command is sent if the credentials supplied are invalid (either USERNAME and PASSWORD, or the client-certificate used for the connection).

```