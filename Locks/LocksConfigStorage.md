# Locks Config Storage

Each Locks server will have the potential of running permenantly (as opposed to an ephemeral instance).   This means that some information needs to be stored and used upon startup. 

This config includes:
* Connection Information
* Authentication Information (for Server to join the cluster)

NOTE: That there is also the possibility of using a remote config service.  This means that the local instance is configured with information on how to get the config information.  The remote service will determine if it should supply the config to this server.   The server could be pre-built with a one-time configuration which means that when it connects to the remote config service, the initial config is removed, and the remote config determines if the returned config is kept or not.


In normal uses though, the config is retained on the server.


Question:
	Should we store the config in a human readable format such as JSON or YAML, or should we use something that is more controlled like RISP.  If we use RISP, then we should include tools that can create the config, and make modifications to it.

Answer:
	Neither.  The initial connection config should be stored in typical CONFIG format.  The main configuration details should not change from the cluster itself.  The main config tells the service how to run.   Once the server is connected to the network, it will get additional sync'd details telling it what to do.   Depending on how the cluster itself is configured, it may store connection details in another file.  This will allow the service to know what cluster members existed during it's last connection.

# Initial Authentication

When the service is initially configured to connect to the cluster, it gets information from the cluster on the method that should be used to connect in the future.   Those methods are presented from the cluster itself, and can change in the future.

# Startup

When the service starts, it needs to know how to connect to the cluster.  In some cases, the cluster exists at specific addresses that are continuously known.  
It can use the following methods to know how to connect to the cluster initially.   

1. DNS
2. It can connect to another source, to find out addresses. (such as OpenCluster-Data, Reddis, etc)
3. It remembers the cluster member details and tries to connect to those.

All of those options can be included in the configuration.   In other words, it could be configured to try the known cluster members, if that fails, try DNS, and if that fails, try a remote listing service (that is used by calling a script).

# Server Specific config.

When a new node is configured, it can be given information about it's location, etc.   Once connected to the cluster, this information then becomes managed by the cluster itself.   

