# DevSecOps GitOps Repository

This repository contains the Kubernetes manifests used to deploy and manage the Java Spring Boot application in the DevSecOps project. It is separate from the application and CI/CD repository.

## Purpose

The application source code and Jenkins pipeline are maintained in the [application repository](https://github.com/YoussefJammoussi/devsecops-cicd-pipeline-java-springboot). This repository holds the deployment configuration that Argo CD tracks to synchronize the desired application state with the K3s cluster.

## Deployment workflow

1. Jenkins builds and tests the application.
2. Docker builds the application image.
3. Trivy scans the image as part of the CI/CD pipeline.
4. Jenkins pushes the image to Nexus Repository.
5. The deployment configuration in this repository is updated to reference the desired image tag.
6. Argo CD detects the Git change and synchronizes the Kubernetes resources to the K3s cluster.
7. Kubernetes pulls the image from Nexus and runs the application.

## Repository structure

The exact structure depends on the manifests currently committed to this repository. A typical layout may look like this:

```text
devsecops-gitops/
└── apps/
    └── my-app/
        ├── deployment.yaml
        └── service.yaml
```

Use the actual committed file and directory names when navigating the repository.

## Application deployment

The `my-app` application is deployed to the `my-app` Kubernetes namespace. Its Kubernetes Deployment references a versioned container image stored in Nexus Repository. The cluster uses an image pull secret to authenticate to the registry when required.

Argo CD is responsible for synchronizing the GitOps configuration. Check the Argo CD application status to verify whether the application is synchronized and healthy.

## Updating the application image

When a new application image has been published to Nexus:

1. Update the image reference in the relevant Kubernetes Deployment manifest in this repository.
2. Commit and push the change to the Git repository.
3. Let Argo CD synchronize the change, or initiate a sync if automatic synchronization is not enabled.
4. Verify the application status in Argo CD and check the running pods in the `my-app` namespace.

Example commands for checking the deployment:

```bash
kubectl get applications -n argocd
kubectl get deployments,pods -n my-app
kubectl get services -n my-app
```

If your environment requires elevated permissions or a specific kubeconfig, use the corresponding `sudo kubectl` command or configured Kubernetes context.

## Security and configuration

- Do not commit registry passwords, access tokens, private keys, or Kubernetes Secret values containing credentials.
- Use Kubernetes Secrets or an appropriate secret-management solution for registry authentication.
- Keep the image tag in the Deployment manifest aligned with the image published by the CI/CD pipeline.
- Review image scan results before promoting an image to deployment.

## Related repository

- **Application and CI/CD pipeline:** [devsecops-cicd-pipeline-java-springboot](https://github.com/YoussefJammoussi/devsecops-cicd-pipeline-java-springboot)

## Author

**Youssef Jammoussi**
