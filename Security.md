# Security

The entirety of the OpenCluster products need to take security at a very high level.  Security needs to be implemented in a way that doesn't become a huge barrier for entry though.  This means that all stored 'data' needs to be be encrypted, and if possible, the keys should not be available.  

If the storage device gets compromised, they should not have access to the data.
This means, that the keys to access the data, should require the client key to unlock.
This is a very difficult requirement to engineer.

## Cloud based Hosts

For the most part, the hosts people configure will be in the 'cloud' of some sort.  This means the server is publically available, on infrastructure that is provided by another company, with staff at that company having the ability to login to those servers and provide support.  It is also possible that the data on that server could be snapshotted without your knowlege.   How do you secure such a system?


## Highly Secure Nodes 

### Short Host lifetimes

One way to make a node more secure would be to configure it with a short lifetime.  If the node only exists for a short time before being destroyed and re-created, then it is far more difficult for an attacker to maintain a foothold on it.

The more risky the environment, the shorter the lifetime should be.   This also means that there is an advantage by having systems that can can be spun up, or re-created quickly.

### Short configuration lifetimes.

Even if the host has a short lifetime, another option is to have short config lifetimes.  In this case, the passwords and authentication being used, should have a short lifetime and be renewed frequently and co-ordinated, so if credentials are stolen, they only have a limited lifetime.  There are a number of complications with this approach, and also we run the risk of not being able to detect if a rougue system is requesting the new passwords.

## Database Access

Normal security principles apply.  Each service that needs to access a database should have a particular account that just gives access to the components that is needed.  Habing completely seperate databases for each microservice is also recommended.  You need to ask "If this account is compromised, what could it do?"

