# Microservice Helm Chart

This is a generic Helm chart for deploying a microservice to a Kubernetes cluster. It is designed to be reusable for the various services in the Vote-Poll application by overriding values.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.2.0+
- A pre-existing namespace (e.g., `vote-poll`).
- Pre-existing `vote-poll-config` ConfigMap and `vote-poll-secrets` Secret in the namespace.

## Installing the Chart

To deploy a microservice (e.g., `auth-service`), you can use this chart. You will need to provide values specific to that service.

A recommended way is to create a `values.yaml` file for each microservice or use `--set` flags during installation.

### Example Deployment

Here's how you could deploy the `auth-service` using this chart.

1.  **Create a `auth-service-values.yaml`:**

    ```yaml
    # values for auth-service
    image:
      repository: auth-service
      tag: "20231120-123456" # Replace with your image tag
    
    # Override service name to be more specific if needed
    nameOverride: "auth-service"
    
    service:
      port: 8080 # Port the service runs on
    
    # Probes for Spring Boot Actuator
    livenessProbe:
      path: /actuator/health
    readinessProbe:
      path: /actuator/health
    
    # Environment variables for this specific service
    env:
      - name: DB_NAME
        valueFrom:
          configMapKeyRef:
            name: vote-poll-config # This is the default, can be omitted if correct
            key: AUTH_DB_NAME # The key in the configmap for auth service's DB name
    ```

2.  **Install the chart:**

    Navigate to the `k8s-helm-charts/microservice` directory and run:
    ```bash
    helm install auth-service . \
      --namespace vote-poll \
      -f /path/to/your/auth-service-values.yaml
    ```

## Configuration

The following table lists the most important configurable parameters of the microservice chart and their default values.

| Parameter | Description | Default |
| --- | --- | --- |
| `replicaCount` | Number of replicas for the deployment. | `1` |
| `image.repository` | **(Required)** The repository for the container image. | `""` |
| `image.tag` | **(Required)** The tag for the container image. | `""` |
| `nameOverride` | String to override the chart name for the service. | `""` |
| `service.type` | Kubernetes service type. | `ClusterIP` |
| `service.port` | The port the service listens on. | `8080` |
| `resources` | CPU/memory resource requests and limits. | (Defined for Minikube) |
| `livenessProbe.path` | Path for the liveness probe. | `/actuator/health` |
| `readinessProbe.path` | Path for the readiness probe. | `/actuator/health` |
| `existingConfigMap` | Name of an existing ConfigMap to source environment variables from. | `vote-poll-config` |
| `existingSecret` | Name of an existing Secret to source environment variables from. | `vote-poll-secrets` |
| `env` | Additional, service-specific environment variables to set. | `[]` |

---

This chart provides a flexible way to manage microservice deployments. For a full deployment of all services, you might consider creating an "umbrella" chart that lists this chart as a dependency for each microservice, each with its own values section.