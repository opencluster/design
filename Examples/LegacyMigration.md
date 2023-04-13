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

After the success of the first mini project, it was desired to try it out for a larger and more complex peice of work.  This larger peice of work was to migrate a bunch of expensive cloud-based services back to physical systems in their datacenters.  After investigating their various options, they still didn't know what would be the most effective, but they decided to try a 2-phase approach.   OpenCluster has the ability to integrate services in the cloud, and so the first phase was to integrate the affected cloud-services with OpenCluster.  This was OK for some of the services which were ElasticCompute services (basically virtual machines in the cloud).  They setup configuration in OpenCluster to build and maintain EC2 instances which have the same configuration and settings as the existing systems, and then were able to link the existing systems to OpenCluster.  They were able to set it up so that OpenCluster at first didn't make any changes, it just reporting anything that it thinks it needs to change to match its configuration.  They were able to set it up in the Dev and Test environments, and also had it rebuild the instances.  This turned out to work ok.

While setting up the configuration, they did it in the recommended way where most of the config didn't matter if it was served in the cloud or on other hardware.  Once the initial setup was complete, and rolled out to OpenCluster managed services in the cloud, it was then able to be used to rebuild anywhere (possibly even different cloud services).

The other big key factor was how the existing services recieve their traffic.  There is cloud-based routing, as well as local routing.  They had a very complicated network infrastructure.  Before implementing these things, a decision was made where anything managed by OpenCluster will utilise the OpenCluster Load-Balancing service.  So things existing outside of opencluster were using legacy routing, but anything brought into OpenCluster would just have that traffic delivered to OpenCluster and it managed it once it arrived there (including outbound traffic in reverse).  This was a little complicated to setup at first, as there were a lot of things to juggle, but after they got some experience in it, they were able to get progress with it.

To implement this, they did setup several Load-Balancer layers (with the goal that once a lot of migration completed, some of those layer could then be combined.

For example, one of the services that was migrating, was a website that was maintained and developed by this company, but it was to provide a service for a different company.  Customers accessed this service directly through the internet, and it was hosted in the cloud, and the traffic was delivered to the servers that hosted the website.  Once they migrated that service to be managed by OpenCluster, rather than delivering the customer traffic directly to the web service in the cloud, it instead delivered it to the OpenCluster Load-Balancer which then delivered it to wherever the service now existed (either in teh cloud or on physical hardware, OpenCluster knew where it was).
