```bash
#!/bin/bash

# Define common labels as a Bash associative array
declare -A commonLabels=( ["app"]="microservice" )

# Define common pod spec as a heredoc JSON
commonPodSpec=$(cat <<EOF
{
  "containers": [
    {
      "name": "microservice",
      "ports": [
        { "containerPort": 80 }
      ],
      "resources": {
        "requests": { "cpu": "100m" },
        "limits": { "cpu": "200m" }
      }
    }
  ]
}
EOF
)

# Function to create a deployment JSON using jq
create_deployment_json() {
    version=$1
    image=$2

    jq -n \
        --argjson labels "$(jq -n --argjson labels "${commonLabels}" '{app: $labels.app, version: $version}')" \
        --argjson selectorLabels "$(jq -n --argjson labels "${commonLabels}" '{matchLabels: {app: $labels.app}}')" \
        --argjson podSpec "${commonPodSpec}" \
        '{
            "apiVersion": "apps/v1",
            "kind": "Deployment",
            "metadata": {
                "name": ("microservice-" + $version),
                "labels": $labels
            },
            "spec": {
                "selector": $selectorLabels,
                "template": {
                    "metadata": { "labels": $labels },
                    "spec": ($podSpec | .containers[0].image = $image)
                }
            }
        }'
}

# Generate deployments
deployment_v1_0_0=$(create_deployment_json "v1.0.0" "microservice:v1.0.0")
deployment_v1_1_0=$(create_deployment_json "v1.1.0" "microservice:v1.1.0")

# Convert JSON to YAML and save to files
echo "$deployment_v1_0_0" | jq -r 'to_entries | .[] | .value' | yq e -P - > deployment_v1_0_0.yaml
echo "$deployment_v1_1_0" | jq -r 'to_entries | .[] | .value' | yq e -P - > deployment_v1_1_0.yaml

# ConfigMap and version-controller Deployment can be similarly templated
```
