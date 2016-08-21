# Trellis Controller

The Trellis Controller is the central component that keeps all the Trellis parts running when config is changed, or the environment changes.

## Static Environments.

In many cases it is preferable that the environment is static and runs the same way all the time.  In this case, the controller is not needed to keep things running, as HA is handled by the static environment (Loadbalancers / Redundant Nodes).    The controller would therefore only be used for monitoring and orchestration.  It will also be used to apply changes.  





