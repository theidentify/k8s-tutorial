# Kubernetes Ingress
- Forward request to the internal service

### Configure Ingress in your Cluster
- You need an implementation for Ingress!, Which is Ingress Controller

### What is Ingress Controller?
- evaluates all the rules
- manages redirections
- entrypoint to cluster
- many third-party implementations
- K8s Nginx Ingress Controller and blah blah

### Environment on which your cluster runs
#### Cloud service provider:
- Out-of-the-box K8s solutions
- Own virtualized Load Balancer
### Advantage
- You don't have to implement load balancer by yourself!

### Bare Metal:
- You need to configure some kind of entrypoint
- Either inside of cluster or outside as separate server

### Software or hardware solution(Proxy):
-  Separate server
-  public IP address and open ports
-  entrypoint to cluster
#### Advantage
- No server in K8s Cluster is accessible from outside!
- Security practices

### Ingress Controller in Minikube
- Install Ingress Controller in Minikube
- Automatically start the K8s Nginx implementation of Ingress Controller
```bash
$ minikube addons enable ingress
```
# Demo
```bash
$ kubectl apply -f dashboard-ingress.yaml
ingress.networking.k8s.io/dashboard-ingress created

$ kubectl get ingress -n kubernetes-dashboard

$ kubectl  get ingress -n kubernetes-dashboard --watch

$ sudo vim /etc/hosts <-- mapping hostname

```

### Configure Default Backend in Ingress
```console
$ kubectl describe ingress myapp-ingress
```
```yaml
apiVersion: v1
kind: service
metadata:
  name: default-http-backend
spec:
  selector:
    app: default-response-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
      ...
```

### Multiple paths for same host
- Example Google
- One domain but many services
  ```yaml
  rules:
    - host: myapp
      http:
        paths:
          - path: /analytics
            backend:
              serviceName: analytics-service
              servicePort: 3000
          - path: /shopping
            backend:
              serviceName: shopping-service
              servicePort: 8080
  ```
### Multiple sub-domains or domains
- Instead of http://myapp.com/analytics -> http://analytics.myapp.com
- Instead of 1 host and multiple path. You have now multiple hosts with 1 path. Each host represents a subdomain
  ```yaml
  rules:
    - host: analytics.myapp.com
      http:
        paths:
          backend:
            serviceName: analytics-service
            servicePort: 3000
    - host: shopping.myapp.com
      http:
        paths:
          backend:
            serviceName: shopping-service
            servicePort: 8080

  ```

### Configuring TLS Certificate - https//
1. Data keys need to be "tls.crt" and "tls.key"
2. Values are file contents NOT file paths/locations
3. Secret component must be in the same namespace as the Ingress component
```yaml
## Ingress
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: tls-example-ingress
spec:
  tls:
  - hosts:
    - myapp.com
    secretName: myapp-secret-tls
  rules:
    - host: myapp.com
      http:
        paths:
          - path: /
            backend:
              serviceName: myapp-internal-service
              servicePort: 8080
---
## Secret
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret-tls
  namespace: default
data:
  tls.crt: base64 encoded cert
  tls.key: base64 encoded key
type: kubernetes.io/tls

```

# In Kubernetes Cluster Diagram
                                Ingress Controller Pod
                                        |
                                        |
                  (internal)            v
my-app pod <-- my-app service <-- my-app ingress
