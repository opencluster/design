# Trellis Configuration

This document describes how new configuration will be stored and deployed within the environment.

There will be two kinds of config information in a running system.  
1. Source Config
2. Running Information

## Source Config

The source config is the main config that describes how an environment should be configured.  
The source config should be have the following attributes:
* Should be in text files 
* Should be source controllable.
* Should be the same for all environments (test/staging/prod)
* Enviironmental information is not mentioned specifically in the config, such as Hosts.  The actual hosts in the system is not in config, but Running Info.
* Passwords 

## Running Information

The running information is the state of things in a running environment.  This state is shared amongst all the controllers in the environment.

# Config Encoding / Format

YAML is a popular format for configuration.  I'm not sure how good the parsing libraries are, and will experiment, and I'm not sure how good it is when comparing old config to new config, but it is probably the most likely option at this point.

JSON is a good option from a programming point of view.  However, that is not as useful for actually editing the config.

A Hybrid model might be to actually use JSON for feeding the config into the system, but people can create all their config in YAML and then convert it to a JSON format.  This is probably not the best option, would prefer to just stick to YAML if possible.



# Trellis Controller Configuration

The Trellis controller will need some configuration for itself.
This will include:
* where and how to access the cluster configuration.  
* Authentication methods.
* Links to controllers in other environments if they are seperate.
* Logging

# Cluster Configuration.

We should start with a starting config, and everything is loaded after that.

```
Main Config
   Roles
   Containers
   Services
   Storage
   Providers
```




# Loading the Configuration

The controller component that monitors the config can do it in a couple of ways.   It can do the GIT checkout itself, and take note when changes have arrived.  Or it can monitor files.

It will load the entire config into a new dataset and will only apply that dataset if there are no errros.


