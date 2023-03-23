# (Software Defined) Load Balancer

The existing load balancing solutions (HA Proxy, etc) are not entirely suited to a HA changing environment such that OpenCluster is promoting.  You want services to define the traffic they are willing to accept and the load-balancer deliver that traffic to them, changing config without outages, adding and removing large numbers of hosts and services.  This would be a large undertaking as it will certainly be front-facing and require very good security.  The important thing is it will not just be a single isolated load-balancer but an entire tree of load balancers, all fairly well co-ordinated.  

An example: A LoadBalancer (LB) instance is setup on each host to deliver traffic to the services on the host.  A service starts up and informs the LB that it will handle traffic on particular domains and paths.  The LB then needs to inform its upstream LB that it will accept the same traffic.  It will keep going up until it reaches the public facing LB.   This establishes a path for the traffic to flow.

----

To ensure that traffic for sites (and paths) are delivered only to compatible services, need a mechanism to verify that.  When a service starts up, and it connects to the LB, it will need to indicate what site (and paths) it will handle.  However, it will also need to either a unique Identifier for its version, or a version number.  It should also be able to indicate what other versions it is compatible with. 

Lets say we have a service called `catalog`.  Version 1.0.1.  So maybe it will be actually referred to as `catalog-1.0.1`.

* It starts up, and connects to a LoadBalancer pool it has been configured to use.   
* It tells the LB that it handles `example.com/catalog*`
* The LB doesn't have any conflict with that, and depending on its setup either handles it on its own, or passes that config up to another appropriate LB layer.
* all traffic to `example.com/catalog` now gets delivered to the `catalog-1.0.1` service.

So now a little time later, the catalog service is updated.   The developers (or rather those managing the OpenCluster structure) have the ability to change how things are applied, but by default for the services version numbers, it will assume that `service-XXX.YYY.ZZZ` is the normal format.  And the ZZZ fields show changes that are compatible.  For example `service-2.4.1` and `service-2.4.2` are compatible and can exist side-by-side (and will eventually roll out to the same version).  However, `service-2.4.2` and `service-2.5.1` are not compatible, and this means that as the 2.5.1 version is being rolled out, no new traffic will be delivered to it until all the 2.4.2 ones have been stopped.  There can be other things also which have to occur before 2.5.1 is active, so there would need to be something that helps orchestrate that.

So when slight changes occur:

* Service `catalog-1.0.1` is the currect service handling `example.com/catalog*`
* A new version is made available and rolling out.  `catalog-1.0.2`
* On each node that is running the role which contains that service, it will:
  * Become aware that `catalog-1.0.2` is available and will need to be deployed.
  * Stop the `catalog-1.0.1` service.  Which will also cleanly remove it from the LB pool.
  * Install the `catalog-1.0.2` service.
  * `catalog-1.0.2` will connect to the LB, and tell it that it will handle `example.com/catalog*`
  * The LB will determine that `1.0.1` is compatible with `1.0.2` and will start delivering traffic to it.
  
Note also, that the Load-Balancers can be configured to give priority to services.  In some cases it may be better to give priority to services with a higher version.. in others it may be better to give priority to services with lower versions.  This is up to the implementer to determine.  Why?  Because it you have 4 nodes and are deploying a new version, eventually all the older versions will need to shutdown, so would be better to start sending new traffic to nodes with the newer versions.   In other instances, when rolling out new versions, you may want only a little traffic going to them, so you can check logs and metrics to ensure it is working as expected.  Then the implementor can change the settings once they are happy and change the delivery mechanism.   This could also be implemented. 



----

When setting up a host that will be providing HTTPS loadbalancing, need to also ensure that it is secure against the latest attacks.
Might also indicate whether this server is public facing or not.
It can generate a unique set of DH parameters for the site, and include that in the config,  especially for haproxy which uses the param file attached to the end of the certificate PEM file.

----
