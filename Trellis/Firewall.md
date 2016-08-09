# Firewall

Each service in a role can indicate what connectivity it needs, and the firewall (iptables, etc) on the host will be automatically adjusted for the services that end up running on it.

This can be tricky to implement in a changing environment but should be doable.   

### An example of this:
* A service on host A requires to connect to a database service on host B.
* The firewall on host A will be adjusted to allow the connection out from host A to host B.
* The firewall on host B will be adjusted to allow the connection in from host A to host B.
* The firewall will be set to represent the potential connectivity.

If a service is pinned to a particular host, then the rules could be present on just that host, but if a service could exist on any host in the role, then the rule should be set on all hosts in the role.  This follows the idea that the config for every host within a role should be fairly identical.  If we can have the firewall config adjusted on each host to allow for the actual changing environment (eg, a DB only existing on one host out of many), then the firewall should be closed by default, and opened if the service ends up on that host.
