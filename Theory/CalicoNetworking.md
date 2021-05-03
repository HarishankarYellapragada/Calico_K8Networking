## Calico Network Policy
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
kind: GlobalNetworkPolicy
metadata:
  name: red-policy
spec:
  order: 100      
  selector: color == 'red'
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
```order```number is the priority among the network policies.

One of the calico feature includes in allowing and dening ingress.

## Istio Integration:

* Single familiar network policy model
* Match on L5-7 application layer attributes
* Match cryptographic identity of Service Accounts
* Multiple enforcement points
    - Network Infrastructure layer
    - Service mesh layer
```
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: summary
spec:
  selector: app == 'summary'
  ingress:
  - action: Allow
    http:
      methods: ["GET"]
      paths:
          - exact: "/foo/bar"
          - prefix: "/baz"
    source:
      serviceAccounts:
          names: ["customer']
```
## Best practices for Network Policy
Should specify always both ingress and egress traffic
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
        - 80
  egress:
  - action: Allow
    protocol: TCP
    destination:
      selector: color == 'green'
    destination:
      ports:
        - 80
```
### Policy and Label Schemas
```
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: blue-policy
  namespace: production
spec:
  selector: app == 'back-end'
  ingress:
  - action: Allow
    protocol: TCP
    source:
      selector: app == 'back-end'
    destination:
      ports:
        - 80
  egress:
  - action: Allow
    protocol: TCP
    destination:
      selector: app == 'database'
    destination:
      ports:
        - 80
```
Specifing microservices(app-labels) is another best practie to identify easily 
```
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: blue-policy
  namespace: production
spec:
  selector: app == 'database'
  ingress:
  - action: Allow
    protocol: TCP
    source:
      selector: database-client == 'true'
    destination:
      ports:
        - 80
  egress:
  - action: Deny
```
Specify permission labels is another method among many options.
##Default Deny
Good practice is to apply default deny which wont allow traffic explicitly
Per namespace:
```
apiVersion: projectcalico.org/v3
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: my-namespace
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
For GlobalNetworkPolicy 
```
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: default-deny
spec:
  selector: all()
  types:
  - Ingress
  - Egress
```
Using this may cause problem 
* All pods in all namespaces
* All host endpoints
* Including the control plane
        * (except for configured Calico "failsafe" ports)!
To avoid these problem

