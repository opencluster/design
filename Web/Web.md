# Opencluster-Web

OpenCluster-Web is a dynamic, flexible and reliable web front-end to fascilite service delivery.  Provides teh same abilities as Nginx, Apache, etc... but is managed by OpenCluster and provides delivery to microservices as well as static content.   It can be highly integrated with the [Opencluster-Loadbalancer](../LoadBalancer/LoadBalancer.web) service, although it can be integrated on its own, and also provide some load-balancing services directly.  The Web side, and the LoadBalancer could end up being the same product.

## Simplicity

The Web side can be rather complex in some cases, but the agent can be small in size and deployed everywhere.

For example:

> A company is providing a website for its customers.  They have a Web agent on some servers in the cloud, which deals with static content and delivers traffic to backend nodes for some paths.  The web agent is also installed on teh servers handling the backend services and the front-end web knows how to talk to them directly.   The web agent on the backend then can process the request directly (using whatever language they use to integrate).


## Load-Balancing

A key aspect is the ability to expand and reduce the footprint of the services.   It can also be setup to automatically allocate resources globally to improve performance and reduce latancy.

For example:

> A company is providing a website for its customers.   Most of their customers are in Australia so the company has hosting in Australia.  They do have some customers in other countries though.  Their load-balancer and website are setup to be flexible and by default running in Australia.  When a customer from another country accesses the website, it spins up resources in that location.  Once services in the other location are functional, the traffic from those locations will begin to transition to that closer environment.   Some functionality might still have to end up in the Australia location (the active source), there can be other data cached in the remote location.   In other words, 90% of the activity might be able to be handled in the other location, and only 10% has to end up back at the active source.  This can improve the performance and delivery of the service to the customer.
> The remaining question is "How do we get the client to direct traffic to the closest node?"  This will require geo-location DNS, which could be done with some cloud services, or can even handle DNS directly (which can also be used for delivery based on other performance metrics also).

The load-balancing will generally be handled by the [Opencluster-Loadbalancer](../LoadBalancer/LoadBalancer.web) service, but the web service will need to interact with it to tell it what services it needs to provide.

## App interaction

The way any solution is delivered is flexible and dynamic.  Apps that start up can tell the web services what URL's it can handle, and then traffic that arrives will then be delivered to those services.   The Web service will therefore be mostly acting like a reverse-proxy.

## Static content

It can also be presented with static files.  This can also be flexible.  It very possible to simply keep track of website content with a git repository.  The web will then present that content to the allocated URL's and locations.

For example:

> A company has a github repo for the static content.  They also have specific paths redirected to a application.  It can be setup so that the traffic is delivered to the app as HTTP (like a reverse-proxy), or could be setup to use RQ message queues that then get processed by the application.   The static and dynamic parts of the website can therefore be intermingled.

## Web Application Delivery

We want to enable a solution where websites can be deployed and managed easily.
