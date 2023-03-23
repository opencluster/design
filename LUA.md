# LUA

The OpenCluster systems are very customisable and modular.   A common method in systems like this is to use plugins and modules.   OpenCluster will also allow modules to be integrated.  The main method to do this is to install module that are LUA programs.

LUA is an embeddable language.  This means that the LUA script can reference functionality that is specific to OpenCluster and therefore the LUA module can integrate itself within the internal workings of OpenCluster.

A lot of the web-based GUI will be driven by LUA behind the scenes.   The majority of API calls will simply be running LUA scripts with the supplied parameters.  

Modifications to the Cluster will be essentially driven by a LUA script that performs some action.

This does mean that the majority of the cluster is code-driven rather than config-driven. 

The implementation needs to be done in a POC before a mature design can be completed.

> Note:
> This should be noted that the LUA scripting is for the internal interaction *inside* the OpenCluster customization, but *not* for the actual Cluster implementation itself.
> Pretty much anything that can be customizable on how the Cluster actually determines what order or method to process the interactions will be LUA scripts which controls how a certain functionality behaves.
> The actual stuff that is controlled and managed by the OpenCluster Cluster, will not be LUA (at least not the internal LUA layer)
