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

After going live, and seeing people using their website, and a lot of motivation for adding new features, they then start to realise that they really need to setup a Development environment to mimic a real service, but just for testing before they actually put it out where their customers will experience it.    This became important because Joe and Barry had made changes to their website, which actually didn't work as they expected, which actually caused their customers to complain, so they realised that they need to be a bit more careful to avoid customers dumping them.   Its a tricky part of their startup, and reality is kicking in.

Joe logs into the OpenCluster controller and sets up a new zone called `Development`.   He manually sets up a new cheap small server called `DevServer1` in DigitalOcean and adds it to the `Development` zone. 

The service that he created originally that deploys the code from github (and triggers a script which deploys the code changes) is not tied to a specific zone.  He sets up an action in github that will trigger a deployment of the service to the `DevServer1` server.   He also manually sets up a DNS record pointing to the new dev server (in the future he can set up stuff in OC that can automatically do that for him, but at this scale, not that important)

Now they have it setup so that when they deploy code to a specific branch, it gets deployed automatically to their new Development server.  Quite easy, and doesn't cause too much problems as there is only the 2 of them doing the work.

## Expanding.

Now that Joe and Barry's business is become more and more successful, they are having to start taking things more seriously.  They originally setup a simple database on the webserver they setup, but it needs to be re-structured better.  They are also getting a lot more traffic on their site, which makes them a little worried that it wont be able to handle the load if things kick off higher.  They are taking their new business seriously, and some of their college mates are offering to work there also.  They are nervous about committing their future to this project, and are going to have to make some tough decisions.  They dont want to spend a lot of time worrying about the underlying infrastrucutre, they want to focus on making the bsuiness work, and developing new functionality into their websites.

Barry focuses on the development, but Joe will focus on getting the underlying services lined up.  They discuss how to deal with their data storage.  They decide to setup another server for Database storage.  Although there are many many options out there.. and many they think they would actually end up using the in the future, it is not currently beneficial to use those, as they are more expensive at the current scale.   So they setup a small server which costs them only about $5 a month, but at least it is a seperate instance than their webserver.

Joe changes the current 'Default' role to the 'Webserver' role, and adds the existing Server1 to it.  Everything still remains the same.  Nothing really changed there.

Joe then creates a new role called 'Database' and add the new database server to it.  Although it isn't managing much at this point, he sets it up to monitor the database service on the server, to ensure it is running.  He also sets it up to notify him if services are not running, but should be.
