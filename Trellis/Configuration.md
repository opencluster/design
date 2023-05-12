# Trellis Configuration

This document describes how new configuration will be stored and deployed within the environment.

There will be two kinds of config information in a running system.  
1. Source Config
2. Running Information

# Design Decisions

* [Source Config or Direct API](Decisions/SourceOrDirect.md)


## Running Information

The running information is the state of things in a running environment.  This state is shared amongst all the controllers in the environment.

### State of things?

This includes the details of each instance of a controlled service (like, how many are running, what are their IP's, and any colab keys... etc).
It also includes custom information that can be shared between services, and load-balancer paths and more.


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

OpenCluster will provide a service where users can have their config encrypted and stored offsite, so that if they have a total data-center outage, they can recover their environment.  There is no ability to also store the data in there, (although we could provide that as a paid option).  Essentially, if their system goes offline, they can spin up a whole new environment in say, Amazon and have everything up and running again.

Note, the off-site stored configuration is just the controller information and wont contain any data maintained outside the controller.   However, off-site backups can be configured and maintained as part of the setup and can be utilised off-site.

## Example #1

A company has some physical equipment in their office building.  Everything is primarily running on this equipment, but the maintainers have configured OpenCluster to use DigitalOcean as a backup.  To accomodate this, they use the S3 services in digital ocean to store data frequently (it is confiugured to provide updates every 15 minutes).   This means that it is using OpenCluster-Data service, and it has the ability to store that data in raw (but compressed) form in the DigitalOcean S3 service.  They have a small node running in DigitalOcean (which only costs $4 a month) to monitor the functionality and to co-ordinate the backup sync.   In the instance of disaster at the office, the DigitalOcean detects this and when triggered will automatically spin up the resources to run everything.  It then restores the backed up data to teh new instances and then triggeres functionality that starts sending traffic to the new DigitalOcean services (rather than the on-prem systems which are now offline).  It can also be set so that certain website know that this has occurred, so it automatically adds a notice to the website to let people know that an issue occurred, but functionality should be available, but if they have problems, they can contact support.

In this note it describes an issue where an unexpected outage occurred, however if the staff at the office noticed a pending issue where they were expecting a fault to occur, they can trigger it to do it in a more controlled fashion so that customers are affected far less.  In this case it can close things down and migrate data.

When the issue in the office is cleared up, they can trigger functionality that moves all the services back to the original site and syncs up all the data that changed during that time.

Because they were running services out of DigitalOcean during this outage, they were still able to provide all their functionality to their clients.  The DigitalOcean services would have accrued some costs, but far less of a problem than if their site was just down for that entire time.

It should be noted that not everything can easily be migrated.  Like all the usual functionality staff have in the office may not be effective to migrate that to digitalocean during this incident.  So only the core fundamentally required services should be migrated.  However, if they determine that the incident will last a longer time, they could still add the remaining functionality to it... but that would be up to them to determine the cost/benefit ratio.  For example, If they use on-prem microsoft services for Email, but want that to run in the cloud, might be a more challenging problem to initiate.  But whatever is used within an organisation should have some level of planned redundancy for these dire situations.

## Example #2

A company is using cloud resources (like Digital Ocean), but also have Amazon configured as a backup.  The controller config is backed up to Amazon, as well as seperate sync services which constantly copy the required data to Amazon.   If Digital Ocean have a significant outage, an action can be triggered that will start everything needed in Amazon and move all the traffic there (DNS changes, etc).

This would not be super easy, but still possible and capable if engineered correctly.  To spin up similar services in Amazon it would likely be more effective to just start up EC2 instances and migrate the functionality just on virtual machines, rather than use teh more diverse cloud services.  However, it is possible to do it either way.





