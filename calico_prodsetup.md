# Self-managed on-premises (Install Calico with Kubernetes API datastore, 50 nodes or less)

1. Download the Calico networking manifest for the Kubernetes API datastore. (on one Master)
~~~ txt
curl https://docs.projectcalico.org/manifests/calico.yaml -O
~~~
2. If you are using pod CIDR 192.168.0.0/16, skip to the next step. If you are using a different pod CIDR with kubeadm, no changes are required - Calico will automatically detect the CIDR based on the running configuration. For other platforms, make sure you uncomment the CALICO_IPV4POOL_CIDR variable in the manifest and set it to the same value as your chosen pod CIDR.

3. Customize the manifest as necessary.
   
Apply the manifest using the following command.
~~~ txt
kubectl apply -f calico.yaml
~~~

## Additional changes needed if there are any errors encountered.

make a nodename file under /var/lib/calico/ on all nodes

copy kube config file under /root/.kube/ on all nodes

To avoid CrashLoopBackOff nodes use the following command on all nodes
~~~ txt
docker system prune
~~~

# Install Calicoctl on Cluster on a Single host
1. Log into the host, open a terminal prompt, and navigate to the location where you want to install the binary.
Tip: Consider navigating to a location that’s in your PATH. For example, ``` /usr/local/bin/. ```

2. Use the following command to download the calicoctl binary.
```
curl -o calicoctl -O -L  "https://github.com/projectcalico/calicoctl/releases/download/v3.19.0/calicoctl" 
```
3. Set the file to be executable.
```
chmod +x calicoctl
```
check if calicoctl is installed correctly
```
calicoctl version
```
Note: If the location of calicoctl is not already in your PATH, move the file to one that is or add its location to your PATH. This will allow you to invoke it without having to prepend its location.


# To install BGP route-to-router reflector
By default calico network work in a node-to-node mesh as follows

![image](https://user-images.githubusercontent.com/14257200/119192272-d85aa780-ba4d-11eb-87b6-54393d6fe58f.png)

When requirement grows to 100's/1000's of nodes, node-to-node mesh network may not be sustainable or scalable. To avoid this, route-reflectors are dedicated nodes to make the routes more efficient and scalable option.

![image](https://user-images.githubusercontent.com/14257200/119192247-ced13f80-ba4d-11eb-95ca-90056d681835.png)

Select few nodes from you kubernetes cluster to make them router reflectors.
```
calicoctl get node nodename --export -o yaml > filename.yaml
```
Example: calicoctl get node tmp-k8swk9c3 --export -o yaml > wk9.yaml

Once you get all your nodename yaml files, add the following in each node yaml file
```
labels:
        route-reflector: true
spec:
  bgp:
        routeReflectorClusterID: 1.0.0.1
```
Replace all yaml files indivudually with following command
```
calicoctl apply -f filename.yaml
```
## Create the following yaml files ( rr-peers.yaml, bgp-peer.yaml, bgp-config.yaml)

## a) rr-peers.yaml
```
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: rr-mesh
spec:
  nodeSelector: has(route-reflector)
  peerSelector: has(route-reflector)
```
```
calicoctl apply -f rr-peers.yaml
```
## b) bgp-peer.yaml

```
kind: BGPPeer
apiVersion: projectcalico.org/v3
metadata:
  name: peer-to-rrs
spec:
  nodeSelector: !has(route-reflector)
  peerSelector: has(route-reflector)
```
```
calicoctl apply -f bgp-peer.yaml
```

## Try to get bgp-config Yaml file
```
calicoctl get bgpconfig default  

resource does not exist: BGPConfiguration(default) with error: bgpconfigurations.crd.projectcalico.org "default" not found
```
{if you get the below error, create one}

## c) bgp-config.yaml

```
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  logSeverityScreen: Info
  nodeToNodeMeshEnabled: false
  asNumber: 63400
```
```
calicoctl apply -f bgp-config.yaml
```













### References : 
https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises

https://www.tigera.io/blog/configuring-route-reflectors-in-calico/#:~:text=Once%20started%2C%20Calico%20runs%20a,an%20IP%20range%20of%2010.30.
