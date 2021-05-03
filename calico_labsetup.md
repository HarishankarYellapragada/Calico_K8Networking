# Calico Lab
 We will be creating 4 VMs. (3 Kubernetes nodes and 1 standalone host) each with predefined static IP addresses:

Control (198.19.0.1) - Kubernetes control-plane node

Node1 (198.19.0.2) - Kubernetes worker node

Node2 (198.19.0.3) - Kubernetes worker node

Host1 (198.19.15.254) - General purpose host

(Note that each VM will also have a second dynamically allocated IP address, but in the course we will always use the static IP addresses listed above.)

## Installing Tigera-operator
#### The Tigera Operator manages the lifecycle of a Calico or Calico Enterprise installation on Kubernetes or OpenShift. Its goal is to make installation, upgrades, and ongoing lifecycle management of Calico and Calico Enterprise as simple and reliable as possible.

The command below will install the operator onto our lab kubernetes cluster:
```
kubectl create -f https://docs.projectcalico.org/archive/v3.16/manifests/tigera-operator.yaml
```
## Validating the Operator installation
Following the operator being installed, we will validate that the operator is running:
```
kubectl get pods -n tigera-operator
```
The output from this command should indicate that the operator pod is running:
```
NAME                               READY   STATUS    RESTARTS   AGE
tigera-operator-64f448dfb9-d2fdq   1/1     Running   0          2m33s
```
## Validating the Calico installation
Following the configuration of the installation resource, Calico will begin deploying onto your cluster. This can be validated by running the following command:
```
kubectl get tigerastatus/calico
```
The output from the command when the installation is complete is:
```
NAME     AVAILABLE   PROGRESSING   DEGRADED   SINCE
calico   True        False         False      10m
```
We can review the environment now by invoking:
```
kubectl get pods -A
```
Example output:
```
NAMESPACE         NAME                                      READY   STATUS      RESTARTS   AGE
tigera-operator   tigera-operator-84c5c5d6df-zb49b          1/1     Running     0          5m48s
calico-system     calico-typha-868bb997ff-l22n7             1/1     Running     0          4m6s
calico-system     calico-typha-868bb997ff-fvmws             1/1     Running     0          3m24s
calico-system     calico-typha-868bb997ff-8qt45             1/1     Running     0          3m24s
calico-system     calico-node-r94mp                         1/1     Running     0          4m6s
calico-system     calico-node-w5ptt                         1/1     Running     0          4m6s
calico-system     calico-node-zgrvb                         1/1     Running     0          4m6s
kube-system       helm-install-traefik-t68vd                0/1     Completed   0          35m
kube-system       metrics-server-7566d596c8-pccvz           1/1     Running     2          35m
kube-system       local-path-provisioner-6d59f47c7-gh97b    1/1     Running     2          35m
calico-system     calico-kube-controllers-89cf65556-c7gz7   1/1     Running     3          4m6s
kube-system       coredns-7944c66d8d-f4q6g                  1/1     Running     0          35m
kube-system       svclb-traefik-9bxg2                       2/2     Running     0          32s
kube-system       svclb-traefik-pb72f                       2/2     Running     0          32s
kube-system       svclb-traefik-l6mzn                       2/2     Running     0          32s
kube-system       traefik-758cd5fc85-8hcdx                  1/1     Running     0          32s
```
## Introduction to Sample Application

Customer (which provides a simple web GUI)
Summary (some middleware business logic)
Database (the persistent datastore for the bank)

![image](https://user-images.githubusercontent.com/14257200/116883959-2e2cf400-abf4-11eb-8a26-6c70c13cbad2.png)

All the Kubernetes resources (Deployments, Pods, Services, Service Accounts, etc) for Yaobank will all be created within the yaobank namespace.
```
kubectl apply -f https://raw.githubusercontent.com/tigera/ccol1/main/yaobank.yaml
```
## Verify the Sample Application

### Check the Deployment Status
To validate that the application has been deployed into your cluster, we will check the rollout status of each of the microservices.

Check the customer microservice:
```
kubectl rollout status -n yaobank deployment/customer
```
Example output:
```
deployment "customer" successfully rolled out
```
Check the summary microservice:
```
kubectl rollout status -n yaobank deployment/summary
```
Example output:
```
deployment "summary" successfully rolled out
```
Check the database microservice:
```
kubectl rollout status -n yaobank deployment/database
```
Example output:
```
deployment "database" successfully rolled out
```
