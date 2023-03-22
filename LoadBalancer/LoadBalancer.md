# (Software Defined) Load Balancer

The existing load balancing solutions (HA Proxy, etc) are not entirely suited to a HA changing environment such that OpenCluster is promoting.  You want services to define the traffic they are willing to accept and the load-balancer deliver that traffic to them, changing config without outages, adding and removing large numbers of hosts and services.  This would be a large undertaking as it will certainly be front-facing and require very good security.  The important thing is it will not just be a single isolated load-balancer but an entire tree of load balancers, all fairly well co-ordinated.  

An example: A LoadBalancer (LB) instance is setup on each host to deliver traffic to the services on the host.  A service starts up and informs the LB that it will handle traffic on particular domains and paths.  The LB then needs to inform its upstream LB that it will accept the same traffic.  It will keep going up until it reaches the public facing LB.   This establishes a path for the traffic to flow.

----

When setting up a host that will be providing HTTPS loadbalancing, need to also ensure that it is secure against the latest attacks.
Might also indicate whether this server is public facing or not.
It can generate a unique set of DH parameters for the site, and include that in the config,  especially for haproxy which uses the param file attached to the end of the certificate PEM file.

----
