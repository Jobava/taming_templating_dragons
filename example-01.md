```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-red
  labels:
    app: microservice
    version: red
spec:
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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - microservice
            topologyKey: "topology.kubernetes.io/zone"
      containers:
      - name: microservice
        image: microservice:red
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-green
  labels:
    app: microservice
    version: green
spec:
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
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - microservice
            topologyKey: "topology.kubernetes.io/zone"
      containers:
      - name: microservice
        image: microservice:green
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
          limits:
            cpu: "200m"

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-red-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-red
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-green-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-green
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

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
