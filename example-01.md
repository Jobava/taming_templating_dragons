```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-red
  labels:
    app: microservice
    version: red
spec:
  replicas: 3
  selector:
    matchLabels:
      app: microservice
      version: red
  template:
    metadata:
      labels:
        app: microservice
        version: red
    spec:
      containers:
      - name: microservice
        image: microservice:red
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-green
  labels:
    app: microservice
    version: green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: microservice
      version: green
  template:
    metadata:
      labels:
        app: microservice
        version: green
    spec:
      containers:
      - name: microservice
        image: microservice:green
        ports:
        - containerPort: 80
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: microservice-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /red
        pathType: Prefix
        backend:
          service:
            name: microservice-red
            port:
              number: 80
      - path: /green
        pathType: Prefix
        backend:
          service:
            name: microservice-green
            port:
              number: 80
```
