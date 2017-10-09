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
