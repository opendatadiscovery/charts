# Helm Chart for AWS Marketplace QuickLaunch

For AWS Marketplace with product offering with Helm chart delivery method and support of QuickLaunch we need to prepare a parent helm chart that has to child charts: odd-platform and postgresql database for it.
This approach would benefit from utilizing (QuickLaunch)[https://docs.aws.amazon.com/marketplace/latest/buyerguide/buyer-configuring-a-product.html#buyer-launch-container-quicklaunch]: first AWS Marketplace creates a new EKS cluster with its built-in CloudFormation for EKS and in the same stack it deploys mentioned Helm chart that should be stored at AWS ECR managed by AWS Marketplace that was created during new product registration.

There are some differences to the base helm chart of odd-platform that should reflect new approach of AWS Marketplace QuickLaunch.

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


# Steps to update version
Change tags for odd-platform and chart version according to the actual state. In this example odd-platform version 0.19.0 and chart version 1.0.8 are used.
```commandline
$ aws ecr get-login-password --region us-east-1  --profile ${AWSMarketplaceProfile} | docker login --username AWS --password-stdin 709825985650.dkr.ecr.us-east-1.amazonaws.com
$ docker pull ghcr.io/opendatadiscovery/odd-platform:0.19.0
$ docker tag ghcr.io/opendatadiscovery/odd-platform:0.19.0 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0
$ docker push 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0
$ # Update Helm chart version in Chart.yaml file that will be used in AWS Marketplace. This version is not correlated to odd-platfrom version but should be unique from each deployment 
$ helm dependency update .
$ helm package .
$ aws ecr get-login-password --region us-east-1 --profile ${AWSMarketplaceProfile} | helm registry login --username AWS --password-stdin 709825985650.dkr.ecr.us-east-1.amazonaws.com
$ # In this example 1.0.8 has been set as Helm chart version in previous steps. Please, change Helm chart version to the actual one before pushing Helm chart to the AWS ECR 
$ export HELM_EXPERIMENTAL_OCI=1 && helm push ./odd-quicklaunch-1.0.8.tgz oci://709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/
```

# Add new version in AWS Marketplace (Examples)

## Version Information 

| Property                    | Value                                                                                                                                    |
|-----------------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| Version title               | odd-platform 0.19.0 quicklaunch                                                                                                          |
| Release notes               | Data Quality Dashboard with QuickLaunch capabilities (odd-patform and postgres in one helm chart)                                        |

## Delivery option: Open Data Discovery Platform (Delivery method Container image)

| Property                        | Value                                                                                                                                                       |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Container image                 | 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0                                                                                           |                        
| Delivery option description     | A simple installation of one instance of Open Data Discovery platform application. More info at https://github.com/opendatadiscovery/odd-platform/tree/main |
| Usage instructions              | (*) Usage instructions (see reference)                                                                                                                      |
| Supported services              | Amazon Elastic Container Service (ECS), Amazon Elastic Kubernetes Service (EKS), Amazon ECS Anywhere, Amazon EKS Anywhere / Self-managed Kubernetes         |
| Deployment templates - optional | Resource title: Helm Chart; Resource URL: https://github.com/opendatadiscovery/charts/tree/main/charts/odd-platform                                                                                                                  |                                                                                                                           


(*) Usage instructions:
```
Running as a separate container.

Setting up PostgreSQL connection details, for example, if you run PostgreSQL locally in as a docker container and provided login/password as postgres/mysecretpassword - change login/password to actual ones:
```
export POSTGRES_HOST=172.17.0.1 \
export POSTGRES_PORT=5432 \
export POSTGRES_DATABASE=postgres \
export POSTGRES_USER=postgres \
export POSTGRES_PASSWORD=mysecretpassword
```
Starting new instance of the platform:
```
docker run -d \
  --name odd-platform \
  -e SPRING_DATASOURCE_URL=jdbc:postgresql://${POSTGRES_HOST}:${POSTGRES_PORT}/${POSTGRES_DATABASE} \
  -e SPRING_DATASOURCE_USERNAME=${POSTGRES_USER} \
  -e SPRING_DATASOURCE_PASSWORD=${POSTGRES_PASSWORD} \
  -p 8080:8080 \
  709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0
```
More info at https://github.com/opendatadiscovery/odd-platform
```

## Delivery option: Open Data Discovery Platform Quick Launch (Delivery method Helm chart)

| Property                    | Value                                                                                                                                     |
|-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| Helm chart                  | 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd-quicklaunch:1.0.13                                                             |                        
| Container Image             | 709825985650.dkr.ecr.us-east-1.amazonaws.com/provectus/odd:0.19.0                                                                         |                               
| Delivery option description | This delivery method allows to deploy odd-platform and its database with one helm chart on newly created AWS EKS cluster with QuickLaunch | 
| Usage instructions          | Utilizing QuickLaunch through CloudFormation enables a hassle-free setup of the Open Data Discovery platform. QuickLaunch streamlines the process by establishing a new EKS cluster and deploying a pre-set Helm chart that includes the odd-platform and a corresponding Postgresql database. You can explore the Chart's details and its AWS Marketplace inclusion at ODD Helm Chart GitHub - https://github.com/opendatadiscovery/charts/tree/main/charts/odd-quicklaunch.  When setting up, choose a name of your liking for the stack and the EKS cluster (for example, you might use 'ODD-EKS' for both). Keep the 'Helm release name' as 'odd-quicklaunch'. For 'postgresPassword', select a secure password that will be used by the 'postgres' admin in the database and for connecting the odd-platform to the database. For 'AllowedIPAddresses', input a CIDR-formatted IP range or single IP (like 8.8.8.8/32) that will have access to the odd-platform. Determine your IP address at resources such as https://whatismyipaddress.com/. Remember, this IP might change with network reconfigurations, and you'll need to update access rules accordingly, as guided here: https://blog.opendatadiscovery.org/quick-launch-of-open-data-discovery-odd-platform-on-amazon-elastic-kubernetes-service-eks-5a0a4489e492.  The entire setup process should take approximately 30-40 minutes. Once completed, navigate to the EKS section in the AWS Console. Locate your named EKS cluster, and under 'Resources', proceed to 'Service and networking', then 'Services'. Here, find and click on 'odd-quicklaunch-odd-platform'. The service page will display 'Load Balancer URLs', leading to your odd-platform. Note that accessing these URLs defaults to an HTTPS protocol, which QuickLaunch does not support. Ensure you access it via HTTP, like http://[your-load-balancer-URL]. Double check that if, for example, 'Load Balancer URLs' for the server looks like this 'ac3a9a5400b1d457dacba505325901cd-951018742.us-east-1.elb.amazonaws.com',   the url in you browser would like this 'http://ac3a9a5400b1d457dacba505325901cd-951018742.us-east-1.elb.amazonaws.com'. Important Considerations: This setup only supports HTTP, not HTTPS, and is not secure for transmitting sensitive data. For production, consider enabling HTTPS.  The database data is not persistent. A restart of the postgres pod will erase data, so for production use, consider a persistent postgresql setup.  This setup excludes odd collectors and adapters, meaning no automatic data ingestion. To add these, refer to our blog post https://blog.opendatadiscovery.org/introducing-odd-collector-configuration-for-aws-eks-bcc2bf04ae7e.  We are always happy to assist if you need any help! Just reach out to our community at ODD Community Slack https://go.opendatadiscovery.org/slack                                                                                                                                          |
| Supported services          | Amazon Elastic Kubernetes Service (EKS)                                                                                                   |
| Helm release name           | odd-quicklaunch                                                                                                                           |                                                                                                                           
| QuickLaunch                 | Enable QuickLaunch                                                                                                                        |

# Override parameters

| Override parameter | Override parameter key | Override parameter default | CloudFormation parameter name | CloudFormation parameter description |  Hide passwords and secrets | 
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Override parameter 1 | global.platformServiceType | LoadBalancer | platformServiceType | Leave default LoadBalancer so that odd-platform has a public host | false |
| Override parameter 2 | global.postgresql.auth.postgresPassword |  | postgresPassword | Password for user postgres that is used to connect odd-platform to its database | true |
| Override parameter 3 | global.loadBalancerSourceRanges | 1.1.1.1/32 | AllowedIPAddresses | List of CIDRs separated with comma that would have connection to odd-platform. For instance, it could be your IP address found at https://whatismyipaddress.com/.Format for single IP: x.x.x.x/32 | false |
