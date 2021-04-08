# OpenDataDiscovery Helm charts

## Usage
[Helm](https://helm.sh) must be installed to use the charts.  Please refer to
Helm's [documentation](https://helm.sh/docs) to get started.

Once Helm has been set up correctly, add the repo as follows:
``` bash
helm repo add opendatadiscovery https://opendatadiscovery.github.io/charts
```
If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages.  You can then run `helm search repo
opendatadiscovery` to see the charts.

To install the <chart-name> chart:
``` bash
helm install my-<chart-name> opendatadiscovery/<chart-name>
```
To uninstall the chart:
``` bash
helm delete my-<chart-name>
```
## Example
As we use the single chart for all our adapters, need to pass option to the deployment about the wanted adapter, for example, to install [odd-glue-adapter](https://github.com/opendatadiscovery/odd-glue-adapter):
``` bash
helm install odd-glue-adapter opendatadiscovery/odd-adapter --set nameOverride=odd-glue-adapter --set image.repository=ghcr.io/opendatadiscovery/adapters/odd-glue-adapter
```
