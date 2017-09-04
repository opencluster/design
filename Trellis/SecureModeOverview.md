# Secure Mode Overview

Opencluster is designed to flexible enough to be suitable and applicable for as many styles of implementation as possible.  However, for sensitive workloads, then being the most ultimately secure environment is a very important goal.  

To this end, OpenCluster will specifically allow the enforcing of a Secure Mode, which includes config and methodology.

To use this feature, the entire pipe-line needs to be very rigidly controlled.   This is very different to other methodologies where opencluster sets up services and then has a hands-off approach.  The services can exist completely independantly of the Opencluster controllers.  

All services in the environment that run on OpenCluster need to be scalable and automatically rolled out.  

All hosts that are in use by opencluster will be _completely_ locked down.
* All user accounts disabled.
* Root account will be given a random password after the opencluster services have started.  Essentially, once running, the box cannot be used by human operators at all.
* IPTables will disable all incoming and ports that are not specifically controlled by opencluster.
* Servers will have a finite lifetime.
* All data stored on the server will use encryption.
* The server will need to maintain a constant connection and heartbeat to the controller.  If it loses connectivity, it is deemed compromised, and all traffic and flows will be discarded, until the server has been wiped and restarted.
* opencluster will monitor EVERY file and process on the host, and will alert if any of them change unexpectedly.
* All connections in and out of the host should be encrypted.
* All connections in and out should be implemented in a scalable and controlled way.
* All sensitive data should be included in protected memory.
* All stored data should be external to the host.
* General cloud-hosted services can be used.
* Host should be setup to detect a paused or snapshotted state (if possible).
* Processes on the host are monitored and blocked.
* Containers are locked down to only what OpenCluster manages.
* Typical Command-line tools on the host are removed.
* Ability to update and install packages removed.   Essentially, only the minimum packages required to run the node will remain.


## Validation

The challenge should be raised where people can run up a vmware host, and deploy opencluster services to it, and if they can manage to compromise the system, or find any useful information, then they can win the pool.

It should be strong enough that banks would feel comfortable running it in the cloud.