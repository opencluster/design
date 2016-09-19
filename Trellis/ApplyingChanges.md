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

```


