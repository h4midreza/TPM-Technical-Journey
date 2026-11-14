# Topic 7: DevOps, CI/CD & Containerized Infrastructure

## 1. Objective

Understand how Data Platform components are developed, packaged, deployed, configured, and operated within containerized infrastructure.

This topic focuses on DevOps practices, automated software delivery, container technologies, Kubernetes/OpenShift architecture, and infrastructure dependencies.

The goal is to understand deployment processes, operational requirements, technical dependencies, and potential failure scenarios from a Technical Program Manager (TPM) perspective.

## 2. Practical Scenario: Data Platform Deployment

Assume a Data Platform consists of several components:

| Component | Responsibility | Deployment Consideration |
|---|---|---|
| Metadata API | Handles application requests and document metadata | Application Deployment |
| Document Processor | Processes documents and extracts metadata | Requires network and storage access |
| PostgreSQL | Stores operational data | Requires persistent storage |
| Kafka | Transfers and retains events | Requires persistent storage and cluster configuration |
| MinIO | Stores document files | Requires persistent storage |

These components may run on Kubernetes or OpenShift.

For example, when a new version of Metadata API is released, the delivery process includes building the application, creating a container image, publishing it to a registry, and deploying it to the cluster.

The deployment also depends on configuration, available resources, network connectivity, and external services.

## 3. DevOps Principles and Software Development Lifecycle

### DevOps

DevOps combines Development and Operations practices to improve collaboration, automation, and software delivery.

Its main principles include:

- **Collaboration:** Development, Operations, and Platform teams share responsibility.
- **Automation:** Build, testing, and deployment processes are automated.
- **Continuous Feedback:** Monitoring and production feedback improve software quality.
- **Consistency:** Standardized processes reduce differences between environments.

For example, when a new application version cannot connect to PostgreSQL:

- Development investigates application code and connection handling.
- Platform investigates deployment configuration and environment variables.
- Database investigates availability, credentials, and connection limits.
- Network investigates DNS, firewall, and connectivity.

### Software Development Lifecycle (SDLC)

SDLC describes the main stages of software development and maintenance:

1. Planning and requirements
2. Development
3. Code Review
4. Testing
5. Build and packaging
6. Release
7. Deployment
8. Monitoring and maintenance

Changes to application code may also require database schema migrations. These changes must be coordinated to maintain compatibility between application versions and database structures.

## 4. Git, Version Control, and Configuration Management

### Git and Version Control

Git tracks changes to source code and configuration files and supports collaboration between developers.

| Concept | Purpose |
|---|---|
| Repository | Stores project source code and related files |
| Commit | Records a specific change |
| Branch | Supports independent development |
| Merge | Combines changes |
| Pull Request | Supports review and approval |
| Tag | Identifies a specific release version |

Example workflow:

```text
main
  |
  +-- feature/new-metadata-field
  |              |
  |          Development
  |              |
  |          Pull Request
  |              |
  +---------- Merge
                 |
              Release
                 |
               v1.8.2
```

### Configuration Management

Configuration Management defines how application and infrastructure settings are maintained across environments.

For example, the same application may connect to different PostgreSQL endpoints in Development, Testing, and Production.

Configuration files can be maintained in Git to provide version history and traceability.

Sensitive values such as passwords and API tokens should not be committed as plain text into source repositories.

## 5. CI/CD and Application Deployment

CI/CD automates software integration, validation, packaging, release, and deployment.

### Continuous Integration (CI)

Code changes are frequently integrated into a shared repository and validated automatically.

A typical CI pipeline:

1. Developer pushes code to Git.
2. Automated build starts.
3. Tests and code-quality checks run.
4. Security checks are performed.
5. A container image is created.

### Continuous Delivery vs. Continuous Deployment

- **Continuous Delivery:** Software is automatically built and validated and remains ready for release. Production deployment normally requires approval.
- **Continuous Deployment:** Changes that pass the required checks can be deployed automatically to production.

### Common Tools

| Tool | Main Purpose |
|---|---|
| Jenkins | Build, test, and deployment automation |
| GitLab CI/CD | Integrated source control and CI/CD |
| GitHub Actions | Repository-based workflow automation |
| Argo CD | GitOps-based Kubernetes deployment |
| Tekton | Kubernetes-native CI/CD pipelines |

### Build, Test, Release, Deployment, and Rollback

These stages move an application from source code to a running service.

- **Build:** Compile and package the application, for example as a Java JAR.
- **Test:** Run unit, integration, API, and security tests.
- **Image Build:** Package the application into a container image.
- **Release:** Identify an approved application version.
- **Deployment:** Run the new image in the target environment.
- **Rollback:** Return to a previously working version when the new deployment causes problems.

Example:

```text
metadata-api:v1.8.1  Previous Version
metadata-api:v1.8.2  New Version
```

Kubernetes supports rollout and rollback operations for Deployments.

**Important:** Rollback is not always as simple as restoring the previous image. Database schema changes must remain compatible, and persistent data should not be reverted unintentionally.

## 6. Containers and Container Images

### Container

A Container is an isolated runtime environment that executes an application together with its required dependencies.

### Container Image

A Container Image is the packaged template used to create containers. It typically includes application files, runtime dependencies, and startup instructions.

Example:

```text
Container Image
  |
  +-- Java Runtime
  +-- Application JAR
  +-- Required Libraries
  +-- Startup Command
```

### Common Container Concepts

| Concept | Responsibility |
|---|---|
| Image | Packaged application template |
| Container | Running instance of an image |
| Container Registry | Stores and distributes images |
| Container Runtime | Starts and manages containers |

Common technologies:

- Docker: Commonly used to build and manage container images.
- containerd and CRI-O: Container runtimes commonly used in Kubernetes environments.
- Harbor: A container registry that can store and manage images.

### Practical Considerations

- Images should have identifiable versions.
- Production images should be scanned and approved.
- Images should be treated as immutable artifacts.
- Environment-specific settings should be provided through runtime configuration.
- Important application data should not depend on a container's temporary writable filesystem.

## 7. Kubernetes and OpenShift Architecture

### Kubernetes

Kubernetes is a container orchestration platform that automates application deployment, scheduling, scaling, and recovery.

It manages a group of machines called a Cluster and continuously works to maintain the desired state of deployed applications.

### OpenShift

OpenShift is an enterprise application platform built on Kubernetes.

It provides additional capabilities such as:

- Web Console and developer tools
- Integrated security and access controls
- Routes for external application access
- Operators for managing platform services

OpenShift uses Kubernetes concepts such as Pods, Deployments, Services, and Namespaces.

### Kubernetes Architecture

A Kubernetes Cluster has two main parts:

**Control Plane**

Manages cluster state and coordinates workloads.

| Component | Responsibility |
|---|---|
| API Server | Receives Kubernetes API requests |
| etcd | Stores cluster configuration and state |
| Scheduler | Selects Nodes for new Pods |
| Controller Manager | Maintains the desired cluster state |

**Worker Nodes**

Run application workloads.

| Component | Responsibility |
|---|---|
| Kubelet | Manages Pods on the Node |
| Container Runtime | Runs containers |
| Network Components | Enable communication between workloads |

## 8. Cluster, Node, Pod, Deployment, and Namespace

These are fundamental Kubernetes building blocks.

| Concept | Description |
|---|---|
| Cluster | Complete Kubernetes environment |
| Node | Physical or virtual machine that runs workloads |
| Pod | Smallest deployable unit, containing one or more containers |
| Deployment | Manages application replicas and updates |
| ReplicaSet | Maintains the desired number of Pods |
| Namespace | Logically separates resources within a Cluster |

Example:

```text
Kubernetes Cluster
   |
   +-- Namespace: dms-prod
          |
          +-- Deployment: metadata-api
                  |
                  +-- ReplicaSet
                         |
                         +-- Pod 1
                         +-- Pod 2
                         +-- Pod 3
```

If a Deployment specifies three replicas and one Pod fails, Kubernetes attempts to create a replacement.

Pods are temporary and may be recreated during failures, scaling, or updates.

## 9. Kubernetes Services and Service Discovery

Pods may be replaced and receive different IP addresses. Applications therefore need a stable way to communicate with them.

A Kubernetes Service provides a stable network endpoint for a group of Pods selected through labels.

### Common Service Types

| Type | Purpose |
|---|---|
| ClusterIP | Internal communication within the Cluster |
| NodePort | Exposes a Service through a port on cluster Nodes |
| LoadBalancer | Exposes a Service through an external load balancer when supported |

### Service Discovery

Kubernetes provides DNS-based service discovery.

For example:

```text
postgresql.dms-prod.svc.cluster.local
```

Applications can use this DNS name instead of depending on individual Pod IP addresses.

For external access:

- Kubernetes commonly uses Ingress or Gateway API.
- OpenShift provides Route resources.

A Service provides stable access to Pods; a Route or Ingress provides an external entry point.

## 10. Configuration, Resources, and Persistent Storage

### ConfigMaps and Secrets

Kubernetes separates application configuration from container images.

**ConfigMap:** Stores non-sensitive settings such as database hostnames, ports, and application profiles.

**Secret:** Stores sensitive information such as passwords, API tokens, and certificates.

Example:

```text
metadata-api-config
  DB_HOST
  DB_PORT
  APP_PROFILE

metadata-api-secret
  DB_USERNAME
  DB_PASSWORD
```

Secret data is not automatically encrypted simply because it is stored as a Kubernetes Secret. Base64 encoding is not encryption. Production environments require appropriate encryption, access controls, and credential rotation.

### Resource Requests and Limits

Requests and Limits control CPU and memory usage.

- **Requests:** Resources requested by a container. The Scheduler uses these values when selecting a Node.
- **Limits:** Maximum permitted resource consumption.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "512Mi"
  limits:
    cpu: "1"
    memory: "1Gi"
```

CPU limits can cause throttling. Exceeding a memory limit can result in an OOMKilled container.

Resource values should be based on actual workload measurements.

### Persistent Volumes and Persistent Volume Claims

Containers and Pods are temporary. Persistent data must be stored outside their temporary writable filesystems.

- **Persistent Volume (PV):** Represents storage available to the Cluster.
- **Persistent Volume Claim (PVC):** A workload's request for persistent storage.
- **StorageClass:** Defines how storage volumes are provisioned.

Example:

```text
PostgreSQL Pod
      |
      v
     PVC
      |
      v
     PV
      |
      v
 Storage System
```

For example, PostgreSQL may use a persistent volume backed by Ceph RBD or another supported storage system.

A StorageClass can define different storage types, such as SSD-backed or general-purpose storage.

Persistent storage is not a replacement for independent backups.

### Readiness, Liveness, and Startup Probes

Kubernetes uses health checks to manage application availability.

| Probe | Purpose | Failure Impact |
|---|---|---|
| Readiness | Checks whether the application can receive traffic | Pod is removed from Service endpoints |
| Liveness | Checks whether the application is functioning | Container may be restarted |
| Startup | Allows extra time for application initialization | Other probes wait until startup succeeds |

Example: An application may take 90 seconds to initialize. A Startup Probe prevents Kubernetes from treating it as unhealthy before it has finished starting.

Incorrect probe configuration can cause unnecessary restarts or route traffic to an application that is not ready.

## 11. Environment Configuration, Configuration Drift, and IaC

### Environment Configuration

Applications commonly run in Development, Testing, Staging, and Production environments.

The same application image can be deployed across environments with different configuration values.

| Setting | Development | Production |
|---|---|---|
| Database | Development DB | Production DB |
| Log Level | DEBUG | INFO |
| Replicas | 1 | 3 |
| Resource Limits | Lower | Based on workload |

### Configuration Drift

Configuration Drift occurs when the actual environment differs from its approved or declared configuration.

Example:

Git defines three application replicas, but someone manually changes the live Deployment to five replicas without updating the version-controlled configuration.

This creates differences between the declared and actual state.

Common prevention methods:

- Version-controlled configuration.
- Controlled approval of changes.
- Avoiding undocumented manual modifications.
- GitOps and automated configuration reconciliation.

### Infrastructure as Code (IaC)

IaC means defining and managing infrastructure through version-controlled configuration instead of relying entirely on manual operations.

Common tools:

| Tool | Main Purpose |
|---|---|
| Terraform / OpenTofu | Infrastructure provisioning |
| Ansible | Server configuration and automation |
| Helm | Kubernetes application packaging |
| Kustomize | Managing Kubernetes configuration variations |

For example, Terraform can provision infrastructure, while Helm can deploy an application and its Kubernetes resources.

These tools serve different purposes and can work together.

## 12. Infrastructure Dependencies and Deployment Readiness

A successful deployment requires more than a valid application image.

Applications depend on infrastructure, networking, storage, and external services.

### Infrastructure Dependency Matrix

| Dependency | Purpose | Potential Problem |
|---|---|---|
| Container Registry | Provides application images | ImagePullBackOff |
| CPU and Memory | Runs application workloads | Pod Pending or OOMKilled |
| Network and DNS | Connects application components | Connection timeout |
| Database | Provides operational data | Connection or authentication failure |
| Persistent Storage | Stores stateful application data | PVC Pending |
| Secrets and Credentials | Provides secure access | Authentication failure |
| Service Account / RBAC | Controls permissions | Forbidden access |
| Certificates | Supports secure connections | TLS errors |

### Deployment Readiness

Before production deployment, verify:

- Approved container image is available.
- Target Namespace and permissions exist.
- ConfigMaps and Secrets are configured.
- Required storage claims are bound.
- Network and DNS connectivity are available.
- Database and external services are reachable.
- Sufficient cluster resources exist.
- Monitoring and rollback procedures are ready.

Deployment readiness requires coordination between Development, Platform, Database, Storage, and Network teams.

## 13. Practical Kubernetes Deployment Example

The following sample shows a simplified Metadata API Deployment.

### Sample Deployment Manifest

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metadata-api
  namespace: dms-prod

spec:
  replicas: 3

  selector:
    matchLabels:
      app: metadata-api

  template:
    metadata:
      labels:
        app: metadata-api

    spec:
      containers:
        - name: metadata-api

          image: registry.example.com/dms/metadata-api:v1.8.2

          ports:
            - containerPort: 8080

          env:
            - name: DB_HOST
              valueFrom:
                configMapKeyRef:
                  name: metadata-api-config
                  key: DB_HOST

            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: metadata-api-secret
                  key: DB_PASSWORD

          resources:
            requests:
              cpu: "250m"
              memory: "512Mi"
            limits:
              cpu: "1"
              memory: "1Gi"

          startupProbe:
            httpGet:
              path: /actuator/health
              port: 8080

          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080

          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
```

### Key Configuration Fields

| Field | Meaning |
|---|---|
| replicas | Desired number of Pods |
| image | Container image and version |
| namespace | Target environment |
| ConfigMap | Non-sensitive configuration |
| Secret | Sensitive configuration |
| requests | Resources requested for scheduling |
| limits | Maximum permitted resources |
| startupProbe | Checks application startup |
| readinessProbe | Checks traffic readiness |
| livenessProbe | Checks application health |

The referenced ConfigMap and Secret must exist, and the health endpoints must be implemented by the application.

A Service must also be configured to provide stable network access to the Pods.

## 14. Deployment Lifecycle: From Git to Running Pod

A typical deployment process:

```text
Developer Changes Code
          |
          v
       Git Commit
          |
          v
      Pull Request
          |
          v
     Code Review
          |
          v
      CI Pipeline
       /   |   \
   Build  Test  Scan
          |
          v
    Container Image
          |
          v
    Container Registry
          |
          v
   Deployment Pipeline
          |
          v
    Kubernetes API
          |
          v
      Deployment
          |
          v
       ReplicaSet
          |
          v
          Pods
          |
          v
   Startup / Readiness
          |
          v
     Service Traffic
```

The main stages are:

1. Code is committed and reviewed.
2. CI builds and validates the application.
3. A versioned container image is created and published.
4. The deployment process updates the Kubernetes configuration.
5. Kubernetes schedules Pods on suitable Nodes.
6. The container runtime starts the application.
7. Health checks determine whether the application is ready.
8. The Service routes traffic to available Pods.

A Pod existing in the Cluster does not necessarily mean the application is ready for use.

## 15. Deployment Failure Scenarios and Troubleshooting

| Failure | Possible Cause | Investigation |
|---|---|---|
| ImagePullBackOff | Image missing or registry authentication failure | Check image tag and registry access |
| Pod Pending | Insufficient resources or storage issue | Check Node capacity and PVC |
| CrashLoopBackOff | Application repeatedly exits | Inspect logs and configuration |
| OOMKilled | Memory limit exceeded | Check memory usage and limits |
| Readiness Probe Failed | Application not ready | Check health endpoint and dependencies |
| PVC Pending | Storage provisioning failure | Check StorageClass and storage system |
| Database Connection Error | Wrong endpoint, credentials, or network | Check DNS, Secret, and connectivity |
| Service Unreachable | Incorrect selector or network configuration | Check Service endpoints and network policies |

### Useful Troubleshooting Commands

```bash
kubectl get pods -n dms-prod

kubectl describe pod <pod-name> -n dms-prod

kubectl logs <pod-name> -n dms-prod

kubectl get deployments -n dms-prod

kubectl get services -n dms-prod

kubectl get pvc -n dms-prod

kubectl rollout status deployment/metadata-api -n dms-prod

kubectl rollout history deployment/metadata-api -n dms-prod
```

These commands help inspect workload status, application logs, resource events, storage claims, and deployment history.

OpenShift provides corresponding `oc` commands for many Kubernetes operations.

