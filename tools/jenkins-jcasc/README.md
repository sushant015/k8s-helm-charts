# Jenkins JCasC Helm Chart

This Helm chart deploys a Jenkins master configured with Jenkins Configuration as Code (JCasC).

It is pre-configured for easy deployment on a local Minikube cluster, but can be adapted for any Kubernetes environment.

## Features

-   **Jenkins with JCasC**: Manage Jenkins configuration in a `values.yaml` file.
-   **Persistence**: Uses a PersistentVolumeClaim to retain Jenkins home directory data across restarts.
-   **Customizable Image**: Designed to work with a custom Jenkins Docker image. A `build.sh` script is included.
-   **Minikube Ready**: Defaults are set for easy local deployment using Minikube.

---

## 🚀 Local Deployment Guide for Minikube

This guide provides instructions for deploying the Jenkins chart to a local Minikube cluster.

### 1. Prerequisites

-   **Docker**: Install Guide
-   **Minikube**: Install Guide
-   **kubectl**: Install Guide
-   **Helm**: Install Guide

### 2. Start and Configure Minikube

Start a Minikube cluster with sufficient resources and enable the storage provisioner, which is required for persistence.

```bash
# Start Minikube with recommended resources
minikube start --cpus=4 --memory=4Gi

# Enable the storage provisioner for Persistent Volumes
minikube addons enable storage-provisioner

# Verify the cluster is running
minikube status
```

### 3. Build and Load the Jenkins Docker Image

This chart is designed to use a custom Docker image named `jenkins-jcasc`. The build script is located in the `k8s-app/jenkins-jcasc/` directory of the repository.

```bash
# Navigate to the image build directory
cd /path/to/your/repo/k8s-app/jenkins-jcasc

# Run the build script. It will print the full image name.
# e.g., jenkins-jcasc:20231027-123456
./build.sh

# Load the newly built image into Minikube's Docker daemon.
# Replace 'jenkins-jcasc:TAG' with the full image name from the output above.
minikube image load jenkins-jcasc:20231027-123456
```

### 4. Install the Helm Chart

Deploy Jenkins using Helm. You must override the `image.tag` with the one you just built and loaded.

```bash
# Replace '20231027-123456' with your actual image tag.
export IMAGE_TAG="20231027-123456"

# Navigate to the Helm chart directory
cd /path/to/your/repo/k8s-helm-charts/tools/jenkins-jcasc

# Install the Helm chart into a 'jenkins' namespace
helm install my-jenkins . \
  --set image.tag=$IMAGE_TAG \
  --namespace jenkins --create-namespace
```

The output will provide instructions on how to access your Jenkins instance.

### 5. Access Jenkins

The service is exposed via `NodePort` by default.

1.  **Get the Jenkins URL**:

    ```bash
    export NODE_PORT=$(kubectl get --namespace jenkins -o jsonpath="{.spec.ports[0].nodePort}" services my-jenkins)
    export NODE_IP=$(minikube ip)
    echo "Jenkins URL: http://$NODE_IP:$NODE_PORT"
    ```

2.  **Get the Admin Password**:

    The password is auto-generated and stored in a secret.

    ```bash
    kubectl get secret --namespace jenkins my-jenkins-admin -o jsonpath="{.data.jenkins-admin-password}" | base64 -d
    ```

3.  **Login**: Open the URL in your browser and log in with `admin` and the retrieved password.

---

## ⚙️ Configuration

The chart can be configured through the `values.yaml` file.

| Parameter              | Description                                                                                             | Default                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| `image.tag`            | The tag of the `jenkins-jcasc` image to use. **Must be set during installation.**                         | `""`                                   |
| `service.type`         | The type of service to expose Jenkins. `NodePort` is ideal for Minikube.                                | `NodePort`                             |
| `service.nodePort`     | The static port to expose on the node.                                                                  | `30080`                                |
| `persistence.enabled`  | If true, create a PersistentVolumeClaim for Jenkins home.                                               | `true`                                 |
| `persistence.size`     | The size of the persistent volume.                                                                      | `8Gi`                                  |
| `persistence.storageClass` | The storage class for the PVC. `standard` works with the Minikube storage-provisioner addon.          | `standard`                             |
| `jcasc.configScripts`  | A map where keys are filenames and values are the JCasC YAML content.                                   | (example configuration)                |
| `resources`            | CPU/Memory resource requests and limits for the Jenkins pod.                                            | (defined requests/limits)              |

To apply changes to `values.yaml` after installation, use `helm upgrade`:

```bash
helm upgrade my-jenkins . --namespace jenkins
```

---

## 🧹 Cleanup

To uninstall the chart and delete all associated resources:

```bash
helm uninstall my-jenkins --namespace jenkins
```

To stop the Minikube cluster:

```bash
minikube stop
```

To delete the cluster entirely (including all data):

```bash
minikube delete
```