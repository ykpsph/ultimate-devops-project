# Ultimate DevOps Project

A hands-on, end-to-end DevOps learning project built around the **OpenTelemetry Astronomy Shop** microservices demo. The goal was to take a multi-language, multi-service application and take it through the full DevOps lifecycle — from running it locally with Docker, to deploying it on Kubernetes (EKS), provisioning the infrastructure with Terraform, and automating the whole pipeline with CI/CD and GitOps.

## Tech Stack

| Area | Tools |
|---|---|
| Containerization | Docker, Docker Compose |
| Orchestration | Kubernetes, Amazon EKS |
| Networking / Ingress | AWS Load Balancer Controller, ALB Ingress |
| Infrastructure as Code | Terraform (VPC + EKS modules, S3 + DynamoDB remote backend) |
| CI | GitHub Actions |
| CD / GitOps | Argo CD |
| Cloud Provider | AWS (EC2, EKS, IAM, S3, DynamoDB) |

## Project Structure

```
.
├── docker/               # Dockerfiles for each service + docker-compose setup
├── kubernetes/           # Per-service K8s manifests (deploy + svc) and complete-deploy.yaml
├── terraform/            # Terraform notes + eks-install (VPC & EKS modules, remote backend)
├── cicd/                 # GitHub Actions CI pipeline + GitOps/Argo CD notes
├── notes/                # General setup notes (EC2, running the project locally)
├── notes-kubernetes/     # Kubernetes concepts and step-by-step notes
└── README.md
```

## The Application

The demo application is a distributed e-commerce app ("Astronomy Shop") made up of independent microservices communicating over gRPC/HTTP and Kafka, including:

`frontend`, `frontendproxy`, `productcatalog`, `cart`, `checkout`, `payment`, `currency`, `shipping`, `email`, `accounting`, `ad`, `recommendation`, `quote`, `frauddetection`, `imageprovider`, `loadgenerator`, `kafka`, `valkey` (cache), and `flagd` (feature flagging).

Services are written in different languages (Go, Java, Python, .NET, etc.), which made it a good playground for practicing multi-stage, language-specific Docker builds.

## Learning Path

### 1. Docker

- [Build Services](/docker/03-build-services.md) — building the Go-based Product Catalog service locally and understanding language-specific build steps
- [Dockerfile — Java microservice (Ad)](/docker/Dockerfile.Ad.yaml)
- [Dockerfile — Go microservice (Product Catalog)](/docker/Dockerfile.ProductCatalog.yaml) — [notes](/docker/04-dockerfile-Product-Catalog.md) on multi-stage builds
- [Dockerfile — Python microservice (Recommendation)](/docker/Dockerfile.Recommendation.yaml)
- [Docker Compose](/docker/docker-compose.yaml) — spinning up the entire application stack locally with one command
- [Running the project locally](/notes/02-run-the-project-locally.md)

### 2. Kubernetes

- [Kubernetes Resources overview](/notes-kubernetes/14-kubernetes-resources.md) — why Kubernetes over plain containers (scaling/healing, service discovery)
- [Deployment resource structure](/notes-kubernetes/15-deployment-resource-structure.md)
- [Service resource structure](/notes-kubernetes/16-service-resource-structure.md)
- [Service types](/notes-kubernetes/17-kubernetes-service-types.md) — ClusterIP, NodePort, LoadBalancer
- [Deploying on EKS](/notes-kubernetes/18-deploying-on-EKS.md)
- [Disadvantages of the LoadBalancer service type](/notes-kubernetes/19-disadvantages-of-LB-service-type.md)
- [Ingress and Ingress Controller](/notes-kubernetes/20-ingress-and-ingress-controller.md)
- [Deploying the AWS Load Balancer (ALB) Ingress Controller](/notes-kubernetes/21-deploying-ALB-ingress-controller.md)
- [Setting up Ingress resources](/notes-kubernetes/22-setup-ingress-resources.md)
- [Service Accounts](/notes-kubernetes/13-kubernetes-service-account.md)
- [Connecting to a Kubernetes cluster (kubeconfig/context)](/notes-kubernetes/12-connect-to-Kubernetes-cluster.md)
- Manifests: [per-service manifests](/kubernetes/) and the [full deployment file](/kubernetes/complete-deploy.yaml)

### 3. Terraform

- [Terraform Lifecycle](/terraform/05.terraform-lifecycle.md) — init, plan, apply
- [AWS provider configuration](/terraform/06-terraform-AWS-configuration.md)
- [State file basics](/terraform/07-terraform-statefile.md)
- [State file management](/terraform/08-terraform-statefile-management.md) — why local state breaks down with multiple contributors
- [Remote backend using S3](/terraform/09-terraform-remote-backend-using-s3.md)
- [State locking using DynamoDB](/terraform/10-terraform-state-locking-using-dynamodb.md)
- [Creating the S3 bucket and DynamoDB table](/terraform/11-terraform-create-S3bucket-and-dynamodb.md)
- [EKS + VPC module code](/terraform/eks-install/) — reusable `vpc` and `eks` modules with a remote backend

### 4. CI/CD & GitOps

- [CI concepts](/cicd/23-cicd.md) — checkout → test → build → image → scan → push → update manifest
- [Product Catalog CI pipeline (GitHub Actions)](/cicd/24-Product-Catalog-Service-ci.yaml.md) ([workflow file](/cicd/ProductCatalog-ci.yaml))
- [GitOps principles](/cicd/25-GitOps.md) — keeping the desired cluster state in Git and letting a controller reconcile it
- [Argo CD setup and usage](/cicd/26-ArgoCD.md) — installation, exposing the UI, retrieving the admin password, and connecting a Git repo as an application

## How It Fits Together

```
Developer pushes code
        │
        ▼
   CI (GitHub Actions)
   build → test → docker build → scan → push image → update K8s manifest
        │
        ▼
     Git repo (desired state)
        │
        ▼
      Argo CD  ── watches the repo, detects drift
        │
        ▼
  Kubernetes cluster (EKS)
   reconciled to match the manifest — new version deployed
```

Infrastructure (VPC + EKS cluster) is provisioned ahead of time with Terraform, using an S3 backend for remote state and a DynamoDB table for state locking, so the setup is safe for multiple contributors.

## Running It Locally

```bash
git clone https://github.com/iam-veeramalla/ultimate-devops-project-demo.git
cd ultimate-devops-project-demo
docker compose up
```

See [notes/02-run-the-project-locally.md](/notes/02-run-the-project-locally.md) for details, including common gotchas (e.g. disk space on a fresh EC2 instance and opening the right security group ports).

## Deploying to EKS

```bash
# 1. confirm you're pointed at the right cluster
kubectl config current-context

# 2. create the service account
kubectl apply -f kubernetes/serviceaccount.yaml

# 3. deploy every microservice
kubectl apply -f kubernetes/complete-deploy.yaml

# 4. check status
kubectl get all
kubectl get svc
```

Once the AWS Load Balancer Controller and an Ingress resource are set up (see the Kubernetes section above), the app becomes reachable through an ALB instead of a per-service LoadBalancer.

## Key Takeaways

- **Docker**: multi-stage builds keep images small and language-agnostic across a polyglot microservice stack.
- **Kubernetes**: Deployments give self-healing/scaling; Services provide stable networking; Ingress + ALB Controller avoid provisioning a separate cloud Load Balancer per service.
- **Terraform**: remote state (S3) + locking (DynamoDB) are what make IaC safe for teams, not just individuals.
- **CI/CD**: separating CI (build/test/push image) from CD (deploy to cluster) is what GitOps is built on — Argo CD continuously reconciles the cluster to match what's declared in Git, instead of anyone running `kubectl apply` by hand.

## Credits

Based on the [Ultimate DevOps Project](https://github.com/iam-veeramalla/ultimate-devops-project-demo) by [@iam-veeramalla](https://github.com/iam-veeramalla), which itself uses the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo) application as its sample microservices app.
