# Environment Specific Data

Each environment (Test, Staging, Prod, etc) will likely have some config that will remain the same, and config that will be different.
Trellis will maintain that data.

When data is set in an environment it can be used by the config and change-sets that are applied to it.

So this means that the same actual change-set can be applied to different environments, yet stil apply environment specific data.
This can include accounts and passwords for services.  These will likely be different in non-prod and prod environments.

When a change-set is being applied to an environment, it will be parsed in a very specific way.

There is non-environment specific data.  This will be used as a default if the data has not been specified for that environment.  This is refered to as 'Default' data.

When applying params in code, you wrap the full env path in double-brackets.

eg...

```
{{apptier.main.password}}
```

In this case, if we are applying the change to the Staging environment, it will look in there for the param called 'apptier.main.password'.  If it cannot find it there, it will see if it can find it in 'Default'.

In some cases, in the change-set itself, you want to ensure that it does NOT use a default value, but must require one in the Dataset for that specific environment.
In that case, you add a ! (exclamation point) to the start of the param name.

eg...

```
{{!apptier.main.password}}
```

This will cause the change-set to fail if that parameter does not exist in the Data for the environment it is being applied to.

Parameters are likely to be IP addresses, DNS names, service accounts, usernames, passwords, etc.

## Securing and Encrypting data.

We may have to include some capacity to encrypt the passwords and other secure data in a way that allows them to be referenced and used without being visible.  This is less necessary in this system compared to say, Puppet, where the code/config is pushed to a GIT repository and parsed.  In that case, you definately dont want your passwords and other environment data committed forever in source code.

Since environment data is entered into OpenCluster directly and not through code, then this isn't a problem.

