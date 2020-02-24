# Cloud Interoperability 

This repository contains yaml files that shall be consumed on OCP/K8 for Cloud Interoperability.
The docker images involved are custom built for this particular usecase and leverages manifest that identifies
underlying architecture(s) accordingly and pull respective docker image.

It also demonstrates ease of moving from Public Cloud traditional x86_64 (like GCP,AWS) to 
powerful IBM POWER Systems hosted in Hybrid/Private Clouds ( like Openshift Container Platform)

### Steps to deploy the MongoDB + NodeJS Application on OpenShift Container Platform v3.11-
```
oc create -f mong-service.yaml 
oc create -f node-service.yaml
oc create -f mong-deployment.yaml 
oc create -f node-deployment.yaml 
```
Upon Successfully creating respective Service/Deployment you will see the Output Similar to this -

```
[root@p230n134 cloud_interop]# oc create -f mong-service.yaml 
service/mong created
[root@p230n134 cloud_interop]# oc create -f node-service.yaml 
service/node created
[root@p230n134 cloud_interop]# oc create -f mong-deployment.yaml 
deployment.extensions/mong created
[root@p230n134 cloud_interop]# oc create -f node-deployment.yaml 
deployment.extensions/node created
[root@p230n134 cloud_interop]# 
```

To check the status of pods in your desired namespace issue the following
```
[root@p230n134 cloud_interop]# oc get po -n mithun
NAME                    READY     STATUS    RESTARTS   AGE
mong-6f6cbff4fb-q4ds6   1/1       Running   0          1m
node-b6f55bdb9-rpszs    1/1       Running   0          1m
[root@p230n134 cloud_interop]#
```
Where `-n mithun` is our targeted namespace

#####Todo
How to create a route and test how the endpoint exposed from NodeJS Container hits the Mongodb Container
and indeed pulls in the data.
######

Steps to deploy  the application on Kubernetes cluster running on IBM Cloud Platform

Create a docker-compose.yaml file and generate node-service.yaml,mong-service.yaml,node-deployment.yaml,mong-deployment.yaml
using command "kompose convert -f docker-compose.yml"
contents of sample docker-compose.yaml file are as below

```
docker-compose.yaml 

version: '3'
services:
  node:
    image: docker.io/mithunhr/appnode:latest
    ports:
      - "3000:3000"
    depends_on:
      - mong
  mong:
    image: docker.io/mithunhr/dbmongo:latest
    ports:
      - "27017:27017"
  ```   
On Kubernetes cluster apply the above created files to deploy the application

```
kubectl apply -f mong-service.yaml,node-service.yaml,mong-deployment.yaml,node-deployment.yaml
```
Create a service to expose the deployment

```
kubectl expose deployment/node --type=NodePort --port=8080 --name=node-service --target-port=3000
```

Describe the service to check Nodeport

```
Name:                     node-service
Namespace:                default
Labels:                   io.kompose.service=node
Annotations:              <none>
Selector:                 io.kompose.service=node
Type:                     NodePort
IP:                       172.21.193.88
Port:                     <unset>  8080/TCP
TargetPort:               3000/TCP
NodePort:                 <unset>  31629/TCP
Endpoints:                172.30.39.101:3000
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
````

Query using the Cluster IP and port number

```

http://<Cluster-IP>:31629/api/getInspectionsByZipCodeIteration/10100/10150/3

````

