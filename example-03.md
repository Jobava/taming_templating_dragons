```yaml
.commonLabels: &commonLabels
  app: microservice

.commonSelector: &commonSelector
  matchLabels:
    app: microservice

.commonPodSpec: &commonPodSpec
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
    <<: *commonLabels
    version: red
spec:
  selector:
    <<: *commonSelector
    version: red
  template:
    metadata:
      labels:
        <<: *commonLabels
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
    <<: *commonLabels
    version: green
spec:
  selector:
    <<: *commonSelector
    version: green
  template:
    metadata:
      labels:
        <<: *commonLabels
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

# ... Ingress
```
