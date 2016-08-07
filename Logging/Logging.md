# Logging

Almost all the services that are running, produce logs.   Having the logs available for the entire cluster is an important thing, and often overlooked.  This service should have hooks into the various exiting logging services that log things locally, and push all that into the overall cluster.    So if you have 6 instances of a particular service across your cluster, you can view the logs as if it is a single instance, or drill into a particular instance.
