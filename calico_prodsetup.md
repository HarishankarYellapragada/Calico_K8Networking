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

# Install Calicoctl on Cluster on a Single host (On Master)
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














ref: https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises
