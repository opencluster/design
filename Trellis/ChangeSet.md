# ChangeSet

A ChangeSet is a set of commands that make up an atomic change.

The ChangeSet can be applied as a unit to one environment, and then promoted up through the other environments.  This allows a single complete change to be deployed to a Testing environment first, and then applied to Staging, then production.

Most changes in Trellis would be descriptions about Services, containers, load-balancing directives and dependencies, etc.

The ChangeSet does not include changes to the data services that run inside OpenCluster itself.

# Enforcing Change-Sets

When an environment is setup, it can be specified that when changes are made, they are done in change-sets.
NOTE: When making changes to the system, it can prompt to add the change to a change-set and asks for a name describing it.  It can then add the change to a section on teh screen showing the change-sets.   The change-set can be applied, and it will push it to the Dev environment (or whatever ones accept change-sets directly).   The user making the change may only have rights to make changes but not apply them.  Another person may have the authority to approve them.  Or it could be set so that even if a person has authority to apply changes, they cannot apply their own changes.  Another admin who has the rights might get a message asking them to review the change and apply.

If they want the change to be applied to the other environments, they can do a Pull Request (or mark a change as ready to promote) which will inform an admin for the next environment that there are changes to apply.

# Environment Specific Config

Each environment could have configuration that is specific to just that environemnt.  For example, the Test environment might have config that points to a Test version of the database.

When changesets are applied that reference the config, it will verify that the required config is available for that environment before applying.

# Variable Substitution.

When making changes to the environment, and a user is putting in details, if they use {{some.value}} and it will get substituted from the Config for the environment it is being applied to.

# Command Line

Changes can be applied using the command-line in two ways.

## From a file.

Changes can be written in a file.  The file is specified, along with a name for the change, and possibly a command to deploy it to a particular environment.

## Iterative.

When the opencluster command-line tool is started, a command can be entered that opens a change-set.
A session might look like....

```
opencluster
> change-set create --name="Add Web Service"
Change> service create --name="Blog Tracker"
Change> role add-service --role="Web" --service="Blog Tracker"
Change> close-change
>
> change-set append --name="Add Web Service"
Change> service modify --name="Blog Tracker" --system-service="blogtracker.service" --autostart="yes" 
Change> close-change
>
> change-set commit --name="Add Web Service" --deploy-env="Dev"
Change "Add Web Service" applied to "Dev" Environment.
>
```

In this session, we created a change-set, added some functionality to it, and then added some more functionality to it.  Then we committed the change and at the same time deployed it to the Dev environment.

Then, when that change has been deployed successfully to the Dev environment, we can promote it to be used for Test.
Depending on the authorisation that the user has that is making the request, it could be applied automatically, or it can remain pending until someone who has authorization applies it.


### Example of a change where the user does not have authorisation to deploy to Test.

```
opencluster
> change-set promote --name="Add Web Service" --env="Test"
Not authorised to deploy.  Pull request sent.  Change is pending deployment to "Test" environment.
>
```

### Now another person logs in who does, and after applying it.. promote it to the next environment.

```
opencluster
Pending changes to apply to "Test" environment.
 - Add Web Service
> change-set commit --name="Add Web Service" --env="Test"
"Add Web Service" has been applied to "Test" environment.
>
> change-set promote --name="Add Web Service" --env="Staging"
Not authorised to deploy.  Pull request sent.  Change is pending deployment to "Staging" environment.
>

```




