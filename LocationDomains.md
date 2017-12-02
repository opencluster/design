# Location Domains

Location Domains are used to provide detail to the servers and clients around the location of the services.  In some organisations this does not matter, but some organisations have seperated data centers, or regional zones, and they want as little traffic as possible to go over the links between those data centers.  This means that clients connect to servers in that zone, and only cross over to the other zones when there is no other choice.   The actual structure of the Location Domain is entirely up to the implementation.   

As an example, lets say that a fairly large organisation has a presence in the US, Australia, and Europe.   In the US, they have two datacenters, one on the west coast, one on the east coast.  In Australia they have two datacenters, one in Perth, and one in Sydney.  In Europe, they have just one datacenter.  In the datacenter in Perth, they have two sets of racks, one set in DataHall1, and one set in another part of the building DataHall4.

You might then have multiple Location Domains.
* au.perth.dh1
* au.perth.dh4
* au.sydney
* us.east
* us.west
* eu

Keep in mind, that this structure is completely up to you to define in your implementation.  There are no hard-configured parts of this structure.


You may also want to designate the provider of the services.  For example, if you use Amazon Web Services for compute, as well as DigitalOcean, and Azure.  You could have your Location Domain be represented by provider as the highest level of the structure.
* amazon.sydney
* amazon.east
* amazon.west
* azure.sydney
* azure.east
* do.singapore


Or alternatively, you may actually want the location to be of higher importance.

* au.sydney.amazon
* au.sydney.azure
* us.east.amazon
* us.east.azure
* asia.singapore.do

The Location Domain distinction is completely up to you.  The Locks services will try and use this information to improve responsiveness, and reduce bandwidth.   It is important to remember that the Cluster will be treated as a single entity, even if it is spread all over the globe.   

If you want to have distinct difference between locations, then you would implement multiple clusters (and probably co-ordinate those clusters with another cluster that co-ordinates changes between locations).

# Client co-ordination

Most of the OpenCluster products are built to be provide a clustered service.   To assist clients in connecting to appropriate nodes, location-domains are used.   This means that if a client knows where it is, it can connect to the cluster, and then re-connect to the most appropriate server.

If the client has a location domain for "au.perth.dh4.rack3.srv5", it will try to connect to nodes in the following order.
* "au.perth.dh4.rack3.srv5"
* "au.perth.dh4.rack3.srv5.*"
* "au.perth.dh4.rack3"
* "au.perth.dh4.rack3.*"
* "au.perth.dh4"
* "au.perth.dh4.*"
* "au.perth"
* "au.perth.*"
* "au"
* "au.*"
* "*"

Essentially this means that it has the opportunity to connect to a local service, but if there are none local, it will attempt to connect to others anyway.  The location domain config also has the option to specify a weighting.   This means that if you have "au.perth.dh1" and "au.perth.dh4", but you give dh4 a higher weighting, any client that is outside of those domains, but is trying to connect to anything in "au.perth", since dh4 has a higher weighting, servers in that domain will be at the top of the list, and clients will connect to them.  Note that each node can have a configuration that specifies how much load it can have (and this can be dynamic too), so being at the top of the list, doesn't necessarily mean that all traffic will go to it.   Servers that have the same weighting, might be configured to spread the load evenly.

