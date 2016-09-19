# ChangeSet

A ChangeSet is a set of commands that make up an atomic change.

The ChangeSet can be applied as a unit to one environment, and then promoted up through the other environments.  This allows a single complete change to be deployed to a Testing environment first, and then applied to Staging, then production.

Most changes in Trellis would be descriptions about Services, containers, load-balancing directives and dependencies, etc.

The ChangeSet does not include changes to the data services that run inside OpenCluster itself.

# Environment Specific Config

Each environment could have configuration that is specific to just that environemnt.  For example, the Test environment might have config that points to a Test version of the database.

When changesets are applied that reference the config, it will verify that the required config is available for that environment before applying.

