# OpenCluster-Locks

Allows different processes a method to perform functionality that requires other components do not interfere at the same time.    For example, you might have 4 instances of a service that performs account management, but you only want one instance making changes to a particular account at the same time...  

The Locking service is used similar to Mutex's and thread-locking in single-process applications, but has been expanded to a cluster of applications.

It is important to note that the Locking service does not have any way to enforce the lock.  The applications accessing the resource must comply with the Locking agreements.
