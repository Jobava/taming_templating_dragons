```jinja2
{% macro common_labels() %}
app: microservice
{% endmacro %}

{% macro common_selector(version) %}
matchLabels:
  app: microservice
  version: {{ version }}
{% endmacro %}

{% macro common_pod_spec(image) %}
containers:
  - name: microservice
    image: {{ image }}
    ports:
    - containerPort: 80
    resources:
      requests:
        cpu: "100m"
      limits:
        cpu: "200m"
{% endmacro %}

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-v1-0-0
  labels:
    {{ common_labels() }}
    version: v1.0.0
spec:
  selector:
    {{ common_selector('v1.0.0') }}
  template:
    metadata:
      labels:
        {{ common_labels() }}
        version: v1.0.0
    spec:
      {{ common_pod_spec('microservice:v1.0.0') }}

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: microservice-v1-1-0
  labels:
    {{ common_labels() }}
    version: v1.1.0
spec:
  selector:
    {{ common_selector('v1.1.0') }}
  template:
    metadata:
      labels:
        {{ common_labels() }}
        version: v1.1.0
    spec:
      {{ common_pod_spec('microservice:v1.1.0') }}

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: deployment-monitor-script
data:
  monitor.sh: |
    #!/bin/bash
    GREEN="microservice-v1-1-0"; RED="microservice-v1-0-0"
    while :; do
      READY=$(kubectl get deploy $GREEN -o jsonpath='{.status.readyReplicas}')
      TOTAL=$(kubectl get deploy $GREEN -o jsonpath='{.status.replicas}')
      if [[ "$READY" == "0" ]] && [[ "$TOTAL" != "0" ]]; then
        kubectl scale deploy $GREEN --replicas=0
        kubectl scale deploy $RED --replicas=$TOTAL
      fi
      sleep 1
    done

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: version-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: version-controller
  template:
    metadata:
      labels:
        app: version-controller
    spec:
      containers:
      - name: version-controller
        image: bitnami/kubectl:latest
        command: ["/bin/bash", "/scripts/monitor.sh"]
        volumeMounts:
        - name: script-volume
          mountPath: /scripts
      volumes:
      - name: script-volume
        configMap:
          name: deployment-monitor-script
      serviceAccountName: version-controller-sa
```
