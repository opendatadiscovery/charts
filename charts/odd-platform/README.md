# Odd Platform 

This chart installs [Odd Platform](https://github.com/opendatadiscovery/odd-platform).

## Configuration
The application configuration must be passed under `config` section.
Application specific configuration parameters.

| Parameter  | Description                                                                                                                             | Default |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------|--------|
| config.env | All **name** keys values, will be passed as ENV name.  <br/>All **values**, will be passed as ENV values.                              | {}     |
| config.yaml | Yaml formatted configuration file of application. All content of that param will be passed as application.yaml to the pod via ConfigMap | {}     |

Other params are standard for most of other Charts, you could find the example of their usage in `template/values.yaml`

The list of the application parameters could be found [here](PLEASE GIVE THE LINK).



## Usage

Once installed, you can access application on port 80. 

You need to set up port-forwarding to access the application from the local machine.
For example by executing 
`kubectl port-forward svc/odd-platform 8080:80 -n namespace` and open `open http://localhost:8080`

Or configured ingress resources to expose application.

One option to access and examine application configuration is
`NAMESPACE="you-namespace"`
`PODNAME=$(kubectl get pod --selector='app.kubernetes.io/name=odd-platform' -n $NAMESPACE --output=jsonpath="{.items[].metadata.name}")`
`kubectl exec -ti $PODNAME -n $NAMESPACE -- /bin/sh`

## Application specific information
For more information about application architecture and configurations, please visit official [site](https://docs.opendatadiscovery.org/)