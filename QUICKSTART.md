# Quick Launch of Open Data Discovery platform and collector on Amazon Elastic Kubernetes Service (EKS)

***

## What will be deployed

This is the new way for data teams to discover, understand, trust, and collaborate on data assets. ODD serves as a tool to put Data Governance strategies into practice and this guide will show you an easy way to get Open Data Discovery up and running on Amazon EKS.

New environment will consists of:

* ODD Platform – an application that collects, structures, indexes and provides a metadata via REST API and UI
* PostgreSQL database that is used by ODD Platform as a persistence storage&#x20;
* ODD Collector with configured PostgreSQL adapter that grabs metadata from the ODD Platform's database

## Prerequisites

* Before you start, ensure that you have an **AWS account** and if not, then you have to create one.

## Overview of the Quick Launch

* Provision an EKS Cluster
* Install and deploy PosgreSQL
* Deploy and run Open data Discovery (ODD)
* Configure deploy and run Collector

## Start an EKS Cluster

* **Step 1**. Click on [Quick lunch](https://us-east-2.console.aws.amazon.com/cloudformation/home?region=eu-central-1#/stacks/create/review?templateURL=https://odd-ct-templates.s3.us-east-2.amazonaws.com/odd\_cloudformation.yaml\&stackName=ODD-EKS) and you’ll be redirected to Cloud Formation Stack on AWS the account where you are logged in. Please, check that you are in one of the supported regions: us-west-2, us-west-1, us-east-2, us-east-1.
* **Step 2**. You’ll be directed through several setup stages, including following ones:
  * **Cluster Setup**
    * Cluster Name: Supply a unique and descriptive name for your EKS cluster, like “MyEKS-Cluster”. The default name is pre-set as: ODD-EKS.
  * **Node Group**
    * Instance Types: Choose EC2 Instance types for your worker nodes. The default type is pre-set as: t3.large.
    * Desired Capacity: Indicate the quantity of worker nodes you want in the node group, The default is configured as 1.
    * SSH Key Pair: Opt for an existing or create a new one for secure worker node access.
  * **Role**
    * Provide an existing role with sufficient privileges or create and assign a new one.
* **Step 3**.Check all your configurations to confirm their correctness.
* **Step 4**.Click “Create Stack” to confirm the EKS cluster creation process.

## Access and Manage your EKS Cluster

### Authentication with AWS EKS

To begin, authenticate kubectl with your EKS cluster. AWS offers a convenient command:

`aws eks --region <region> update-kubeconfig --name <cluster-name>`

Replace with the AWS region where your EKS cluster is deployed and with the name of your EKS cluster to have a command similar to following:

`aws eks --region us-east-1 update-kubeconfig --name ODD-EKS`

At the current state only following regions are available:

* **us-west-2**
* **us-west-1**
* **us-east-2**
* **us-east-1**

### Verification and Configuration

Confirm that your kubectl configuration is correctly set by listing the available nodes in your cluster:

`kubectl get nodes`

## Install Helm for your EKS Cluster

### Obtain the Helm binary

Visit the Helm [Github releases page](https://github.com/helm/helm/releases) and download the suitable Helm binary. You can use the following command:

`sudo yum install -y openssl && curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash`

To ensure a successful installation, use the command:

`helm version --short`

### Add a Helm Chart Repository

Add a repository to access pre-built charts:

`helm repo add bitnami https://charts.bitnami.com/bitnami`

## Install PosgreSQL using Helm

Install PosgreSQL with the command:

`helm install postgresql bitnami/postgresql --set primary.persistence.enabled=false --set global.postgresql.auth.database=odd-platform`

This basic deployment can be tailored by adjusting values in the Helm chart to meet your specific requirements.

To check the status of your deployment after the installation is done, use:

`kubectl get pods`

Upon the successful installation of PosgreSQL, an auto-generated password becomes available.It’s a good practice to store this password as an environment variable and use it when working with the ODD platform.

To do that, execute the following command:

`export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)`

## Deploy Open data Discovery (ODD)

To deploy ODD platform, first you need to add a repository:

`helm repo add opendatadiscovery https://opendatadiscovery.github.io/charts`

### Install the platform.

`helm install odd-platform opendatadiscovery/odd-platform --set config.yaml.spring.datasource.username=postgres --set config.yaml.spring.datasource.password="$POSTGRES_PASSWORD" --set config.yaml.spring.datasource.url="jdbc:postgresql://postgresql:5432/odd-platform" --set service.type=LoadBalancer --set service.annotations."service\.beta\.kubernetes\.io/load-balancer-source-ranges"="<IPAddressOfYourLocalStationHere>/32"`

To find your IP address follow these instructions.

* For Windows OS, you can search for “What is my IP” in your preferred search engine.
* For MacOS and Linux, use the command `wget -qO- ipecho.net/plain` And your public IP address will be displayed in the terminal output. Also, if you are behind a router firewall, the IP address you retrieve will be the public IP assigned to your router by your ISP.

For example,

`helm install odd-platform opendatadiscovery/odd-platform --set config.yaml.spring.datasource.username=postgres --set config.yaml.spring.datasource.password="$POSTGRES_PASSWORD" --set config.yaml.spring.datasource.url="jdbc:postgresql://postgresql:5432/odd-platform" --set service.type=LoadBalancer --set service.annotations."service\.beta\.kubernetes\.io/load-balancer-source-ranges"="83.3.12.58/32"`

If you wish to enable connectivity with multiple IPs, you’ll need to execute the following set of commands instead:

`helm upgrade odd-platform opendatadiscovery/odd-platform --set config.yaml.spring.datasource.username=postgres --set config.yaml.spring.datasource.password=" $POSTGRES_PASSWORD" --set config.yaml.spring.datasource.url="jdbc:postgresql://postgresql:5432/odd-platform" --set service.type=LoadBalancer --set service.annotations."service\.beta\.kubernetes\.io/load-balancer-source-ranges"="<YourIPAddressHere>/32\,<AnotherIPAddressHere>/32"`

Do not forget to replace the strings and in this command with your IP addresses separated with commas and written in double quotation marks.

### How to be sure everything is Up and Running?

There is a common command for this action:

`kubectl get pods`

`kubectl get svc`

After completing the setup and ensuring everything is up and running, you can start using the ODD platform through your web browser. To do this, obtain the hostname of your Load Balancer and use it to establish a connection to your EKS.

`kubectl get svc odd-platform -o=custom-columns=EXTERNAL-IP:.status.loadBalancer.ingress[0].hostname | tail -n 1`

If the setup is successful, you will be able to access the platform demo page directly from your web browser.

_With the versions of the platform >= 0.18.0 you could get acquainted with the API of the platform by simply visiting_ [_Swagger UI_](../../api/v3/webjars/swagger-ui/index.html)_. For example, if for Load Balancer host `a1e67ff8befc54b75969f9834a6e329a-948212351` we could visit `http://a1e67ff8befc54b75969f9834a6e329a-948212351.us-east-1.elb.amazonaws.com/api/v3/webjars/swagger-ui/index.html`._

## Important Note!

In this setup there are no certificates created to use encrypted communication. Be aware that only http protocol is supported in this setup. For example, `http://a1e67ff8befc54b75969f9834a6e329a-948212351.us-east-1.elb.amazonaws.com/` This protocol is not secure, please, do not send any sensitive information via this connection! Demonstration purpose only! For production cases please configure HTTPS Protocol.

## How to delete Cloudformation Stack?

Deletion starts with uninstalling the platform

`helm uninstall odd-platform`

To avoid incurring additional charges or when you’re confident that you no longer require your current resources any longer you can [delete your Cloudformation Stack](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-delete-stack.html).

## ODD Collector Configuration for AWS EKS

Setting up the Collector involves several steps.

* Create a **Namespace** and proceed to initiate the addition of **a new collector**. Chose a namespace from the drop-down list of available options, optionally include a description and save the settings.
* Make sure to securely copy and store the **token** generated by the platform for future use and if not, then the token will need to be regenerated for your next session.
* Now, it is time to proceed with adding the ODD repository and configuring the collector files. This can be accomplished by executing the following commands in the specified order.

`helm repo add opendatadiscovery https://opendatadiscovery.github.io/charts`

`wget https://raw.githubusercontent.com/opendatadiscovery/charts/main/cloudformation/collector-values.yaml`

**Note:** you need to replace the **Generated token** part in following command with the token you have copied earlier and run it.

`sed -i 's/odd-token/<Generated token>/g' collector-values.yaml`

`export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgresql -o jsonpath="{.data.postgres-password}" | base64 -d) helm install odd-collector opendatadiscovery/odd-collector --set nameOverride=odd-collector --set passwordSecretsEnvs.POSTGRES_PASSWORD=$POSTGRES_PASSWORD -f collector-values.yaml`

If you’ve followed the instructions correctly, you should see in outcome in your Cloudshell informing you that ODD Collector is up and running.

Furthermore, we’ve made it available for you to include additional plugins if desired.

To do that, manually update the `collector-values.yaml` file with your chosen text editor and then run the following command in the CloudShell:

`helm upgrade --install odd-collector opendatadiscovery/odd-collector --set nameOverride=odd-collector --set passwordSecretsEnvs.POSTGRES_PASSWORD=$POSTGRES_PASSWORD -f collector-values.yaml`
