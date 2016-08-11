# Providers

Providers are technically a plugin or integration that allows the Trellis system to obtain or modify resources.  A good example of this would be an Amazon provider that can be used to provide computing power, DNS, storage, etc.

Providers are normally an interface that links to an API of some service provider.

Another exmaple might be Digital Ocean.  If we add a provider with appropriate account credentials, then new servers, storage, dns can be setup on demand.  Certain constraints can be included, but it gives the possibility of expanding and reducing the overall environment as needed.