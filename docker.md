# OpenCluster in Docker


The world of containers is becoming very big right now and there is a lot of momentum.

We need to provide a clustered container solutions right out of the box, especially for the Data services when not being orchestrated through [OpenCluster Trellis](Trellis/Trellis.md).

# Data Services

This would be useful where a client container could be setup where it expects a corresponding opencluster node container on the same host.

So... say an implementation has 3 servers which contain the webapp.  They decide they need a fourth and simply fire up a fourth server, which automatically is given the webapp role and it fires up the various app containers and a cluster container.

The customer can setup an opencluster image which has their own security keys in it.   They can use the default one if they want, but then it is using default keys.

When they fire up an opencluster instance, it will automatically have its own config adjusted, and will join the cluster and accept connections.

There are some customisations they can make, such as putting config in the container.  In that case, they can simply make and store their own customer container based on the official one.

Each 'host' that has containers on it that would need to use the clustered data, would have an opencluster container running also.  All local requests would go to that local instance.   The instance itself will maintain the data and connections to other instances.  Therefore, the cluster should be able to rebalance quickly, and not affect much while it is rebalancing.

Additionally, cluster nodes in containers might have some restrictive memory constraints.  Considering a lot of services are going towards a micro-services architecture (especially with docker), where you have a lot of small virtual machines that can spawn in dynamically.  Those small machines tend to have low CPU and RAM resources.  Also, we dont want the cluster to consume all the resources in the environment, because those resources will be needed for the actual services that use the data.

The cluster server node therefore should have a mode to use the least amount of memory resources as possible.  It will store the data in disk files.  It will also attempt to cache data that it pulls from other nodes.  When it does this it would put an event on that data so that it is alerted if the data changes so it can clear it's cache.

Need to do some performance tests to see what method works out to be the most efficient, especially considering that the operating system also caches disk activity.  So using mmap might actually be very fast if the data is already cached.   Also need to see how it handles swapping out that memory.  Is it acceptable to just mmap all the files and let the OS handle the swapping in and out of memory?  That wouldn't be good if the system has real restrictions and cannot use lots of swap space.

Also, how does it handle it when we have lots of large files memory mapped into memory?  More in size than we have memory and swap space?  Surely when we memmap and it swaps out, we could have an application that has allocated more memory than we have in the system.   That will be interesting.

Things to resolve.
1. Best practices in regards to config over private IP's and public IP's.
2. Should an app, and an opencluster node be combined into a single container?  No.
3. Docker containers maintained over multiple data centers, using a Connector component.

