# Roles

Describes the role of a group of one or more hosts.  The hosts dont have to be defined for a role to exist.  Any hosts that are given the Role, will have the behaviours that are dictated by the role.

* Roles have specific modes.
* Most other constructs 


### An example.
A __Role__ has some specific modes.   Some roles dictate that certain files are deployed to the host, and some containers started up, and some iptables set, etc.  All servers started up with this type of role, should provide identicle services.

Roles can be combined as long as they dont conflict.
An example. 
you might have a "webserver" role.
You add a haproxy service to that role, and indicate that every host (that has that role) should have one (and only one) instance running.
You add several website containers to that role, and again, indicate that every host should have one (or more).  The haproxy config may be static, so should include config to send traffic to all those containers if they exist.
You might add some files that are put on the host (some files are static, but some can be generated from code).

You might have a "web load balancer" role, which handles the load balancing outside of the host (role) to deliver traffic.  
You might externally have DNS pointing traffic to two different IP's.
You might have two hosts in this role, and they have config that points them to the respective hosts that they will direct traffic to.  You might have two datacenters or geo-regions (each host can have some meta-data that is used for this).
To make it simple, you might actually have different haproxy configs, and it depends on the meta-data which config is used.

You might have another role called "database".
You might add some specific storage for this database with particular replication methods or whatever.  This might mean that only one host is active at a time on storage.
You add a database container like mysql or something that uses that specific storage.  You link it so that whatever host has the active storage, is also the host that must run the database container.  You specify that there should only be one instance per role (not host).  If you have 4 hosts that have this role, then they are co-ordinated amonst themselves.  Eg, the database storage is replicated to each.

You might also have a "database load balancer" role.
This will be similar to the 'web load balancer' role, but remember that the database may be active/passive, so all traffic should go to the primary.  This means that all servers in that role may have the same config, and only have active connections to the active host.

You might also have another "generic compute" role.
This one has generic services added to it... some might be that there should only be one, and it will be deployed to whichever host has the least load (or whatever the policy is).
