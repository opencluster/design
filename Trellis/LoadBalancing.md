# Load Balancing

A very common communication method for a micro-service architecture is by accessing the different components using REST API's and similar.

Having the environment abstracted through containers and compute nodes complicates this significantly as the requests need to be directed to the actual running component.   We can ensure that Containers are linked to the appropriate nodes.
[Docker Swarm](https://docs.docker.com/engine/swarm/) adds functionality that may make this process a little simpler.

Trellis needs to use the options available to ensure that a component that needs to talk to another component through a load-balancer is done automatically.

