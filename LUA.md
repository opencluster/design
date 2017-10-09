# LUA

The OpenCluster systems are very customisable and modular.   A common method in systems like this is to use plugins and modules.   OpenCluster will also allow modules to be integrated.  The main method to do this is to install module that are LUA programs.

LUA is an embeddable language.  This means that the LUA script can reference functionality that is specific to OpenCluster and therefore the LUA module can integrate itself within the internal workings of OpenCluster.

A lot of the web-based GUI will be driven by LUA behind the scenes.   The majority of API calls will simply be running LUA scripts with the supplied parameters.  

Modifications to the Cluster will be essentially a LUA script that performs some action.

This does mean that the majority of the cluster is code-driven rather than config-driven.

The implementation needs to be done in a POC before a mature design can be completed.