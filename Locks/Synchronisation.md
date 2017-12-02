# Locks Synchronisation

The Locks service has multiple nodes, without the need of a master node.  

When a new node connects, it needs to ask for the Locks picture as it currently sits.

Once a new node has connected, and it is ready to received the synchronisation, it will send a REQ_SYNC command to one of the servers that it is connected to.


Question:
  Should the config items be sycn'd as a unit regardless of the content of the config.  Or should each type of config item be synchronised seperately?  
  
Question:
  Should the sync items be a JSON structure for simplicity?  Should it be LUA commands?   Should it be a key/value system?   If it is a key/value system, would need to keep the syncID with the key/value so that it can replay as needed.   An index (singly-linked-list) to the syncID could allow for iteration.  A binary tree might be more suitable for the key/value lookup (glibc btree would work well here).

Question:
  Should config items take effect immediately, or wait until quorum have responded?   
  
Question:
  Should the server making the config change be the one to distribute to ALL other nodes?
Answer:
  That is the simplest solution.  However, if new nodes are connecting, then it can include its current config level when it does a heartbeat.

# Quorum.

Since some of the synchronisation mechanisms within the Locks environment uses a majority as confirmation, there needs to be an Odd number of nodes.   To accomplish this, even if there are an even number of servers in the cluster, it will only allow an odd number of them to be active.   

To reduce impacts on clients, the node that is disabled, will still accept traffic, and will still receive updates on the locks status, but it will not be authoritative.   Clients that connect to it will have requests responded to.  If teh client requests a lock, the server will pass the request onto another authoritative node.

If another node shuts down, the pending node can join the cluster with a minimum amount of catch-up (as it's config should already be synchronised).


# Locks 

When a node has joined the cluster, it will send a REQ_LOCK_SYNC message to one of the nodes it is connected to.  
The responding server will send a LOCK_TOKEN (which is essentially a number that increments with every Lock action (Lock, unlock, expire, etc)).
The responding server will then send, one after the other, the LOCK details to the new server (as if they were just made).

# Client ID Synchronisation

When clients connect to the cluster, they are given a client ID.  This is a random number that should be unique.  All of the Locks Nodes should maintain a list of connected clients (and especially this Client ID).   The clients can therefore connect to multiple cluster members, and it is known that they are connected.   If the client loses connectivity to one server, but maintains connectivity to other servers, it can therefore keep the locks that it has set.  It can also inform the servers that it has lost connectivity to one of teh nodes.  This can be useful if locks servers are responding to each other, but not to clients.   When a client is able to connect to one node, but not another, it can report that to the server.  The server will continue to run, but can pass that information on to monitoring agents.



