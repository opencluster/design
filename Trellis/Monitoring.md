# Monitoring.

The OpenCluster system can provide a monitoring interface to support personal.   If you dont want to use the built-in monitoring but instead feed events into your existing monitoring infrastructure, then the events themselves can be fed directly to most monitoring systems, or scripts can be triggered for ultimate flexibility on handling events.  In most cases, a combination of these solutions can be put in place.

Trellis is an orchestration system that sets up environments and gets it running.    Once it is running, we need to know that everything is working as expected. 
Therefore, we need to be able to report the status and health of the various components within the system.
Some of the Trellis functionality would be accomplished by adding trellis services to your environment.  
Each service added to the system should have additional config that describes how to monitor the services.  Some default options can be added.

The Monitoring system can also enable triggers that change certain parameters.  When these parameters are changed, it can cause the environment to change.

For example, with [Metrics](Metrics.md) it can be configured that once server processing capacity reaches a certain point (could be CPU usage, memory usage, number of incoming requests per second, etc)... it can trigger another instance (vm in a cluster spread over different hardware, or a new instance in teh cloud, or even a new instance with higher capacity) to be spun up.   Once the load reduces, it can trigger to revert back to the lower cost systems.

Similar can be done with monitoring.  When someone is detected that is a concern... like a service has crashed, it can trigger an action that spins up a new system.  In many cases there would be an active system and a passive system (or load-balanced), and the passive then takes over as the main.  For diagnostic purposes it may also be prudent to keep the crashed system around, so that developers and support staff can investigate the issue.  It is very dependant on the overall design of the system in the best way to do this.  It is highly flexible.


