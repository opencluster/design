# Applying Changes

When changes are applied to an environment they are done through an API.
The API will apply the change.  If there is multiple environments (ie, Development, Testing, Staging, Production), then it will ensure that changes are applied to only the Development environment.
When each environment is setup, it is set to allow changes to be applied or merged.

Changes can be applied individually, or as part of a change set.

Some changes will require a [Processor](Processor.md) to run them

## Example Commands

Commands will be similar in structure to the docker command. 

```
# Create a new network.
opencluster network create --name="Internet Facing"

# Add a host, changing the credentials and setting up a local firewall that blocks off all in/out traffic except for the ports that OpenCluster needs.
opencluster host add --name="ny546" --orgunit="newyork.site1" --host="201.1.2.3" --network="Internet Facing" --username="root" --password="fajrac87jdf" --resecure

# Create a Role
opencluster role create --name="Web Server"

# Set a Host Param in a Role (This means all hosts that this role is applied to, will have this param.)
# In this case, when a host reboots, we do not want services to start automatically.
opencluster role set-host-param --param="StartServicesOnBoot" --value="no"

# add a role to a host.
opencluster host add-role --host="ny546" --role="Web Server"

# create a new environment
opencluster environment create --name="Development"  --short-name="dev"

opencluster environment create --name="Testing" --short-name="test" --restrict-change="yes" --merge-from="dev"

opencluster environment create --name="Staging" --short-name="staging" --restrict-change="yes" 
opencluster environment modify --name="Staging" --merge-from="test"

opencluster environment create --name="Production" --short-name="prod" --merge-from="staging"
opencluster environment modify --name="Production" --restrict-change="super"

# Add a host to an environment.  
opencluster environment add-host --env="Production" --host="ny546"

# Add a network to an environment.  Networks and Environments must match for a Host.  Networks are for restrictions, if none is assigned, then it doesn't restrict.
opencluster environment add-network --env="Testing" --network="Test Network"


```

# Interactive Command-line

The 'targetcli' tool for managing iSCSI targets has a faily similar interface to what I was intending for OpenCluster, and it seems to work quite well for that.



## Command Groups

* environment
	* ls | list
	* create
	* modify
	* delete
* change-set
	* ls | list
	* create
	* discard
	* commit
	* promote
* role
	* ls | list
	* create
	* delete
* host
	* ls | list
	* create
	* add-role
* service
	* ls | list
	* create
	* modify
	* delete
* container
	* ls | list
	* create
	* modify
	* delete
* webservice
	* ls | list
	* create
	* modify
* storage
	* ls | list
	* create
	* modify
* network
	* ls | list
	* create
	* modify
* provider
	* ls | list
	* create
	* modify
* [data](EnvironmentData.md)
	* ls | list
	* create
	* modify
	* delete



When applying a bunch of changes as part of a change-set, you give it a name, which should describe what it is you are deploying.  The name can be anything.  Some organisations might want to put a Change Number as an identifier, others are more human interactive.

All the commands that you apply to the tool, can then be placed in a file. 
It will not actually commit the changes unless every single command can complete successfully.
If any command fails, then all will fail and no change should have been made.

Keep in mind that changes also get applied to only certain environments, and then those individual changes can get pulled in to the other environments.  
The chain of changes can be restricted, so that: 
* Change sets can only be applied to Dev.
* Test can only pull in changes from Dev.
* Staging can only pull in changes from Test.
* Production can only pull in changes from Staging.
* Production can have changes applied directly, but require an over-ride authorization.

This is all configurable though, some people might not want to restrict the change chain in this way, and allow changes to happen anywhere in whatever fashion.


