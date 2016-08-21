# Zones

Most products should have a concept of isolated Zones.   These should be implemented in a way that allows the environment to be used as a Turnkey product.  It should be possible for a service provider to have an environment that can be used by multiple customers independantly of each other.  To the point that each customer would not be aware of the others on the same system.

This also simplifies some of the authentication models.   Some simple services can essentially provide READ/WRITE/ADMIN roles across the whole Zone, rather than more complicated model within the zone. 