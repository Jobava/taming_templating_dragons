```yaml
.commonLabels: &commonLabels
  app: microservice

.commonSelector: &commonSelector
  matchLabels:
    app: microservice

.commonProbes: &commonProbes
  livenessProbe:
    httpGet:
      path: /health
      port: 80
    initialDelaySeconds: 10
    periodSeconds: 5
  readinessProbe:
    httpGet:
      path: /health
      port: 80
    initialDelaySeconds: 5
    periodSeconds: 5

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
    <<: *commonProbes

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
  name: microservice-v1-0-0
  labels:
    <<: *commonLabels
    version: v1.0.0
spec:
  selector:
    <<: *commonSelector
    version: v1.0.0
  template:
    metadata:
      labels:
        <<: *commonLabels
        version: v1.0.0
    spec:
      <<: *commonPodSpec
      containers:
      - name: microservice
        image: microservice:v1.0.0

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-v1-1-0
  labels:
    <<: *commonLabels
    version: v1.1.0
spec:
  selector:
    <<: *commonSelector
    version: v1.1.0
  template:
    metadata:
      labels:
        <<: *commonLabels
        version: v1.1.0
    spec:
      <<: *commonPodSpec
      containers:
      - name: microservice
        image: microservice:v1.1.0

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-v1-0-0-hpa
spec:
  <<: *commonHpaSpec
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-v1-0-0

---

apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: microservice-v1-1-0-hpa
spec:
  <<: *commonHpaSpec
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: microservice-v1-1-0

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
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: microservice-v1-0-0
            port:
              number: 80
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: microservice-v1-1-0
            port:
              number: 80
```
