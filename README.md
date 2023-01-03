# OpenDataDiscovery Helm charts

``` bash 
helm repo add opendatadiscovery https://opendatadiscovery.github.io/charts/
```

To install [odd-collector](https://github.com/opendatadiscovery/odd-collector):
``` bash
helm install odd-collector opendatadiscovery/odd-adapter --set nameOverride=odd-collector --set image.repository=ghcr.io/opendatadiscovery/odd-collector
```
