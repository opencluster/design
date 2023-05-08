# Legacy Migration Example

I company that had been around for decades ended up migrating a portion of their services to OpenCluster.

## The company history

The company was originally an idea that was brought up as part of a bigger company.  They decided to spin it off as a seperate company.  Over the years it grew, and parts of it split again into seperate companies and was invested in by other companies who then took over majority ownership, and eventually an even larger company took complete ownership of it, and embedded it into their corporation.  This lasted for a little over a decade when it was split off and sold to another completely different company that were wanting to embed themselves more deeper in the marketplace that it was involved in.

Over these many years and transitions things changed along the way.   For their technical infrastructure, it started out as some basic physical hardware, and along the journey they implemented high-end networking infrastructure (probably at a much higher level than was needed).   Their technical resources grew considerably over the years and they even migrated through various Datacenters.

Over time, they did implement some mechanisms to help co-ordinate the services, but it was never done in a complete fashion, and maintaining everything became a challenge.

## Goal to transition to the cloud

When it was bought by the final company mentioned above, there was a lot of retrospective review (but mostly from a business perspective, not a tech perspective), and with a lot of high hope in mind, they wanted to eventually move everything to the cloud, and utilise as much 3rd-party resources as possible.  Many who managed things did not beleive that this was a good way forward, and would end up costing the company way more in teh cloud than it did currently.

## Cloud expectations not met.

They did indeed begin setting some thigns up in the cloud, but it was surprising to the business how difficult that was, and how expensive it was.  They did understand that it would be expensive, but over time as things got tuned up and transittioned further those costs would drop.  However, that was not the case.  There was indeed some things that were cost-effective in teh cloud, but not everything.

## Transition back to a hybrid service.

Once reality kicked in, they began looking at things from a more realistic perspective.  At some point one of the engineers had been playing around with OpenCluster, and suggested it would be a good fit for the company and a hybrid reality.   Although the company itself was not immediately keen on that (in actuality, they were struggling to commit to a particular strategy), they did allow the engineer to setup OpenCluster and use it for a particluar project that was being deployed.

The engineer had done a course to be familiar with OpenCluster, and even had some conversations with some experts who had experience in it, so he was able to implement a solution that would both suit his needs for that specific project, as well as be in place to provide additonal services over time.

## The architectural layout.

The company had a fairly complex infrastructure.   Because it was a company that produced and provided services to other companies, and they were in an industry that had very strict government regulations, they had to be quite careful about some things.

They had multiple environments.

* **development** - the development envrionment was where initial code changes were deployed constantly by developers.  It was quite a messy environment.
* **testing** - the testing envrionment was where versions of their code that was considered to be testable were deployed and a seperate team of Quality Assurance testers performed testing functionality on the services.
* **Service Integration Testing (SIT)** - Since the company integrated its services with multple other companies, the SIT environment had links to the testing environments with the other companies.  This allowed for testing of all the functionality.  It also had much more realiable data to use for testing (dev and test were a bit messy and only had the minimum data needed as things went along).   Testing had to be completed in this SIT environment before it could be progressed to the production envrionment.
* **production** - The production environment was the real service provided to customers.
* **infrastructure** - this environment was also considered production but was to provide the basic functionality that the other environments needed.  It provided the services that employees needed, and services that needed to oversight the other environments.

## The initial setup.

The company still had physical servers at the Datacenters,  but they also had a number of vmware clusters.  The engineer originally setup the Controllers as Virtual Machines (VM's).  They had 2 datacenters, so setup one at each site.   Originally the engineer setup just a single cluster in the infrastructure zone, but kept in mind that they could certainly split it so that they have a Production layer, and a non-production layer.   He also considered that in the future they may end up using a physical server for the controller rather than a virtual server... but that could also be implemented fairly easily in the future.

## The first project

The first project involved replacing some existing servers and functionality with some new servers on a newer OS.  The previously used Puppet and were consiering Ansible as a replacement, but due to slow progress in that space, this particular engineer wanted to try OpenCluster.  There was a lot of opposition by other staff members in that space, but that was mostly due to personality problems where they didn't like it when other people implemented things they were not experienced in, and wanting to be the boss and owner of.  The engineer implemented it anyway, and it turned out to be quite effective.

The engineer also made a good decision in taking into consideration their entire infrastructure rather than just implementing a solution suitable only for that one particular peice of work.

The engineer setup the controllers, and the various zones and templates.  The new servers that were being installed to replace legacy older ones were configured in OpenCluster and allowed it to even deploy them in VMWare.  This took a little effort as there was some standards that needed to be applied.  But all the normal stuff that their Puppet system was deploying could also be done by OpenCluster.  To keep things somewhat non argumentative, the engineer set it up so that the new servers still configured and used Puppet for its continued maintenance, so he set it up so that they didn't conflict, but as more systems are built and maintained by OpenCluster, the puppet functionality can be migrated over peice by peice.

The new servers were setup in all the environments, and the engineer was even able to demonstrate the extra capabilities of OpenCluster.  There was some dispute over its effectiveness, so the engineer proved this point, by having the target servers removed from VMware, and OpenCluster performed the various warnings and then implemented replacement servers almost instantly and set them up.

There was also another peice of work on the table to build some new servers as well, and he demonstrated that it could be configured as needed, and implemented automatically.  There was also some content that needed to be deployed to the servers.  He demonstrated that the files could be stored in a central location (in this case it was their internal github instance).  When the developers made changes, and it was approved and committed to a specific branch, OpenCluster deployed it automatically, and even within specific time windows that were allowed for deployment.  This also proved the flexibility of it.  The Dev and Test zones were allowed to be deployed at any time, the UAT ones were deployed automatically unless a specific flag was set for performance testing and QA testing were being deployed to ensure changes weren't being deployed.  And the Production environment had to be specifically authorised and triggered manually (although it could be configured to deploy at specific times).

This demonstration proved that OpenCluster provided the services that were sorely needed within the organisation.

## The second project

After the success of the first mini project, it was desired to try it out for a larger and more complex peice of work.  This larger peice of work was to migrate a bunch of expensive cloud-based services back to physical systems in their datacenters.  After investigating their various options, they still didn't know what would be the most effective, but they decided to try a 2-phase approach.   OpenCluster has the ability to integrate services in the cloud, and so the first phase was to integrate the affected cloud-services with OpenCluster.  This was OK for some of the services which were ElasticCompute services (basically virtual machines in the cloud).  They setup configuration in OpenCluster to build and maintain EC2 instances which have the same configuration and settings as the existing systems, and then were able to link the existing systems to OpenCluster.  They were able to set it up so that OpenCluster at first didn't make any changes, it just reporting anything that it thinks it needs to change to match its configuration.  They were able to set it up in the Dev and Test environments, and also had it rebuild the instances.  This turned out to work ok, and wasn't too overwhelming.

While setting up the configuration, they did it in the recommended way where most of the config worked the same regardless of if it was served in the cloud or on other hardware.  Once the initial setup was complete, and rolled out to OpenCluster managed services in the cloud, it was then able to be used to rebuild anywhere (possibly even different cloud services).

The other big key factor was how the existing services recieve their traffic.  There is cloud-based routing, as well as local routing.  They had a very complicated network infrastructure.  Before implementing these things, a decision was made where anything managed by OpenCluster will utilise the OpenCluster Load-Balancing service.  So things existing outside of opencluster were using legacy routing, but anything brought into OpenCluster would just have that traffic delivered to OpenCluster and it managed it once it arrived there (including outbound traffic in reverse).  This was a little complicated to setup at first, as there were a lot of things to juggle, but after they got some experience in it, they were able to get progress with it.

To implement this, they did setup several Load-Balancer layers (with the goal that once a lot of migration completed, some of those layer could then be combined.

For example, one of the services that was migrating was a website that was maintained and developed by this company, but it was to provide a service for a different company.  Customers accessed this service directly through the internet, and it was hosted in the cloud, and the traffic was delivered to the servers that hosted the website.  Once they migrated that service to be managed by OpenCluster, rather than delivering the customer traffic directly to the web service in the cloud, it instead delivered it to the OpenCluster Load-Balancer which then delivered it to wherever the service now existed (either in the cloud or on physical hardware, OpenCluster knew where it was).

This company did not use a lot of the OpenCluster services at the start, they mostly used it to configure and maintain their environment.  However, one of the services being migrated was a central database for its data, but the developers were also needing to do some updates to this service.  While looking at their options, using the database was a bit annoying and they were wanting to be able to store the data a little easier.   They considered using the OpenCluster Data service.

Some generic servers (virtual machines on-prem) were setup which were for the generic app deployment in OpenCluster.  Originally the devs had the Data store on those servers, however later on it was decided to put them in a database network (more restricted).  Each microservice would have a zone inside the OpenCluster-Data instances.  Since it was setup as part of OpenCluster, with the Load-Balancer delivering traffic, and the Data service keeping the data in sync, it was setup in a way where new instances could be created on-demand, and also redundancy was setup to move things to the cloud as needed.

## Migration of phase 1

After the first two peices of work, and the successfulness of it there was a further push to move much more stuff.  This resulted in a lot of little micro-projects.  It also resulted in some minor changes to the OpenCluster infrastructure.  The load-balacers were modified to use node-injectors (even though the load-balancers existed in a specific VLAN, some of the services that were using the load-balancers wanted as little latency as possible, so a virtual network interface was added to each target VLAN.  This reduced network routing a little.  There was also tweaking in configuration as the administrators became more familiar with the functionality to provide a more consistant and reliable routing.

One thing that was passed on to all the related teams was some training on preferred mechanisms.  For example, there was a lot of redundancy within the infrastructure, however the way it was configured traffic was moving between datacenters.  This was because the load-balancers just balanced the traffic between end-nodes, regardless of which was closest.  The recommended configuration was not used at the start, but as they became more aware of the details, they did change it to have prefferred routes.  At first this seemed difficult and confusing, but once they had experienced it, it was actually quite easy.  But the main part they realised, was that each node didn't really know the state of things.  There were services that were reporting that the were ok to receive traffic, however, the other services they connected to were not ok.

The main thing to note from this, is the difference between the old mechanism and the new mechanism implemented.  The old mechanism just had 2 results of the healthcheck.   GOOD and FAIL.  The load-balancer checked the endpoint and determined if it was good to send traffic to, or not.   For some of the paths, they set up seperate paths on each site.  But that made it more challenging for the services making the requests.   How does a service on site 1 hit a service on site 1 prefferred?  Very challenging.  Especially when they had cross-site VLAN's (meaning that the servers existed in different data centers but was on a network that spread across both data centers).  This meant the load-balancer could not tell what site the request was coming from, as the API requests were just hitting an F5 endpoint that could exist on either data center.


For example.  If we have a service called SAM, and we have an instance in datacenter 1 and 2, so lets call them instances SAM1 and SAM2.   And there is another instance called PRO... again in both DC's, so PRO1 and PRO2.   PRO gets data provided by the load-balancer.  SAM is connected to the LB, but asks for it to be Brokered for higher performance.  This means that when SAM starts up, it tells the load-balacer its details and it asks where possible for services that want to connect to it, to be told to connect directly if they can (but use the LB as a backup if they cant).  When a service wants to use the LB and knows that it is setup this way, it asks the LB who it can connect to and the LB provides a list.  In this case PRO service talks to SAM.  It asks the LB for connection info, and the LB provides network info to it, and then PRO connects directly.

It also is aware of the site differences and prioritises talking to the site it is on.  This means PRO1 wants to talk to SAM1, but can talk to SAM2 if SAM1 is not available.   However, if SAM1 isn't available, we actually want as much traffic as can to go to PRO2, because it can talk to SAM2 and is much closer and quicker.

This can be setup in a couple of ways.  It can be setup so that each service is aware of its location and provides info back so that things can be determined.   Alternatively by default, it can be setup to deliver traffic to the service that responds the quickest.   If the PRO1 traffic response is quicker to respond to the requests, then more traffic will be set to it.  Things level out.   However, this can be tricky to determine also and only really works if the secured data is parsed (meaning un-encrypted, and evaluated and then re-encrypted when talkign to the end-nodes).

They tried out multiple methods.  They setup the healthcheck on SAM to indicate if it is in full access state (meaning it was connected to all the preferred services).  The same was set for the PRO service.  If SAM1 was reporting that it was working 100% fine, then PRO1 was as well.  However, if SAM1 was down, PRO1 could talk to SAM2, but indicated to the load-balancer that it shouldn't be preferred, and so teh LB delivered traffic to PRO1.

This seems tricky but most of this functionality is built into the services they were already using and by default should work this way, but the initial setup implemented bypassed this, and they basically just ended up reverting to the default setup.

