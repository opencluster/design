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
