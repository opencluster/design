# OpenCluster-Common

A set of libraries that clients can use to communicate with the Opencluster services.  The individual libraries for each service will reference common functionality in the common library.

For example, the client applications that need to talk to a OpenCluster-Data will need to use a library that does all the communication, and that library will use the 'common' library features for the low level functionality.

The Common library will include:
* Config loading
* RISP protocol building.
* Encryption
* Authentication
* Message Routing
