# Command Protocol

All operations performed on the OpenCluster system will be initiated through a specific Command protocol.
There will also be an HTTP REST API as well, but that will still use the underlying protocol to actually inject the operations.

## Authentication.

The security system for OpenCluster needs to be very robust and advanced.  It will be entirely possible that the Interface will be exposed to the general internet in many organisations, traversing through many uncontrolled networks, therefore the security needs to be as unbreakable as possible.

Authentication is done in a way where password credentials never have to be stored on the server or transmitted to it.

There will be two kinds of accounts... well actually, two options.

1. Passphrase protected certificate can be sent to the client requesting login.  Client must then have the passphrase to unlock the certificate.  Typically used for accounts for people.
2. Certificate for an account does not have a passphrase, but also are never delivered to the client.  Client must already have the certificate.  Typically used for non-human interfaces.  The certificate on the system must be secured.

The general flow for a user authenticating.

1. Client sends the username to the server.
2. Server responds with a protected certificate along with some session credentials.
3. Client unpacks the certificate with a passphrase.
4. Client decrypts a special code within the session to prove they control the cert.
5. Client sends the session packet back to the server.
6. Server verifies the session packet was decrypted correctly, proving the user had the correct passphrase.

## Encryption.

Once authentication is complete, all further communications need to be encrypted.

### Encryption Negotiation.

We dont want encryption choices to be difficult, but we need to be flexible to support newer encryption protocols as they are released, and stop using older ones that are no longer reliable.

Server will have a list of valid encryption options, and the admins can turn off ones, and give them different priorities. 

1. Client requests encryption options.
2. Server sends list of encryption options and order of choice and an ID.
3. Client chooses the encryption method that fits best, and replies.
4. Server now marks that session to be using that particular encryption type.


### Sending Encrypted data.

Once the client knows what the preferred encryption method is, and supports it, it will use it for encrypting the communications.

When the library is requested to send some data.  It will wrap that raw data inside an encrypted command.  All the previous negotiations would have locked in certain parameters.

psuedo-risp:
```
WRAPPED_ENCRYPTED_COMMAND
  LENGTH_OF_UNCRYPTED STRING
  ENCRYPTED_STRING
```

Note that in the wrapped command, it only includes the encrypted string, and the length of the original.  All the settings around encryption must already have been established.

The encryption method is used for both sending and receiving.  All data received from the server will be encrypted in the same way.

### Validating Encrypted String.

When an encrypted string is received, we need to know if the unencryption was successful.  The session parameters will indicate what is expected.  Typically though, a random integer is provided in the session params, and that must be added to the start of the unencrypted string, or alternatively a seperate command must be provided that encrypts a number, or even a number sequence, and includes that as a seperate RISP command.

This should be flexible, because we dont know what encryption protocols will be recommended in the future.

### Re-key

When establishing an encrypted session, the server will include some session parameters.  One of those parameters will be an expiry time.  Before the session expires, the client must re-establish the session encryption.  

It is up to the client to do this.  If it does not, its session will expire, and the server will no longer accept commands. 

### Disconnection

If the server receives more than a certain number of commands that do not fit the current session, then it will disconnect the socket.  The system can be setup to handle and block certain connections if they cross a threshold on invalid connections and block that IP for a time.


## Operations

### Sending Commands

The operations themselves dont have corresponding RISP commands.  Instead there is a command structure that can send any command to the server.  This means that the protocol should not change when new operations are added to clients and servers.

The parameters supplied to a command are a simple array of params, and can be whatever structure is needed for the command.  This allows for a very flexible command structure, that is easily loggable and re-runnable. 

> DESIGN DECISION:
> A purist would likely suggest that the command structure should be pure RISP rather than mixing RISP which can be a fairly decent binary expression of what JSON can do.  However, the main reasons NOT to use JSON for the entire protocol, is that it is difficult to stream.  Until you have processed an entire structure, you have no idea how much data you need to receive.  With RISP we can wrap a flexible JSON structure in a binary protocol that is easier to stream.  This makes the protocol far less complicated to implement and for other to re-implement.

```
OPERATION
  ID <id number>
  GROUP "op group"
  COMMAND "op command"
  PARAMS <JSON Params Structure>
```

The ID is a client specific ID that is not mandatory, but useful to match up responses.  The response will include the ID whatever it was, or not include it if it wasn't given.

### Receiving Results.

The result returned will normally only require a request code, a certain structure of information (maybe we will use JSON for that?), and maybe some textual output.

```
RESULT
  ID <id number>
  RESULT <result code, 0 normally means success>
  DATA <JSON formatted result data>
  OUTPUT <textual output normally for display>
```

