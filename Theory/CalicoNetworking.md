
Calico supports both namespace and non-namespace(global) network policies
Example with namespace:
```
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: blue-policy
  namespace: production
spec:
  selector: color == 'blue'
  ingress:
  - action: Allow
    protocol: TCP
    source:
      selector: color == 'red'
    destination:
      ports:
          80
```
Example with non namespace:
```
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: blue-policy
spec:
  order: 100      //Calico Feature
  selector: color == 'blue'
  ingress:
  - action: Deny
    source:
      selector: color == 'blue'
  - action: Allow
    source:
      serviceAccounts:
        selector: color == 'green'
    destination:
      ports:
          80
```
order number is the priority among the network policies.
One of the calico feature includes in allowing and dening ingress.
