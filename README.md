# ALB Ingress Controller with SSL and Path-Based Routing
This repository contains Kubernetes configurations to deploy applications using an ALB (Application Load Balancer) ingress controller with SSL and path-based routing. 

## The setup includes two main scenarios:
1. Deploying a single application with SSL.
2. Deploying multiple applications with path-based routing.
3. Canary Deployment with Path-Based Routing


# Deployment Files
## Single Application with SSL

## Deployment Configuration

deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-react-app-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-react-app
      version: v1
  template:
    metadata:
      labels:
        app: my-react-app
        version: v1
    spec:
      containers:
      - name: my-react-app
        image: ronak1907/react-app:latest
        ports:
        - containerPort: 80
```

## Service Configuration

service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: my-react-app-v1
spec:
  selector:
    app: my-react-app
    version: v1
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: NodePort
```

## Ingress Configuration

ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:992382659631:certificate/036f401e-63a2-48c4-8759-8b8ed7dadd08
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
spec:
  ingressClassName: alb
  rules:
    - host: www.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-react-app-v1
                port:
                  number: 80
```

# Path-Based Routing

## Deployment Configuration

deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: facebook
spec:
  replicas: 3
  selector:
    matchLabels:
      app: facebook
  template:
    metadata:
      labels:
        app: facebook
    spec:
      containers:
      - name: facebook
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: instagram 
spec:
  replicas: 3
  selector:
    matchLabels:
      app: instagram
  template:
    metadata:
      labels:
        app: instagram
    spec:
      containers:
      - name: instagram
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 8080
```

## Service Configuration

service.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: facebook-svc
spec:
  selector:
    app: facebook
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: instagram-svc
spec:
  selector:
    app: instagram
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
```

## Ingress Configuration

ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-ingress-rules
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/load-balancer-name: "alb-ingress"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:992382659631:certificate/036f401e-63a2-48c4-8759-8b8ed7dadd08
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
spec:
  rules:
    - host: www.example.com
      http:
         paths:
           - path: /facebook
             pathType: Prefix
             backend:
               service:
                 name: facebook-svc
                 port:
                   number: 8080
           - path: /instagram
             pathType: Prefix
             backend:
               service:
                 name: instagram-svc
                 port:
                   number: 8080
```

# Canary Deployment with Path-Based Routing

## Deployment Configuration based on version

deployment-v1.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: facebook-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: facebook
      version: v1
  template:
    metadata:
      labels:
        app: facebook
        version: v1
    spec:
      containers:
      - name: facebook
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: facebook-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: facebook
      version: v2
  template:
    metadata:
      labels:
        app: facebook
        version: v2
    spec:
      containers:
      - name: facebook
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 8080
```

deployment-v2.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: instagram-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: instagram
      version: v1
  template:
    metadata:
      labels:
        app: instagram
        version: v1
    spec:
      containers:
      - name: instagram
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: instagram-v2
spec:
  replicas: 3
  selector:
    matchLabels:
      app: instagram
      version: v2
  template:
    metadata:
      labels:
        app: instagram
        version: v2
    spec:
      containers:
      - name: instagram
        image: gcr.io/google-samples/hello-app:2.0
        ports:
        - containerPort: 8080
```

## Service Configuration based on version

service-v1.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: facebook-v1-svc
spec:
  selector:
    app: facebook
    version: v1
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: facebook-v2-svc
spec:
  selector:
    app: facebook
    version: v2
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
```

service-v2.yaml

```
apiVersion: v1
kind: Service
metadata:
  name: instagram-v1-svc
spec:
  selector:
    app: instagram
    version: v1
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
---
apiVersion: v1
kind: Service
metadata:
  name: instagram-v2-svc
spec:
  selector:
    app: instagram
    version: v2
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
  type: NodePort
```

## Ingress Configuration based on version

ingress.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: alb-ingress-rules
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/load-balancer-name: "alb-ingress"
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: instance
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:992382659631:certificate/036f401e-63a2-48c4-8759-8b8ed7dadd08
    alb.ingress.kubernetes.io/actions.ssl-redirect: '{"Type": "redirect", "RedirectConfig": { "Protocol": "HTTPS", "Port": "443", "StatusCode": "HTTP_301"}}'
    alb.ingress.kubernetes.io/actions.weighted-routing-app1: >
      {
        "type": "forward",
        "forwardConfig": {
          "targetGroups": [
            {
              "serviceName": "facebook-v1-svc",
              "servicePort": 8080,
              "weight": 10
            },
            {
              "serviceName": "facebook-v2-svc",
              "servicePort": 8080,
              "weight": 90
            }
          ]
        },
        "targetGroupStickinessConfig": {
          "enabled": false
        }
      }

    alb.ingress.kubernetes.io/actions.weighted-routing-app2: >
      {
        "type": "forward",
        "forwardConfig": {
          "targetGroups": [
            {
              "serviceName": "instagram-v1-svc",
              "servicePort": 8080,
              "weight": 50
            },
            {
              "serviceName": "instagram-v2-svc",
              "servicePort": 8080,
              "weight": 50
            }
          ]
        },
        "targetGroupStickinessConfig": {
          "enabled": false
        }
      }
      
spec:
  ingressClassName: alb
  rules:
  - host: www.example.com
    http:
      paths:
        - path: /facebook
          pathType: Prefix
          backend:
            service:
              name: weighted-routing-app1
              port:
                name: use-annotation

        - path: /instagram
          pathType: Prefix
          backend:
            service:
              name: weighted-routing-app2
              port:
                name: use-annotation

```

# Deployment Steps

1. Create Deployments: Apply the deployment configurations to create the pods.
```
kubectl apply -f .\foldername\yaml-file-name
```

2. Create Services: Apply the service configurations to expose the pods.
```
kubectl apply -f .\foldername\yaml-file-name
```

3. Create Ingress: Apply the ingress configurations to set up the ALB.
```
kubectl apply -f .\foldername\yaml-file-name
```

4. Verify Deployments: Check the status of your deployments, services, and ingress.
```
kubectl get deployments
kubectl get services
kubectl get ingress
```

5. Access the Applications:

## For the single application, navigate to https://www.example.com.
## For path-based routing, navigate to https://www.example.com/facebook and https://www.example.com/instagram.
## For canary strategy path-based routing, navigate to https://www.example.com/facebook and https://www.example.com/instagram.

# Conclusion
This setup demonstrates deploying applications with an ALB ingress controller using SSL and path-based routing. The configurations provided can be customized to fit your specific use case and environment.
