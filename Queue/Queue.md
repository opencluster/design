# OpenCluster-Queue

Services can send and receive work over designated queues.  This is a very useful design for scalability.   
Queues can be handled in different ways:
* **Work queues** - Items are put on the queue, and multiple services can pull items off the queue and process them.   Only one service can process an individual item.  All items in a work queue are expected to deliver a reply.
* **Broadcast Queues** - Items are put on the queue and delivered to all nodes listening on it.  Replies are not possible on broadcast queues.  If there are no services online to receive the message then it will remain on the queue until at least one is.
* **Static Queues** - These queues require careful managment of resources but can be very useful.  All messages sent on the queue remain (although they can expire).  When a service connects to the queue it can receive ALL the messages on it.    It is up to the service to know which messages it has already received.  This could be useful to keep services in sync.  Keys are used to control the queue.  If you have the right key, you can remove items from the Static Queue.
