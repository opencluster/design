## Monitoring.

The OpenCluster system can provide a monitoring interface to support personal.   If you dont want to use the built-in monitoring but instead feed events into your existing monitoring infrastructure, then the events themselves can be fed directly to most monitoring systems, or scripts can be triggered for ultimate flexibility on handling events.  In most cases, a combination of these solutions can be put in place.

Trellis is an orchestration system that sets up environments and gets it running.    Once it is running, we need to know that everything is working as expected. 
Therefore, we need to be able to report the status and health of the various components within the system.
Some of the Trellis functionality would be accomplished by adding trellis services to your environment.  
Each service added to the system should have additional config that describes how to monitor the services.  Some default options can be added.

The Monitoring system can also enable triggers that change certain parameters.  When these parameters are changed, it can cause the environment to change.


