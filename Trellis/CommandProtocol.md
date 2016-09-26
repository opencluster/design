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


