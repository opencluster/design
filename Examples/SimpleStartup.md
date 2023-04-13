# Simple Startup Example

This is a situation where someone (or a handfull of people) come up with an idea for an internet-based service and want to implement it.

## The Idea

Joe and Barry are in college and after having discussion they come up with an idea.   
The original purpose of the idea is to build something to help them complete their Computer Science degree.  
They start coding the site, and playing around with it, and even getting it somewhat functional.  
The other students they show it to like the idea, which gets Joe and Barry to start thinking about implementing it for real, and could potentially become a successful busines.

They are now taking it seriously but at the same time dont know if it will be successful. They dont want to waste a lot of time building something that falls flat, but they also want to be prepared in case it takes off rapidly and they need it to grow and expand quickly.

In this situation, they want to focus on building up the functionality and then working on implementing new features.  They dont want to focus on all the infrastructure.

OpenCluster is very useful in this case, because it can be used for simple implementation, but expand and change as needed.

## The Start

Joe and Barry also have a limited budget, so want to limit the amount of money they put into it.  They evaluate various 'cloud' services, but just to get things going, they create a small 'droplet' in DigitalOcean called `Server1` which doesn't have a lot of resources, but only costs $4 a month.

The droplet behaves just like a server, it is a virtual machine.  Joe and Barry are not dealing with physical hardware, but it is pretty much the same.

They install the [OpenCluster Controller](../Trellis/Trellis.md) interactively (it asks the initial required questions) and it is installed.

The Controller is now running on `Server1` and is not really doing much though.   It has a Web Interface, as well as a command-line tool.

Even though the controller is running on Server1, it is currently only maintaining the Core cluster (which is the underlying cluster).

Joe creates a Cluster called "Cluster".
There needs to be at least one zone, so it creates one called 'Default Zone'.
Joe adds `Server1` to the `Default Zone`.

Joe then sets up a Service (by using built in templates that guide him) which will automatically deploy changes from a Github repository, and even able to setup an Action Trigger so that it deploys automatically when changes are committed to a specific Branch.

Joe has already setup DNS for their domain to hit Server1 but since they periodically think about automating some processes, they add the DigitalOcean module to their cluster, so it is available if they ever do use it.  (but realistically, they dont need to yet)

The DigitalOcean module is a seperate module that integrates into OpenCluster so that you can automatically deploy new resources using the DigitalOcean platform.  For example, if their website would typically not get much activity during the weekend or certain times in the day, they can save money by running the website on a single server, but if the website suddenly gets a lot of activity, they can configure it to automatically spin up a new server and balance the load.   Joe and Barry are not in a position where they really care about this yet, but are looking at how it might work.  They also think of the possibility to also using AWS resources, so they also add the AWS module to their controller.   Just adding the module doesn't really do anything (other than it gives them notifications that in order for it to work, they will have to generate an API token and put it in the system).

## The next step in expanding development

After going live, and seeing people using their website, and a lot of motivation for adding new features, they then start to realise that they really need to setup a Development environment to mimic a real service but just for testing before they actually put it out where their customers will experience it.    This became important because Joe and Barry had made changes to their website which actually didn't work as they expected. This resulted in their customers complaining about the issue, so they realised that they need to be a bit more careful to avoid customers dumping them.   Its a tricky part of their startup, and reality is kicking in.

Joe logs into the OpenCluster controller and sets up a new zone called `Development`.   He manually sets up a new cheap small server called `DevServer1` in DigitalOcean and adds it to the `Development` zone. 

The service that he created originally that deploys the code from github (and triggers a script which deploys the code changes) is not tied to a specific zone.  He sets up an action in github that will trigger a deployment of the service to the `DevServer1` server.   He also manually sets up a DNS record pointing to the new dev server (in the future he can set up stuff in OC that can automatically do that for him, but at this scale, not that important)

Now they have it setup so that when they deploy code to a specific branch, it gets deployed automatically to their new Development server.  Quite easy, and doesn't cause too much problems as there is only the 2 of them doing the work.

## Expanding.

Now that Joe and Barry's business has become more and more successful, they are having to start taking things more seriously.  They originally setup a simple database on the webserver they setup, but it needs to be re-structured better.  They are also getting a lot more traffic on their site, which makes them a little worried that it wont be able to handle the load if things kick off higher.  They are taking their new business seriously, and some of their college mates are offering to work there also.  They are nervous about committing their future to this project, and are going to have to make some tough decisions.  They dont want to spend a lot of time worrying about the underlying infrastrucutre, they want to focus on making the bsuiness work and developing new functionality into their websites.

Barry focuses on the development, but Joe will focus on getting the underlying services lined up.  They discuss how to deal with their data storage.  They decide to setup another server for Database storage.  Although there are many many options out there.. and many they think they would actually end up using the in the future, it is not currently beneficial to use those, as they are more expensive at the current scale.   So they setup a small server which costs them only about $5 a month, but at least it is a seperate instance than their webserver.

Joe changes the current 'Default' role to the 'Webserver' role, and adds the existing Server1 to it.  Everything still remains the same.  Nothing really changed there.

Joe then creates a new role called 'Database' and add the new database server to it.  Although it isn't managing much at this point, he sets it up to monitor the database service on the server, to ensure it is running.  He also sets it up to notify him if services are not running but should be.

## Exploding

Till this point, this business venture was only a part-time hobby for Joe and Barry while they were finishing college.  However, after the initial success of the project they are now deciding whether or not to build it up as a real business.  They present their business idea to some family, friends and others they are introduced to, and get some startup funding.   Not a lot of money, but enough to get things going.   Carol, their friend from college also is joining the team.

Now that they are taking this project seriously and want to turn it into a real business, they discuss what they want to accomplish, and the real goals and start planning for it.

While in college, Joe interned at a tech company that was a few years after startup and doing well.  One thing that really caught Joe's attention from a business perspective was that the developers had trouble expanding their original code base.  They kept saying that they struggling to grow and maintain it because it wasn't really designed to handle the scope and growth it was currently experiencing.  They knew they needed to rebuild it, but it was hard to do without disrupting their customer experience.  Joe kept this in mind while starting his business as it seemed to be the biggest thing holding back that company.

Also while in college, Barry interned at a large company that had been around for a while.  Their technology was out-dated, but they were in the process of moving a lot of their functionality to the cloud.  His experience there exposed him to a lot of the various options that are available, but also exposed him to the challenges at a low level.  

Even though they began using OpenCluster from early on, they haven't really used its functionality much.  They continue to maintain their existing system, but based on their exposure to challenges other companies have experienced, they decided that now was an excellent time in their journey to rebuild their project with flexibility and simple expandability in mind.

## New Design

While they maintain their existing services, they now focus on the future.   Joe, Barry and Carol sit down and have numerous discussions and come up with a plan.

Carol will focus on developing mobile apps for their business.  Barry will focus on the main website and the API's (which the mobile apps will need to use).  Joe will focus initially on the underlying infrastructure, but he is also a developer so will also blend in and assist with both Barry and Carol.

While designing the new solution, they keep in mind that currently their business is small and doesn't require a lot of resources, but at any unexpected point, it could explode and they need to increase the underlying resources.  They also may at any point in time discover an opportunity and have to implement something new and integrate it into their services without having to re-design everything.   They want things to be as flexible as possible while also remaining as simple as possible.  They dont want to make things too complicated.

The design also takes into account the multitude of options available.  Multiple cloud services, multiple different languages and products they can use.  They want their system to be flexible so that if they originally start using one cloud service they might discover that it doesn't quite suite them right, and end up switching to a different cloud service.  They also might start using a cheap and simple service, but after their sudden growth they need to move to a different service.

After evaluating their options, their experience with existing software and trying to keep their costs down, they decide to start off with a solution similar to their current implemtnation just utilising the OpenCluster services more.

They start by seperating the functionality with security in mind.  They want to keep their data as secure as possible.  To do this, they want to keep things in different security zones.   So they seperate all their functionality into 3 (well, 4) zones.   Note that even though they are called Zones, the developers ofter do refer to them as Tiers.

* Web Zone - Anything customer facing (exposed to the internet) will be in this zone.  Although not limited at first, they plan to set things up so that anything in the Web Zone must talk through API's to get data.  When requests come in from customers it renders the webpage results and renders it, as well as the static content.
* App Zone - This zone they include in their design, but dont use it straight away.  This is where the majority of the functionality will end up, but at first most of it is in the Web Zone.  They keep this in mind from the start but it takes a bit for them to implement it in a more long-term way.
* Data Zone - This is where the data is kept, and is the most restricted zone.  Even though they start in the cloud, their initial goal is to eventually have this stored on physical servers on their premises.  They dont know at this point exactly what they will end up doing, but this is their initial plan.
* Management Zone - Although in their design plan, they dont initially have a good idea on how it will be implemented finally.

But the question arises... why dont they just put it in the cloud using the resources directly?  While in college and out of curiosity they had used cloud resources and had some experience with it.  At first it seems like it would be cheap and easy, and if desined carefully and implemented with a specific goal in mind, could be cost effective, but the more they dig into it, it starts out costing more.  There may be things much more cost effective in the future when they grow, but to start off they can do it a bit cheaper.  They always keep in mind that they can move parts to cloud services as needed, as they are designing everything to be flexible.   They wont know what will be cost effective until they start implementing solutions.  As long as they keep flexibility in mind, they will keep the options available.

They started off with OpenCluster and decide to continue using it, and actually to take advantage of more of the features available.  It is free after all.

They descide to use the [OpenCluster Load-Balancer](..LoadBalancer/LoadBalancer.md) functionality, as well as the [OpenCluster Data](../Data/Data.md), [OpenCluster Cache](../Cache/Cache.md)

## Network Design (well, sort of)

After they reviewed their goals and how to get there, they decided to start off with [OpenCluster Load-Balancer](../LoadBalancer/LoadBalancer.md).  That is because it gives them a significant amount of flexibility, plus it is generally easy to set-up.  Their situation is fairly common, and so there is already templates that they can start with.

Since they are starting with a minimal amount of resources, they want to use the least amount of cloud-based servers as they can.  They have decided to setup a Data server, and a Web server.  They are keeping in mind that they may want to change things in the future, but this is how it is starting out.

They are actually setting up only 2 cloud-servers to start with, but with the way they setting everything up in OpenCluster, they can expand that fairly easy as needed.

On the webserver they actually have 2 load balancers (LB).  One for handling external traffic, and one for handling internal traffic.

The external LB will be setup as a Proxy-Only Load-Balancer.  The config managing it is all handled within OpenCluster

The internal LB is setup as both a Proxy, as well as a Broker Load-Balancer.  This is for internal traffic, and depending on what it is used for has multiple options.   A proxy means that a connection is made, which indicates a particular end-point, and the LB establishes a suitable connection to that end-point (if it is using HTTP/1.1 then it will continue to use existing connections).   The Broker is where an application wants to connect to something internally, and it askes the LB what it should connect to.   Based on the config, it will either return an IP/Port and potentially other info (such as an access-key, certificate, etc), or it can even say connect to the LB and the LB will actually do teh connection.  The point here, is that the LB can use intelligent configuration to determine where to flow the traffic, without having each app itself having to figure that out.

For example.  There is a back-end service which handles the data.  When it is starting up functionality, it knows how to interact with the LB, so it tells it how it wants apps to connect to it.  In this case, it says that the apps can talk to it directly, but if they cant talk to it directly, then to fall-back to the LB to send traffic through it directly.   

Since both of the LB's are sitting on the Web server, and the web services themselves also on that webserver, it makes it seem more complicated than it should be, but it is fairly easy to setup. The advantage is that this gives almost instant ability to expand their services if under significant load.   For example, if the webserver is being overwhelmed, a small change in OpenCluster can fairly quickly setup an additional webserver and direct traffic to that.   And could even move all web-traffic to that server and just perform its duties as a load-balancer.  Additionally, mutliple load-balancers can be setup with DNS pointing to those multiple servers (multiple A records in DNS).   The load-balancers themselves keep in communications with each other to also keep the load well balanced.  Expanding rapidly requires very little effort because it has already been designed with this expansion in mind... it is just squished into the minimal resources required.

## The Data

The team get together and discuss how they build their new webservice and how it stores and uses data.   When setting up their initial website they  originally setup a simple database, but as they continued developing they realised the structure they had setup was not the best and was causing them troubles in makign changes.  Because of this, they decided to use [OpenCluster Data](../Data/Data.md) which is essentially just a hashmap where you lookup a key (which is made up of a tag, plus the data you are looking for).  Although they haven't yet come up with a completely architected solution, it is the most flexible, and currently fits their ideas.

To setup the Data service, they add it to OpenCluster.  OpenCluster allows you to configure things like this by injecting a config file with all the details, but it also provides a walkthrough interface, so when adding something new, it basically asks questions, and then implements it.   So with this, they setup the Data service in a few minutes.  At first, they set it up on the single webserver that they were putting everything on (for v2).  However, by the time they go-live with the new version, they expect that they would setup a different server for the Data.

While setting it up, they indicate that the Data service should go through the LoadBalancer, but set it as a Preferred Broker.  Although they are only currenly implementing a single instance, they are expecting that as it grows, they may end up having it running on multitple servers.

It should be noted that the Load-Balancer knows how to interact with the Data Service in an efficient way.  It is really only needed when an application is first setting up a connection to the Data service.  They chose to use this with the LoadBalancer because that is what they are implementing also.  However, there are multiple ways to set it up so that the apps know how to connect to the internal services.  One of the other options is a Config service that is part of opencluster.  It works very similar to the load-balancer method, but basically just provides some config  that the app then uses.   They will eventually end up using this as well, but when first setting things up, it wasn't really something they really understood.   Either way works fine though.

One thing to note, the Data service can have multiple tenants.  Also have the ability to have multiple zones inside each 'tenant'.  So they can seperate data as needed.  They are aware of that, but haven't yet decided on how they would set that up.  They know they have a lot of options, and it is fairly flexible so they are not worried.

## The website

Since they are using OpenCluster for a lot of things, they decide to use the OpenCluster Web service to handle the main interface.  It is fairly easy to setup and it is part of the Load-Balancer integration.  They also have the option to use other website services such as apache, nginx, etc.  But they dont have anything specific that they need to use those for.

## The rebuild of the website/application

Now they have most of the infrastructure in place, now they begin work on rebuilding their website.   They originally start out with a Dev server and begin deploying to it.  They have triggers setup so that changes can be pushed out to their dev environment.  However, after reading more into what OpenCluster provides, they learnt about how to setup transitional local instances on their own laptops.  This allowed them to develop their own changes independantly.

Barry starts working on the website.   Most of the functionality their users see will remain the same, its just how it gathers the information and handles it is changing.  There is some new functionality being developed as well, but not too drastic a change.

Since Carol is working on the mobile apps version, she begins working out what API functionality she will need to present information on the mobile apps.  Joe has already setup most of the infrastructure so he is also doing development work and providing the API functionality that Carol will need.  They interact as a team and get a lot done.

When Carol needs some new API she presents it to Joe and Barry.  Initially Barry was implementing the service where his code is accessing the data directly.  But as he sees things progressing with Carol, he realises the that things she needs is pretty much in-line with what he also needs.  So he decides that it would be good to have his application also use the API to get information rather than hitting the data itself.  This came up because at one point they realised that one way they implemented some of the data was inefficient and so wanted to change it.  Joe needed to change the API code to access the data differently, but Barry also had to change his.  But if Barry used the API, he wouldn't have had to change anything.  This also gives an advantage to seperating the web layer from the data later, adding more security.  Ultimately it means they could have seperate servers, and even seperate network accesss.  If someone managed to hack the webserver, they wouldnt have instant access to the data itself.

## The integration of the website/applications and the load-balancer

ALthough it doesn't need to be setup this way, they decided to use the smart interface of the load-balancer with their applications.  The application deployment is handled by the opencluster controller, but it doesn't need to adjust the load-balancer to point to the application node.  Instead the application starts and is given some keys and tokens which allows it to register endpoints with the load-balancer.

To elaborate.  A load-balancer can be configured by the controller in many ways, and these multiple ways can be combined.

These ways are:

* Static Routes (listen on specific IP's/Ports, and handle SNI on certificates) to a list of potential targets.  The viability of the target is determined by a healthcheck.  This normally is just the load-balancer hitting the target and getting a response it is expecting.   The target list can be provided and updated by the controller based on its deployment.  This means that if there are 2 servers created to handle the traffic, but due to load it expands that to 4 servers, the controller tells the load-balancer that there are different endpoints and it adjusts correctly.
* Dynamic Routes (Similar to static routes), however the end-point is determined by the application that registers the endpoint.  Depending on how the token and key is provided, it can tell the load-balancer what to expect and how to deliver it.

Dynamic routes are what is implemented in this case, however it is implemented using what is called Limited Dynamic Routes.   This means that the application cant just register any website, it can only register a path within an already registered website.  The token and key provide this information and limitation.  This is also the preferred solution.   In the off chance that it is required, dynamic routes in load-balancers can be setup so that an application with the appropriate token/key can register anything.

In this case, they basically setup several routes in the load-balancer (and this is maintained and managed by the controller).

* **api** - this is mapped to both api.joebarry.com and joebarry.com/api.  From the application perspective, both those are treated the same.   They set this up because they didn't know which one they preferred and Barry and Carol had opposing opinions and it didn't really matter, so they just set it up so both work.  With the token for the api route, the application can register certain paths, and traffic sent to those paths get sent to the application that registered it.  At first they just use a single application to handle it, but they may decide to implement seperate applications for each api.
* **website** - this is mapped to joebarry.com, www.joebarry.com (but does exclude joebarry.com/api).  This is used by a static content service to deliver the website (and the code on the website hits the API's for certain functionality)
* **mobile** - although this works very similar to the website in that it doesn't actually do anything directly, and most of the functionality uses the **api** route, they implemented this for performance in multiple regions.  To give the customer a better experience in certain locations, they setup a mobile route that actually ends up hitting the api, it just adds some caching in several regions.  This is implemented way futher on, and not used at the beginning.
* **console** - this is for internal use only, and is only visible from specific IP's and client auth certificates.  This is for a console to manage the services...  Their customers will not have access to it however.
* **apptier** - This route was implemented further down the track.  Originally the app that handled the api was considered the app-tier, but it was still public-facing.  After a few years of implementation they actually seperated them for performance and reliability purposes.  This was a result of multiple attempts to DDOS and hack the services.  They originally only needed 2 servers to handle the API calls, but as the progress continued, and the multitude of fake requests (that ended up being blocked) were coming in, they added that additional layer to it.  That way, they can spin up many **api** servers that just filter out the crap traffic, and just pass on the legitimate requests on to the app-tier to handle the real traffic (which ended up only needing the 2 servers to handle the load).  This means that the **apptier** route is internal and not public facing.
* **dbtier** - This route was also added later on.  Originally the applications talked to the data services through direct mapping and config, but at some point as things evolved, it made a little more sense to direct traffic through a load-balancer.   Now they actually dont implement this route in the external facing load-balancer that the normal web-traffic goes through.

So in summary, when the application that handles API requests starts up and is ready to receive traffic, it hits the load-balancer and registers to receive traffic to specific endpoints.  It should be noted that at first, they set it up to send all API traffic to the application.  Later on they refine that to only send specific requests to the application, and even later on they setup a different layer inbetween to filter out false traffic.  The services are so flexible that they had many options along the way and weren't really constrained by the infrastructure.

## Expansion.

Setting everything up in this fashion didn't really block their growth.  As their services grew, and different functionality was implemented, they were able to continue the services.   Their business also grew to the point that they also leased some building space to accomodate the growth of their staff.  They were already paying for network connectivity and the property, and when they evaluated the cost of having their services in the cloud, compared to having it handled on-site, they were shocked to see it would be cheaper to have some of it handled physically.

To do this, they still did maintain some functionality in the cloud as a backup, and also to hanle the external traffic.  When setting up the network connectivity in their building they had seperate infrastructure that was implemented fairly easily, and the equipment wasn't terribly expensive.  They had network traffic for office, and then a seperate network zone for the customer traffic.  They purchased some server equipment that was maintained within the building.  The servers were redundant in case one failed.

### The design worked.

The external load-balancers were kept in the cloud.  They will be used to filter and direct traffic to the headquarters without exposing the corporate IP's directly to the internet.  The data, and apps will be stored locally at the headquarters, however a backup app and some core data will be stored in cloud as a backup in case the connection to headquarters is broken.   When initially setting this up, they basically had an "offline" page setup in the cloud so that people would know that the site was down.  Over time, they introduced functionality that would keep key data in-sync, so that some functionality will be retained in an incident but not everything.

At some point they also keep some physical equipment in a data-center that is on the other side of the city.  Because of the additional redundancy they then use the external data-center as their primary site, with the headquarters equipment for redundancy (and some load-balancing functionality).  Because they began using opencluster at the start, their ability to implement and migrate functionality to different data-centers was actually pretty easy.

They spent a little extra time at the start to make system builds fairly automated (but they didn't go to the trouble to make all of it).

When they got new servers to install in their headquarters the first time, they simply added one of the servers to the core cluster, as were able to migrate the controller functionality to a physical server (as opposed to a virtual server in the cloud).

Also, forgot to mention, when they setup physical servers, they allocated some physical servers for the core application functionality.  These were not sure high-end servers, but had more than enough power to handle pretty much all the application workload.  There was 2 app servers.  There was 2 data servers that had a fair chunk of storage also.  There was 2 controller servers for managing the cluster, and 2 management servers.  The 2 management servers were low in capacity and power, as they were only needed in specific circumstances.  In addition they also had some higher capacity servers that they installed vmware on so that they can create a bunch of virtual servers as needed.

To move the cloud services to the headquarters, it was a fairly easy process as most of the services handle that very thing.  For the data they simply spun up the new database servers, and added them to the cluster.  The functionality was now shared and in sync both in the cloud and at headquarters.  They then just cleanly shutdown the cloud instances through the controller, and it automatically adjusted the load-balancer and the config to point the applications to the new servers.  Although they scheduled this work during their lowest traffic times, this completed without issues and the services continued to function even during the transition.

They did the same thing with the app services.  Simply started up the new app-servers, joined them to the controller and the controller setup everything on the servers automatically and they just started handling traffic.   Then they removed the cloud instances from the cluster.   All simple.

They kept the external load-balancer in the cloud, but the internal one was also moved to the head-quarters.   The 2 management servers also handled the internal load-balancers.   This allowed for internal traffic to remain inside the head-quarters in the internal networks.   This made it easy for some internal managment services to also be load-balanced.

When they transitioned to have an external data-center for redundancy, they also had a similar experience.  They started up the new servers, tagged them and joined them to the cluster, and the cluster just automatically set everything up.  Since the cluster was also then adjusted to have preference to the Datacenter over the headquarters it simply configured things automatically so that they behaved in a redundant fashion.

## Incidents handled.

### Incident #1

At one point an app server had a failure and stopped working.  At the time they only had 2 app servers.  Because of the opencluster configuration and synchronisation between hosts, this did impact some clients active sessions, but because of the way the app was designed, they probably didn't even notice it, other than a timeout delay.  Once the app stopped working, the load-balancer detected this and stopped sending it traffic.   The opencluster monitoring notified staff of this occurance, and were also able to indicate that client functionality remained working, just that redundancy was now reduced.  This failure occured at about 3am.  The on-call staff saw the alert, and determined that everything was ok.  The cluster had been setup so that if there was no redundancy at the headquarters, it will spin up some virtual servers in the cloud to handle any additonal over-loaded traffic.  It did this all automatically based on the event that triggered it.  Because they knew what OpenCluster could do, they had set this up in advance for this very possible scenario.

The next day, the staff contacted the hardware vendor as it is under warranty support.  Depending on the contract, companies may have on-premises support, and in this case they did have that.  The hardware vendor sent out a member to investigate the issue.  They tested the server and determined that it was a failed power supply and motherboard.  They replaced the parts and the server started up again.

Because it still had functional hard-drives it booted up and was ready to continue working.   It knew though, when it connected that the cluster had setup on-cloud redundancy.  So it began the process of syncing up all the data and once it was all ready, it then became functional.  When setting up the cluster, they set it up to do gradual workload.  So the load-balancer began sending it s small percentage of the workload, and as it continued to function as expected, it ended up splitting the load evenly.  Once it was running for a specific length of time without issues, it then started the automatic process of reducing the workload from the on-cloud instance.  Once the load was finished, it then automatically shutdown the on-cloud instances.

Even though starting up the on-cloud servers for a couple of days did cost a little money, it was still only a tiny amount, but provided redundancy and functionality.   After this incident, they also decided that since they also had vmware servers on site that had a fair amount of capacity, rather than pushing the redundancy to the cloud, they could have used the vmware resources instead.  So they set it up to be able to create a VM under that situation, but still maintained their cloud redundancy as a third option.

### Incident #2

After setting up their primary services at the datacenter, everything ran smoothly for some time.  Then another incident occurred.   This incident involved one of the data servers.  Since the data services are redundant, they actually do have one as the Primary, and unfortunately in this case that was the server that failed.  However, it didn't impact significantly.

Because they set things up to have redundancy on-site, as well as redundancy to the headquarters, they essentially had 4 servers, put the primary one failed.  When something is marked as primary it just means it is the one that co-ordinates synchronisation.  All 4 servers are kept in sync, and the load is actually distrubuted across them (although since most of the traffic was handled at the datacenter, the 2 servers at the DC were the ones that handled most of the load).  Since they now only had one active server at the datacenter, the other server then took over and became Primary.  However, due to the load at the time, it began having to offload some of that work to the servers at the headquarters.  Because of the way configured in OpenCluster it was able to do this automatically, but also kept things in sync also.  It began delivering traffic to both the DC and the headquarters, and the app-servers in each location talked by preference to the Data server closest to it.  This resulted in almost no client impact to the incident.

As previouly handled, the hardware vendor went on-site and determined that the storage controller failed, and although the hard-drives on the server were ok, the data would likely be compromised.  So once the hardware was fixed, it was rebuilt with the standard physical OS install, and then was added to the cluster.  The controller then installed everything that needed to be installed, and then it triggered the data synchronisation.  This took some time, but once synced up, then everything began moving back to how it was.   The load-balancer started sending less traffic to the headquarters as the DC equipment began handling the load fine.

Although they do accept that incidents can be unpredictable, in this case it was planned for in advance, and worked mostly as expected.   During this incident though, they did notice some performance issues, and after looking into it, discovered that some config was not done correctly.  Even though the services were functioning, there was some delays in responce because of this.  Once this became known and they tweaked the config (which was related to timing ratios) it all then began working as expected.  This raised questions about possibly doing some yearly Disaster testing to find these kinds of things.

### Incident #3 (the big one)

A big storm was hitting the city and due to some poor circumstance it highly impacted the functionality.  Due to the storm, the power in the datacenter was cutoff completely.  They normally have backup power, but multiple lightning strikes caused it to fail.   This meant all the functionality then quickly took over at the headquarters, which then was functioning just fine.  Because of the DC outage, the HQ systems took over but also began spinning up cloud functionality for redundancy.  This was good because a few hours after the DC power blew, the same storm also blew the power at the headquarters.   When the DC zone was cutoff it transitioned all functionality to HQ, but also set up limited redundancy in the cloud.  The corporation had determined that if they had a serious incident, they wanted certain functionality of their business to remain, but they didn't need all of it.  If they ran everything out of the cloud, it would be quite expensive.  They could do it, but didn't need to.   So when the DC was out, it started setting up the cloud, and giving it the data it needed to remain functional.  When the HQ died also, the redundancy in the cloud took over, and continued to service the customers.

It took a few days to get power back in the HQ, and when it did, it then took over functionality.  Because some limited functionality was running in the cloud, it had to sync the data correctly.  And since not everything can be predicted, they did encounter some issues that required some modification to some code and config, but that was pretty straight forward once it was identified.  For example, normally when new customers come on-board, it triggers some other functionality that was not made redundant.  Because the services were unavailable during that outage, they had to modify some things to trigger them again.  Once this was learned the code was updated to handle it better in the future.

Some time later, the DC was functional again, and everything then ended up pretty much back to normal.

There were things learned.  One of the services, because of the way it was designed it took a lot longer than it should have to recover.  Once this was determined and understood some time was taken to modify the service to handle things better.

Additionally, because of the extra time it took to get power back to both the HQ and the Datacenter, there were discussions on how to spin all functionality up in the cloud.  They discovered that this was both tricky and surprisingly easy because of the way their systems were structured.  The remaining second-tier services relied on additonal storage and resources.  The active data was in the HQ and was powered down.  At first they also did not have access to the building because of the damage from the storm.  They did however have off-site (and interim data actually in the cloud) backups.  They realised that it would only take a little bit of effort to just tell the OpenCluster controller to start up resources in the cloud.  It would take a little bit to get the backup data in these cloud services, but entirely possible to bring everything back up, all back in the cloud.

The tricky part was that the backups were from teh previous day, and there was some information that would have been lost.  When investigating they discovered that most of the missing data was also in the other data stored which had already been synced, but not in the same format as what those services needed.  So they would have some decisions to make... do they revert back to the data that was backed up previously, or do a hybrid combination.    Fortunately power and access to HQ was back before they felt the need to seriously brind all the second-tier services back online.

They also discovered that they had a way to obtain the data from external backup services but would have some challenges bringing that back into functionality in this situation.  So they did begin to evaluate how to handle thigns a bit better in the future.  This took some discussion and planning but ultimately was achievable.  The primary method was to identify the large historical information that the 2nd-tier services used but didn't need, and seperate them from the essential data it needed.  By syncing that essential information to the backup services (and the cloud redundancy if needed), they could also have the 2nd-tier services spin up.. but they could also control the amount of resources, and extra services that would be provided.   In other words, they could spin things up, but tune it down.

## The extra stuff along the way

During this progress through their company services being developed, they used various resources and tools, and were of the mind to try something new but always have a backout plan.. This means that if they bringing new functionality online, and some cloud-based services might provide a useful cost-effective solution, they attempt to implement it with the idea that if it doesn't work as expected, or if it ends up costing more than expected, then they have the ability to implement a different solution.  They are not expected to have an alternative solution already planned out to great detail, however it is expected that they at least have a high level plan as an alternative.  For example, they were thinking of introducing some extra functionality to the mobile app.  Rather than coding it all themselves, they found a 3rd-party product that did most of what they wanted.  They decided to try out that 3rd-party product, but kept in mind during the design that if it didn't work out as expected, or some time in the future they want to add some different functionality that 3rd-party service dont want to do, then they can build up their own replacement.  They keep that in mind while implementing it.  The key takeout is that they dont want to be cemented into using a particular thing.

Another slight example, was that they were originally using some crunchy file syncing crons that copied files from a primary server to a secondary server.  They instead started looking at a cloud-based tool to host the files instead.  They implemented this, but kept in mind that they had some alternatives to follow if they didn't end up liking it.  They did keep using the cloud storage for some time, but ultimately did end up using a different file syncing mechanism (Gluster, and then later used an OpenCluster block service).








