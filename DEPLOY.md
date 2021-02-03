[![DuniaWork](https://img.shields.io/badge/Dunia%20Pay-Storefront-green.svg)]()

# Deploy and setup a docker project on Google Cloud


## Tools & platforms;

```
Docker

Google Cloud / Kubernetes / gke

CircleCI

YAML / BASH
 ```


## Intro;
 
The App is live on http://34.122.206.39:3000/ (you could bring it down after doing the testing to cut on GCP billing costs)

To test out the CI & CD functionality;

## Production deployment changes recommendations;

This test deployment is to test how stuff would work. The following has been done. 

Creating a separate cluster (reaction-test) for test in GKE (Ask Isaac for access).

Created a cluster admin user 

`kubectl create clusterrolebinding "cluster-admin-$(whoami)" --clusterrole=cluster-admin `--user="$(gcloud config get-value core/account)`

Created the db namespace 

`kubectl apply -f navigate/to/kubernetes/folder/mongodb/namespace.yaml`

Created the ssd and hdd classes

`kubectl apply -f navigate/to/kubernetes/folder/mongodb/googlecloud_ssd.yaml`

Created the daemon set as a startup script to disable hugepage on host 

`kubectl apply -f navigate/to/kubernetes/folder/mongodb/mongo-daemon.yaml`

Created a 3 nodes Replica Set, each replica provisioned with a 1Gi SSD disk:

`kubectl apply -f navigate/to/kubernetes/folder/mongodb/mongo-stateful.yaml`

Created a mongo service 

`kubectl apply -f navigate/to/kubernetes/folder/mongodb/mongo-stateful.yaml`

Created a default serviceaccount 

`kubectl apply -f navigate/to/kubernetes/folder/auth-security/serviceaccount.yaml`

Created a default clusterole 

`kubectl apply -f navigate/to/kubernetes/folder/auth-security/role.yaml`

Created a clusterrole-binding

`kubectl apply -f navigate/to/kubernetes/folder/auth-security/rolebinding.yaml`

Created a reaction API deployment 

`kubectl apply -f navigate/to/kubernetes/folder/resources/deployment.yaml`

Created a reaction API service 

`kubectl apply -f navigate/to/kubernetes/folder/resources/service.yaml`


Commands to run 

Get mongo statefulsets

`kubectl get statefulset`

Get mongo pods in db namespace

`kubectl -n db get pods --selector=role=mongo`

Connect to the first Replica Set member in db namespace

`kubectl -n db exec -it mongo-0 -c mongod mongo`

Scale up the number of Replica Set members from 3 to 5

`kubectl scale --replicas=5 statefulset mongo`

Scale down the number of Replica Set members from 5 to 1

`kubectl scale --replicas=1 statefulset mongo`

Get api pods in default namespace

`kubectl get pods`

Get api deployment in default namespace

`kubectl get deployment`

Get api service

`kubectl get svc`

Get api pods in detail

`kubectl describe pod/pod-name`

Get api pod logs

`kubectl logs -f pod-name`


Clean Up

To clean up the deployed resources, delete the StatefulSet, Headless Service, and the provisioned volumes.

Delete the StatefulSet `kubectl delete statefulset mongo`
Delete the Service `kubectl delete svc mongo`
Delete the api service `kubectl delete svc svc-name`
Delete the api deployment `kubectl delete deployment  deployment-name`

Same format above works for all the other resources that have been creayed in the cluster


## CI/CD config

The following the steps for ensuring the project moves in an agile format doing CI/CD


```
CI/CD scripts has been added in a new `scripts` folder (Not yet tested)

I've added a deploy job in `.circleci/config` file with it's corresponding workflow at the bottom (Not yet tested)

Clone the repo `git clone `https://github.com/reactioncommerce/reaction.git` and follow instruction in the readme to have it up and running

Check out to my branch 'scott-duniapay/test/deplyment` which is being used in the circleci config file as a deployment trigger branch

Navigate to the `code` and edit something `that will be reflected as new changes` in any file

Commit your changes on the specified branch and push it to `github`

CircleCI will pick up the changes, run the build, deploy steps. Docker will dockerize the application and Kubernetes will set the new built image as the latest deployment.

Navigate to the browser on `http://34.122.206.39:3000/` and look out for your changes that will be baked in the new image.
```


## Production deployment changes recommendations;


This deployment is to test how stuff would work. To match production level standard / best practices, the following changes would would be ideal,


```
Creating a separate cluster for production.

Using `Helm` (Kubernetes package manager) charts to deploy all the open source (OSS) components of Reaction  (API, Reaction Identity, Reaction Hydra, and Reaction Admin) if you must use all of them

Making use of `Google container registry` service to store the images (`grc`). Currently images are being stored on `Dockerhub` reaction account.

Using `Config-maps` in kubernetes to pass on configuration to the pod.

Using `Secrets` in kubernetes to store credentials / sensitive data.
 ```