# OpenCluster Trellis 

OpenCluster *Trellis* is the internal codename for the overall orchestration and deployment system that for the most part is the central core of OpenCluster.  The other products suppliment and support Trellis.   When people say 'OpenCluster' they would normally be talking about the core orchestration system.  The other products are called by their product name... ie OpenCluster-Locks... which is a seperate product that can be used without any of the other OpenCluster components, including Trellis.

It is essentially a framework for managed docker/hybrid environments. It enables you to more easily expand or shrink.  Simply adding functionality to the overview, and behind the scenes all the plumbing is added.

The main idea, is that you can simply add a new site to the system (and tell it what docker containers are required to run it), and Trellis should do all the rest of the work, possibly including generating LetsEncrypt certificates, haproxy config (linked in with any other existing load balancing config).
Services can come with a hints file which indicates key parts of the service (ie, default paths and containers), etc.

## Running OpenCluster
* [Getting Started](GettingStarted.md)
* [Configuring](Configuring.md)

## Design Goals
* [Security](Security.md)
* [Licensing](Licensing.md)

## Components and Features
* [Monitoring](Monitoring.md)
* [Metrics](Metrics.md)
* [Secure files](SecureFiles.md)
* [Non-Secure files](Files.md)
* [Load Balancing](LoadBalancing.md)




### An example.
A role has some specific modes.   Some roles dictate that certain files are deployed to the host, and some containers started up, and some iptables set, etc.  All servers started up with this type of role, should provide identicle services.
Roles can be combined as long as they dont conflict.
An example. 
you might have a "webserver" role.
You add a haproxy service to that role, and indicate that every host (that has that role) should have one (and only one) instance running.
You add several website containers to that role, and again, indicate that every host should have one (or more).  The haproxy config may be static, so should include config to send traffic to all those containers if they exist.
You might add some files that are put on the host (some files are static, but some can be generated from code).

You might have a "web load balancer" role, which handles the load balancing outside of the host (role) to deliver traffic.  
You might externally have DNS pointing traffic to two different IP's.
You might have two hosts in this role, and they have config that points them to the respective hosts that they will direct traffic to.  You might have two datacenters or geo-regions (each host can have some meta-data that is used for this).
To make it simple, you might actually have different haproxy configs, and it depends on the meta-data which config is used.

You might have another role called "database".
You might add some specific storage for this database with particular replication methods or whatever.  This might mean that only one host is active at a time on storage.
You add a database container like mysql or something that uses that specific storage.  You link it so that whatever host has the active storage, is also the host that must run the database container.  You specify that there should only be one instance per role (not host).  If you have 4 hosts that have this role, then they are co-ordinated amonst themselves.  Eg, the database storage is replicated to each.

You might also have a "database load balancer" role.
This will be similar to the 'web load balancer' role, but remember that the database may be active/passive, so all traffic should go to the primary.  This means that all servers in that role may have the same config, and only have active connections to the active host.

You might also have another "generic compute" role.
This one has generic services added to it... some might be that there should only be one, and it will be deployed to whichever host has the least load (or whatever the policy is).


All the services that have dependencies of other services (even in other roles) can receive the information about that service (as it existed at the time that the system was last balanced).
This means that if the controller stops running, the services continue to run as they are.
If everything is setup correctly, and reduntantly, it should even continue to function if there are failures.
Whenever changes are made (ie, new services are added, or config is changed), then the system will try to balance to fit everything into the new running state.
It should be able to determine what changes it needs to make before applying also.
To re-balance, it may need to move some services, or it may indicate that some hosts dont have the capacity to comply.
Each service can specify memory, cpu and disk requirements, and possibly even network load requirements.
You may have some spare hosts in your system that dont have roles assigned to them.   The system might recommend adding the roles to those servers.
Inversely, the system might determine that some hosts are no longer required, and can be de-allocated.

If you add a new service to a generic compute role, and all the hosts with that role have small containers on them, so that there is not enough capacity to add this new service that has high requirements, it might recommend moving those containers to other hosts compressing them.  So therefore, the service config should indicate if it is safe to stop and relocate the container.   You might have a container that runs a job periodically, and it is safe to do this.  You might have other containers that need to be carefully moved (co-ordinated with an outage, etc).  The system should try to move the relocatable containers to fit the new service that is being added, otherwise it will simply indicate that there is not enough capacity for that service, and administrators will need to to manually resolve it by moving services or adding new hosts.




### Updating Components
When you have your environment up and running, but want to update some config, files or services within the environment, you would make changes.  For example, if you have a service called 'Main Website' and you update it from webimage:7.1 to webimage:7.2, then you make that change and commit it to a particular environment.  If there are some specific instructions within the Service for updating, then it will follow them, otherwise it will simply do the default and stop one at a time all the containers for that service and start them again using the new container image.  
Those changes were committed to a particular environment.  When you want to take those changes and apply them to another environment, you do a merge and it will merge the changes.



### Environments
Most large organisations like to make changes and test them in a different environment before applying the same changes to production.    This can be accomplished in Trellis by setting up multiple environments.  The environments will take all the roles and applies them to the hosts within that environment.  Changes being made are queued up as a number of requests that can be bundled together and applied to the other environments.  There should be some workflow shortcuts to enable this quickly.   For example, each change being made, could be tagged to a particular version number, and then only apply those changes to the other environment.

Some additional information about the services in the role would be required to indicate how to safely migrate the change into a running system.   

### Seperate Environments
Some organisations might prefer for the non-prod environments to be manages by a seperate console than the production environment.  In this case, we could setup some certificates in the prod environment so that it can access and pull change-sets from the non-prod environment.   The prod environment, could also be used to manage the non-prod environment, but not the other way around.  If we have isolated environments, then the master can also be sync'd up to the 'master' on the non-prod.  That would be the branch that new features and releases are branched from.

### Environment Chains
Can setup the system so that whatever changes go in Prod, should have already been applied to UAT, etc... and down the chain.  It could potentially be over-ruled, because there may be some prod only changes that need to be applied, but it should at least warn the user a couple of times before applying it.   

### Hosts
Hosts can be added to Roles.  The actual host themselves do not have to be exclusive or dedicated to Trellis, but obviously anything on there should not conflict with it.  In other words, it is perfectly acceptable to have trellis hosts that have additional things installed on them.  Quite possibly monitoring services, or management and maintenance tools, logging services, etc.

### Roles
Describes the role of a group of one or more hosts.  The hosts dont have to be defined for a role to exist.  Any hosts that are given the Role, will have the behaviours that are dictated by the role.

### Containers (services)
Normally Docker containers that are associated with a Role.
Service details can be imported (the imported config may indicate a preferred role with a generic name, or a specific role with  a specified ID).  The implementor might overrule some of the security considerations, but by default, imported config that should be automatically associated with particular roles, should be signed by a certificate that has been approved to add those services to a particular role.
Note that Role's should not specify environments.   You should not have a 'Staging' and 'Production' role.  Those are environments.   Roles should be logically considered to be within the scope of environments but identical.

### Storage
Different types of Storage can be attached to a role.  Some can be very stand-alone, and some can provide some HA and redundancy capabilities.  The storage options chosen will depend on the resources available.   It can range from a container that has an exported volume that automatically sync's the contents to another identical container, to backend redundant storage like SAN's with site sync, etc.

### Firewall
Each service in a role can indicate what connectivity it needs, and the firewall (iptables, etc) on the host will be automatically adjusted for the services that end up running on it.  This can be tricky to implement in a changing environment but should be doable.   An example of this... a service on host A requires to connect to a database service on host B.  The firewall on host A will be adjusted to allow the connection out from host A to host B, and the firewall on host B will be adjusted to allow the connection in from host A to host B.    The firewall will be set to represent the potential connectivity.    If a service is pinned to a particular host, then the rules could be present on just that host, but if a service could exist on any host in the role, then the rule should be set on all hosts in the role.  This follows the idea that the config for every host within a role should be fairly identical.  If we can have the firewall config adjusted on each host to allow for the actual changing environment (eg, a DB only existing on one host out of many)

Logistics
---------
Trellis itself will need some services to function.    Trellis is required for making changes to the environment, and monitoring the health of the environment.  If all the trellis services are stopped, the main environments should continue to run.
These services will all be in a Trellis Role that will need to be attached to one or more hosts. 
It Trellis Role will contain: 
* storage for the files that it copies to the hosts.  
* the web-bases consoles for admin and monitoring
* the API server
* a load-balancer service to direct requests.

When Trellis is first setup, it will create and maintain all the services that the user activates.
Ideally you should have a single Trellis installation that manages all environments.  The actual running environments would actually be fairly isolated, so you can definately have different hardware running each environment and they are fairly seperate.  Your primary Trellis interfaces (which would normally be Production) can provide a seamless and authenticated control of the other environments.    Each environment would have a seperate Trellis role applied to a host.
The primary trellis environment should be the production environment.  This is because the changes being merged from environment to environment are pulled, rather than pushed.  Dev environment cannot push the change to production.  The changes are 'published' on Dev, and then Production can pull those published changes from Dev.  (changes are tagged, and the entire tag is published). 


### Different Host capabilities.
When a host is added to the environment, trellis will connect to it and gather information about it (and continue to do so periodically, or when a re-scan is requested).   It will look for CPU, RAM and DISK capabilities, check which version of docker and related tools are installed, what OS it is, what access the account supplied has (sudo access, etc).  It will also try to determine what networking is available (internet facing, private VLAN, etc).


### Single-Host environments
It should be possible to put multiple roles on a single host.  This is a valid expansion path.  Start on a single host, and then as needs outgrow that host, add more hosts and spread the load or spread the roles.   Additionally the hosts dont need to be identical between hosts (unless the config dictates it).  For example, your staging environment might be a single host, and your production environment might be two hosts.    To accomodate this, the entire environment should be able to run on a single host with adequate resources for it.


### Seperating services from Roles
It is very likely that as a system starts becoming more complicated, that services that exist in one Role might need to be decoupled, and moved to a different role.  An example of this might be where a service was originally run in the General Compute role, but as it becomes more fleshed out, it might get moved into its own Role.   We should provide a fairly seemless process where that can happen.  Since services in different Roles can be linked, it should be quite easy to automate this process and would be a useful feature.

### Low Cost bare-bones hosts
Trellis should be able to function from the home-brew setups through low-cost bare-bones servers (DigitalOcean, etc), through Amazon and physical datacenters.   It shouldn't force people to a high granularity if they dont need it.     It may be difficult to provide a completely agnostic config for all potentially deployable environments and use the capabilities that are available.  For example, amazon provide load-balancing services.   It may be difficult to put LB config in trellis and it use amazon-LB if it is available and a different LB strategy if it is not.  

### Storing the Data
JSON is one of the better serialization formats.   We could also use YAML, which is already very popular, especially used a lot in Puppet.


### Storing Environmental Config
The trellis environment will need to store some data within each environment for things to work.  This could include service accounts, or end-point URL's, etc....  When adding a service, that might also need some config... can have the config file be processed, or a script run to produce it.  Some config in that script, might need to come from elsewhere... (the script or template should be the same for all environments, but the config can change the final result).  When adding the config value, can specify if it is required, whether it has a default value that is likely applicable for all environments, etc.  This way, when the change is merged into the other environments, then it will ensure that the required config is set.  It could list all the applicable config, and indicate the default values, and ask for the ones that it doesn't have.  

### Re-scanning
When config is changed, Trellis will need to re-evaluate everything to see if everything is still compliant.   For example, if you previously set that there should be a minimum of 4 webservers, but make a change that minimum must now be 8, it will see that it is not running enough webservers and will start 4 more.  Alternatively, if you had gone from 4 to 2, it would know that it can drop down to 2 depending on the triggers.

If trellis is set to maintain the system constantly, it will check for anything that needs adjusting frequently.  However, it is possible to setup Trellis so that it only does this when triggered.

### Feature Branches
A common feature with source control is to use Feature Branches.  Especially when specific branches are used to do automatic deployments.  The Trellis system can work on a very similar protocol.  Although I am looking a little more to using tools (command-line, or interactive web or gui based) to make changes, rather than pushing code, the underlying change is still a change-set that can be tracked using GIT for example.  Since we can have different environments (dev/test/prod, etc), we could also benefit from feature branches.  Make a set of changes that are grouped into a single change-set, then push that to the dev environment, and then test, and then finally to production.   This would be better than simply taking whatever is in Dev, and push it to Test, and Prod.  Instead we are pushing specific changes.

The system could track these change sets, to indicate which ones have not yet been deployed, and can deploy them.  It can be more centralized and looser than typical git commits though.

### Release branches
Along with feature branches, it is also a good practice to use release cycles and include multiple feature-branches in a release.  

### Windows Hosts
Some organisations may have multiple Operating Systems in their environment, so we should have the capability to Communicate and Deploy to Windows servers as well as Linux.   Might have to install an Agent on the Windows host that is used to interact with it.  This could end up like Puppet where it had lots of support for linux, and limited support for windows.  The complications of a Windows environment may mean that this does get treated as a lower priority.

Now that Docker is supported more and more on Windows and Mac, we might have more options for these systems.










