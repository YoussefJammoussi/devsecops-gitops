# DevSecOps GitOps Repository

This repository contains the Kubernetes deployment configuration for the Java Spring Boot application used in the DevSecOps project. It is maintained separately from the application source code and CI/CD pipeline.

## Purpose

The application source code and Jenkins pipeline are maintained in the [application and CI/CD repository](https://github.com/YoussefJammoussi/devsecops-cicd-pipeline-java-springboot).

This repository is used to store the Kubernetes configuration tracked by Argo CD. Argo CD watches the configured Git repository and synchronizes the declared application state with the K3s cluster.

## Deployment workflow

1. Jenkins builds and tests the Spring Boot application.
2. Docker builds the application container image.
3. Trivy scans the image as part of the CI/CD pipeline.
4. Jenkins publishes the image to Nexus Repository.
5. The Kubernetes configuration in this repository is updated to reference the intended image tag.
6. Argo CD synchronizes the Git configuration with the K3s cluster.
7. Kubernetes pulls the image from Nexus and runs the application.

## Repository contents

- `apps/` — application deployment configuration. The exact manifest names and subdirectories are defined by the files committed in this directory.
- `docs/k3s/registries.yaml.example` — example configuration for configuring K3s to work with the Nexus registry over HTTP.
- `README.md` — documentation for this GitOps repository.

## Application deployment

The application is deployed in the `my-app` Kubernetes namespace. Its Kubernetes Deployment references a versioned container image published to Nexus Repository. Where registry authentication is required, Kubernetes uses an image pull secret.

Argo CD is used to synchronize the desired Kubernetes configuration from this repository. The Argo CD application status and the Kubernetes workload status can be checked to confirm synchronization and application health.

## Updating the deployed image

When a new application image has been published to Nexus:

1. Update the image reference in the relevant Kubernetes manifest under `apps/`.
2. Commit and push the change to the configured Git branch.
3. Allow Argo CD to synchronize the change, or trigger a sync if automatic synchronization is not enabled.
4. Verify the application status in Argo CD and check the Kubernetes workloads.

Example commands:

```bash
kubectl get applications -n argocd
kubectl get deployments,pods -n my-app
kubectl get services -n my-app
```

Use the Kubernetes context and permissions configured for your environment. If your setup requires elevated privileges, prefix the commands with `sudo`.

## K3s registry configuration example

The file `docs/k3s/registries.yaml.example` documents an example registry configuration for K3s and Nexus. Treat it as a template: review it for your environment before using it, and do not place credentials or secrets in a public repository.

## Security notes

- Do not commit registry passwords, access tokens, private keys, or secret values.
- Use Kubernetes Secrets or an appropriate secret-management solution for registry authentication.
- Keep the image tag in the Kubernetes manifest aligned with the image published by the CI/CD pipeline.
- Review image scan results before deploying a new image.

## Related repository

- **Application and CI/CD pipeline:** [devsecops-cicd-pipeline-java-springboot](https://github.com/YoussefJammoussi/devsecops-cicd-pipeline-java-springboot)

## Author

**Youssef Jammoussi**
