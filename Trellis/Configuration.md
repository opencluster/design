# Trellis Configuration

This document describes how new configuration will be stored and deployed within the environment.

There will be two kinds of config information in a running system.  
1. Source Config
2. Running Information

# Design Decisions

* [Source Config or Direct API](Decisions/SourceOrDirect.md)


## Running Information

The running information is the state of things in a running environment.  This state is shared amongst all the controllers in the environment.


# Trellis Controller Configuration

The Trellis controller will need some configuration for itself.
This will include:
* Where and how to access the cluster configuration.  
* Authentication methods.
* Links to controllers in other environments if they are seperate.
* Logging

# Cluster Configuration.

The Cluster configuration will be stored internally, but very similar to how [OpenCluster-Data(../Data/Data.md) service works.  Each configuration change will be wrapped RISP, and each controller will process it and ensure that they are all in sync.


# Configuration Changes

Each actual change will be a part of a [ChangeSet](ChangeSet.md), and this will be made up of a bunch of commands.  When a changeset is committed and applied, it will first be sent to all the active controller nodes, and then when all controllers have the config, then it will be processed on all controllers at the same time, in a co-ordinated fashion.



# Synchronising the Config across controllers.

Mechanisms and controls needs to be in place to sync the config between the controllers, and to recover when a controller fails or gets corrupted.  If changes are made to a controller while it is seperated from the cluster, need to be able to recover.

# Off-site configuration.

OpenCluster will provide a service where users can have their config encrypted and stored offsite, so that if they have a total data-center outage, they can recover their environment.  There is no ability to also store the data in there, (although we could provide that as a paid option).  Essentially, if their system goes offline, they can spin up a whole new environment in, say, Amazon and have everything up and running again.


