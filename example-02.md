```
.commonSpec: &commonPodSpec
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
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"

.commonHpaSpec: &commonHpaSpec
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
      <<: *commonPodSpec
      containers:
      - name: microservice
        image: microservice:red

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
      <<: *commonPodSpec
      containers:
      - name: microservice
        image: microservice:green

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-red-hpa
spec:
  <<: *commonHpaSpec
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-red

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-green-hpa
spec:
  <<: *commonHpaSpec
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-green

---

# Unchanged ingress
```
