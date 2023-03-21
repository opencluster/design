# Metrics

In addition to [monitoring](Monitoring), it is often useful to gather metrics.  Each service can also describe how to get metrics data.  Sometimes the service can push that data, and sometimes the data needs to be pulled.   Should be very flexible.   Actually hitting a REST-API would be really beneficial.  The metrics gathered could then be graphed or delivered to some other metrics analysing tool.   Some metrics might be 
* Current connections.
* Requests processed per second.
* Bytes received, transferred.
* CPU load.
* Memory used.
* Disk Space used.

The metrics could also be used as a trigger to grow or shrink the environment automatically.
