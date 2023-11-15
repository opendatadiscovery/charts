
1. configmap.yaml template for odd-platform has been changed 
```
...
data:
  application.yml: |-
    spring:
      datasource:
        username: "postgres"
        password: "{{ .Values.global.postgresql.auth.postgresPassword }}"
        url: "jdbc:postgresql://odd-quicklaunch-postgresql:5432/odd-platform"
...
```
That has been done to get value for password from value.yaml file of parent chart

2. service.yaml template for odd-platform has been changed
2.1. service.type: 
```
...
spec:
  type: {{ .Values.global.platformServiceType }}
...
```
That has been done as AWS Marketplace QuickLaunch does not accept Override parameter key with '-' that we have in the name of chart. 
So we have to use another name.

2.2 annotations: 
```
...
metadata:
  name: {{ include "odd-platform.fullname" . }}
  annotations:
    "service.beta.kubernetes.io/load-balancer-source-ranges": "{{ .Values.global.loadBalancerSourceRanges }}"
...
```
There is a restriction of 50 characters to be propagated with AWS Marketplace QuickLaunch Override parameter key. 
So we have to use new shorter one.

```commandline
$ helm dependency update .
$ helm package .
$ aws ecr get-login-password --region us-east-1 --profile ${AWSMarketplaceProfile} | helm registry login --username AWS --password-stdin 709825985650.dkr.ecr.us-east-1.amazonaws.com
$ export HELM_EXPERIMENTAL_OCI=1 && helm push ./odd-quicklaunch-1.0.8.tgz oci://709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/
```

Version Information in AWS Marketplace (Examples)

| Property                    | Value                                                                                                                                     |
|-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| Version title               | odd-platform 0.19.0 quicklaunch                                                                                                           |
| Release notes               | Data Quality Dashboard with QuickLaunch capabilities (odd-patform and postgres in one helm chart)                                         | 
| Helm chart                  | 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd-quicklaunch:1.0.13                                                             |                        
| Container Image             | 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0                                                                         |                               
| Delivery option title       | Open Data Discovery Platfrom Quick Launch                                                                                                 |
| Delivery option description | This delivery method allows to deploy odd-platform and its database with one helm chart on newly created AWS EKS cluster with QuickLaunch | 
| Usage instructions          | Go to https://blog.opendatadiscovery.org/ to learn how to deploy odd-collectors.                                                          |
| Supported services          | Amazon Elastic Kubernetes Service (EKS)                                                                                                   |
| Helm release name           | odd-quicklaunch                                                                                                                           |                                                                                                                           
| QuickLaunch                 | Enable QuickLaunch                                                                                                                        |                                                                                                                        

Override parameters

| Override parameter | Override parameter key | Override parameter default | CloudFormation parameter name | CloudFormation parameter description |  Hide passwords and secrets | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Override parameter 1 | global.platformServiceType | LoadBalancer | platformServiceType | Leave default LoadBalancer so that odd-platform has a public host | false |
| Override parameter 2 | global.postgresql.auth.postgresPassword |  | postgresPassword | Password for user postgres that is used to connect odd-platform to its database | true |
| Override parameter 3 | global.loadBalancerSourceRanges | 1.1.1.1/32 | AllowedIPAddresses | List of CIDRs separated with comma that would have connection to odd-platform. For instance, it could be your IP address found at https://whatismyipaddress.com/.Format for single IP: x.x.x.x/32 | false |