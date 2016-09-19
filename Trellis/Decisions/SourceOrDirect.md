# QUESTION - What is better - File config, or direct API?

Is it better to have a config which is loaded and parsed, similar to how Puppet is done; or is it better to have an API that causes stuff to happen?  

### File Config? 
This means that changes can be submitted into git repositories and treated as other code.  

### Direct API?
Or is it better to have an API interface, and all changes in the cluster are done through such an interface?   This would mean that all the actual config for the cluster remains internal.   Changes can then be submitted into the non-prod environment, and then integrated into the other environments as sets.  This means that the environment is managed in a seperate fashion.   Even if changes are introduced this way, what mechanism should be used to transfer changesets between environments?  

Additionally, how is the cluster config stored so that everything can be powered off?  In this case, JSON might actually be the best format.

If we do a Direct API, then it is possible to import and export from YAML files.   A seperate peice of code can go through the YAML files which describe the environment, and figures out what changes need to be applied to match it.  I think this way we do get the both worlds.  Doing it the other way might be more clunky.

### Final Conclusion

The underlying mechanism will be a direct API, but we can add a layer that takes source and runs commands based on the changes of that source to get them to match.  This way, different people could even implement their own source and controls, without us dictating what source mechanism should be used.
