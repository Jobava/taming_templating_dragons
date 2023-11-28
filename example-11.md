```python
from pulumi_kubernetes.apps.v1 import Deployment
from pulumi_kubernetes.core.v1 import ConfigMap

# Reusable components
def common_labels(version):
    return {"app": "microservice", "version": version}

def common_selector(version):
    return {"matchLabels": common_labels(version)}

def common_pod_spec(version, image):
    return {
        "containers": [{
            "name": "microservice",
            "ports": [{"containerPort": 80}],
            "resources": {
                "requests": {"cpu": "100m"},
                "limits": {"cpu": "200m"}
            },
            "image": image
        }]
    }

# Deployment definitions
def create_deployment(version, image):
    return Deployment(
        f"microservice-{version}",
        metadata={
            "name": f"microservice-{version}",
            "labels": common_labels(version)
        },
        spec={
            "selector": common_selector(version),
            "template": {
                "metadata": {"labels": common_labels(version)},
                "spec": common_pod_spec(version, image)
            }
        }
    )

# Creating Deployments
deployment_v1_0_0 = create_deployment("v1.0.0", "microservice:v1.0.0")
deployment_v1_1_0 = create_deployment("v1.1.0", "microservice:v1.1.0")

# ConfigMap
config_map = ConfigMap(
    "deployment-monitor-script",
    metadata={"name": "deployment-monitor-script"},
    data={
        "monitor.sh": """
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
        """
    }
)

# Version controller Deployment
version_controller = Deployment(
    "version-controller",
    spec={
        "replicas": 1,
        "selector": {"matchLabels": {"app": "version-controller"}},
        "template": {
            "metadata": {"labels": {"app": "version-controller"}},
            "spec": {
                "containers": [{
                    "name": "version-controller",
                    "image": "bitnami/kubectl:latest",
                    "command": ["/bin/bash", "/scripts/monitor.sh"],
                    "volumeMounts": [{
                        "name": "script-volume",
                        "mountPath": "/scripts"
                    }]
                }],
                "volumes": [{
                    "name": "script-volume",
                    "configMap": {"name": "deployment-monitor-script"}
                }],
                "serviceAccountName": "version-controller-sa"
            }
        }
    }
)
```
