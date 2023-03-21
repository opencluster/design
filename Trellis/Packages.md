# Packages

Packages are a defined set of files that are used to install, update, verify and uninstall components to a node or system.   

## Diversity
The package layout has not yet been defined, but are meant to be flexible enough to be diverse (for example, the same 'package' could be deployed to Windows, Linux, Redhat, etc).

To achieve this, the package itself is mostly meta-data, which references artifacts in a different storage system.  When a package is introduced and applied to a cluster, it will need to verify that the required objects it references are available (and valid).  This will generally result in a number of package files being generated (that match the various OS layers).  
Some packages will only apply to certains Operating Systems, and some will span multiple ones, so there needs to be a way to interpret and compile it.

## External Sources and Layers
THere will likely be a common package store available to all users, and also will likely be  

Processing
The initial processing of a package would 


Requirements
Packages will likely require some things.  Eg, some data.  Or some pre-requisit packages to be installed, and even conflicts.

There can be some interaction with other OS related repositories (like redhat rpm's), so that we dont need to create an OC package for every system package.
Packages can use some other applied services to interact with things directly.  

There can be some 'Handlers' for dealing with external or local resources.  These handlers can be also similar to other modules.   If a package references a handler then it is a requirement that the handler is available (but can be over-ridden)

``` 
proposed example
----------------

package-name: "ipverta"
version: 1.31
requires: gotabi:>2.291, %{yum-repo}(golang)
conflicts: gotabi:<1.12
```
