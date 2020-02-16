# rancher-quickstart
Everything needed to bootstrap a Rancher 2.3.5 server.

# Getting Started
`docker-compose up -d`

# Community Helm Charts
Rancher provides out of the box support for helm charts. To enable the community charts:
1. Navigate to `Tools` -> `Catalogs` and select `Add Catalog`
2. Populate the following and add the catalog
    * Name: helm-stable
    * Catalog URL: https://kubernetes-charts.storage.googleapis.com
    * Branch: Master
    * Scope: Global
3. If successful, the repo will sync and enter an `Active` state

# Cluster Creation
## EKS
1. Generate an `AWSAccessKeyId` and `AWSSecretKey` that can create VPCs, EKS, and EC2 instances. 
2. Log into Rancher and EKS under `Clusters` -> `Add Cluster` 
3. Declare a `Cluster Name` and then under `Account Access` select target region and add `AWSAccessKeyId` and `AWSSecretKey`
4. Hit Next and specific which Kubernetes version to use.
5. Hit Next and allow Rancher to create a new VPC
    * The default VPC allows the internet to talk to the Kubernetes API server - don't use this for anything other than testing. 
6. Hit Next and select the EC2 instance type
    * `t2.small` is the smallest viable instance
7. Hit create and wait ~15 minutes for the cluster to be created.
    * Status can be found in AWS Console under `CloudFormation`, `EKS` and `EC2`
8. Grab the `Kubeconfig File` and save it as `kubeconfig`
    * Can be used to access the cluster via `kubectl --kubeconfig kubeconfig ...`

### Ingress Controller
After adding the community charts, perform the following:
1. Navigate to `Apps` -> `Launch`
2. Search for `nginx-ingress` and launch it with default options
3. After a couple minutes, the ingress controller should be active and located in a new `nginx-ingress` namespace
    * Validate with `kubectl --kubeconfig kubeconfig describe svc nginx-ingress --namespace=nginx-ingress`
    * Grab the DNS name for the new AWS Load Balancer, found at `LoadBalancer Ingress` field
4. Create a `Deployment` and `Service` 
    * `kubectl --kubeconfig kubeconfig apply -f manifests/cafe.yaml`
5. Create an `Ingress`
    * `kubectl --kubeconfig kubeconfig apply -f manifests/cafe-ingress.yaml`
6. Verify that the service is now exposed publicly by visiting `LoadBalancerDNSName/coffee`
    * Should return something like 
    ```
    Server address: 192.168.240.233:8080
    Server name: coffee-6488c448f7-4xkqc
    Date: 16/Feb/2020:19:01:35 +0000
    URI: /coffee
    Request ID: 4392cf0fd590facbd892756aeebf5097
    ```

### Cleanup
1. Uninstall the ingress controller and delete the AWS load balancer by navigating to `Apps` and deleting the `nginx-ingress` app
2. Delete the EKS cluster by selecting `Delete` from the main cluster dashboard.
    * Rancher will show the cluster as being deleted immediately, but it will take a few minutes for everything to be purged from AWS
