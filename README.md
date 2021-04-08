# OpenDataDiscovery Helm charts

``` bash 
helm repo add opendatadiscovery https://opendatadiscovery.github.io/charts/
```

To install [odd-glue-adapter](https://github.com/opendatadiscovery/odd-glue-adapter):
``` bash
helm install odd-glue-adapter opendatadiscovery/odd-adapter --set nameOverride=odd-glue-adapter --set image.repository=opendatadiscovery/odd-glue-adapter
```
