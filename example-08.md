```jsonnet
// Define common configurations as Jsonnet variables or functions
local commonLabels = { app: 'microservice' };
local commonSelector(version) = { matchLabels: commonLabels + { version: version } };
local commonPodSpec(image) = {
  containers: [
    {
      name: 'microservice',
      image: image,
      ports: [{ containerPort: 80 }],
      resources: { requests: { cpu: '100m' }, limits: { cpu: '200m' } },
    },
  ],
};

// Kubernetes manifest using the common configurations
{
  'microservice-v1-0-0-deployment': {
    apiVersion: 'apps/v1',
    kind: 'Deployment',
    metadata: {
      name: 'microservice-v1-0-0',
      labels: commonLabels + { version: 'v1.0.0' },
    },
    spec: {
      selector: commonSelector('v1.0.0'),
      template: {
        metadata: { labels: commonLabels + { version: 'v1.0.0' } },
        spec: commonPodSpec('microservice:v1.0.0'),
      },
    },
  },
  'microservice-v1-1-0-deployment': {
    apiVersion: 'apps/v1',
    kind: 'Deployment',
    metadata: {
      name: 'microservice-v1-1-0',
      labels: commonLabels + { version: 'v1.1.0' },
    },
    spec: {
      selector: commonSelector('v1.1.0'),
      template: {
        metadata: { labels: commonLabels + { version: 'v1.1.0' } },
        spec: commonPodSpec('microservice:v1.1.0'),
      },
    },
  },
  'deployment-monitor-script-configmap': {
    apiVersion: 'v1',
    kind: 'ConfigMap',
    metadata: { name: 'deployment-monitor-script' },
    data: {
      'monitor.sh': |||
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
      |||,
    },
  },
  'version-controller-deployment': {
    apiVersion: 'apps/v1',
    kind: 'Deployment',
    metadata: {
      name: 'version-controller',
      labels: { app: 'version-controller' },
    },
    spec: {
      replicas: 1,
      selector: { matchLabels: { app: 'version-controller' } },
      template: {
        metadata: { labels: { app: 'version-controller' } },
        spec: {
          containers: [
            {
              name: 'version-controller',
              image: 'bitnami/kubectl:latest',
              command: ['/bin/bash', '/scripts/monitor.sh'],
              volumeMounts: [{ name: 'script-volume', mountPath: '/scripts' }],
            },
          ],
          volumes: [
            {
              name: 'script-volume',
              configMap: { name: 'deployment-monitor-script' },
            },
          ],
          serviceAccountName: 'version-controller-sa',
        },
      },
    },
  },
}
```
