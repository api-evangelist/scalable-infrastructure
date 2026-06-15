# Scalable Infrastructure (scalable-infrastructure)

A subject-matter collection covering APIs, tools, and platforms for building and managing scalable cloud infrastructure. This topic encompasses compute, storage, networking, container orchestration, infrastructure as code (IaC), monitoring, and the major cloud providers (AWS, Azure, GCP, DigitalOcean) that power modern scalable systems.

**APIs.json:** [https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/refs/heads/main/apis.yml)

## Tags

- Cloud Infrastructure
- Compute
- DevOps
- Infrastructure as Code
- Kubernetes
- Networking
- Scalability
- Storage

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-05-02

## APIs

### AWS EC2 API

Amazon Elastic Compute Cloud (EC2) provides resizable compute capacity in the cloud. The EC2 API manages instances, AMIs, security groups, VPCs, elastic IP addresses, placement groups, and auto-scaling groups. The most widely used compute API with a 31% global cloud market share.

#### Tags

- Amazon Web Services
- Cloud
- Compute
- EC2
- Infrastructure
- Instances

#### Properties

- [Documentation](https://docs.aws.amazon.com/ec2/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/ec2/2016-11-15/openapi.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://aws.amazon.com/ec2/pricing/)
- [Getting Started](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [SDK](https://aws.amazon.com/developer/tools/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### AWS VPC API

Amazon Virtual Private Cloud (VPC) enables launching AWS resources in a logically isolated virtual network. The VPC API manages subnets, route tables, internet gateways, NAT gateways, VPC peering, PrivateLink, and network ACLs. Foundational networking for scalable AWS architectures.

#### Tags

- Amazon Web Services
- Cloud
- Infrastructure
- Networking
- Security
- VPC

#### Properties

- [Documentation](https://docs.aws.amazon.com/vpc/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/ec2/2016-11-15/openapi.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://aws.amazon.com/vpc/pricing/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### AWS EKS API

Amazon Elastic Kubernetes Service (EKS) is a managed Kubernetes service. The EKS API manages clusters, node groups, Fargate profiles, add-ons, identity providers, and access entries. Powers scalable containerized workloads on AWS with automatic Kubernetes control plane management.

#### Tags

- Amazon Web Services
- Cloud
- Containers
- EKS
- Kubernetes
- Managed Service

#### Properties

- [Documentation](https://docs.aws.amazon.com/eks/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/eks/2018-08-23/openapi.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://aws.amazon.com/eks/pricing/)
- [Getting Started](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Google Compute Engine API

Google Compute Engine provides virtual machines running in Google's infrastructure. The API manages instances, machine types, disks, snapshots, networks, firewall rules, and managed instance groups. Includes powerful autoscaler and load balancing integration. Google Cloud holds ~12% global cloud market share.

#### Tags

- Cloud
- Compute
- Google Cloud
- Infrastructure
- Instances
- Virtual Machines

#### Properties

- [Documentation](https://cloud.google.com/compute/docs)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/googleapis.com/compute/v1/openapi.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://cloud.google.com/compute/pricing)
- [SDK](https://cloud.google.com/sdk/docs)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Google Kubernetes Engine API

Google Kubernetes Engine (GKE) is a managed, production-ready Kubernetes service with auto-upgrade, auto-repair, cluster autoscaling, Workload Identity, and Autopilot mode. The GKE API manages clusters, node pools, and operations.

#### Tags

- Cloud
- Containers
- GKE
- Google Cloud
- Kubernetes
- Managed Service

#### Properties

- [Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/googleapis.com/container/v1/openapi.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://cloud.google.com/kubernetes-engine/pricing)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Azure Resource Manager API

Azure Resource Manager (ARM) is the deployment and management service for Azure. The REST API provides a consistent way to create, update, delete, and manage Azure resources across all services including VMs, storage, networking, and databases. Microsoft Azure holds ~28% global cloud market share.

#### Tags

- Azure
- Cloud
- Infrastructure
- Management
- Microsoft
- Resource Management

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/rest/api/resources/)
- [OpenAPI](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/resources/resource-manager/Microsoft.Resources/stable/2023-07-01/resources.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [SDK](https://learn.microsoft.com/en-us/azure/developer/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Azure AKS API

Azure Kubernetes Service (AKS) simplifies deploying, managing, and scaling containerized applications using Kubernetes. The AKS API manages clusters, agent pools, node pools, maintenance configurations, and trusted access bindings.

#### Tags

- AKS
- Azure
- Cloud
- Containers
- Kubernetes
- Managed Service

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/rest/api/aks/)
- [OpenAPI](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/containerservice/resource-manager/Microsoft.ContainerService/aks/stable/2024-01-01/managedClusters.json) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://azure.microsoft.com/en-us/pricing/details/kubernetes-service/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### DigitalOcean API

DigitalOcean's developer-friendly cloud platform API manages Droplets (VMs), Kubernetes clusters, databases, object storage (Spaces), load balancers, VPCs, firewalls, and app platform deployments. Known for simplicity, transparent pricing, and developer experience for digital-native applications.

#### Tags

- Cloud
- Compute
- Developer-Friendly
- DigitalOcean
- Infrastructure
- Kubernetes

#### Properties

- [Documentation](https://docs.digitalocean.com/reference/api/)
- [OpenAPI](https://raw.githubusercontent.com/digitalocean/openapi/main/specification/DigitalOcean-public.v2.yaml) — [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Pricing](https://www.digitalocean.com/pricing)
- [Getting Started](https://docs.digitalocean.com/products/getting-started/)
- [SDK](https://github.com/digitalocean/doctl)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Terraform Registry API

The Terraform Registry API provides access to infrastructure-as-code (IaC) modules and providers. HashiCorp Terraform is the leading IaC tool for provisioning and managing cloud infrastructure in a declarative, version-controlled way. Critical to scalable infrastructure automation workflows.

#### Tags

- Infrastructure as Code
- IaC
- Open Source
- Provisioning
- Terraform

#### Properties

- [Documentation](https://developer.hashicorp.com/terraform/docs)
- [Git Hub](https://github.com/hashicorp/terraform)
- [API Reference](https://developer.hashicorp.com/terraform/cloud-docs/api-docs)
- [Registry](https://registry.terraform.io/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

### Pulumi Cloud API

Pulumi is a modern infrastructure as code platform that uses general-purpose programming languages (TypeScript, Python, Go, C#, Java, YAML). The Pulumi Cloud API manages stacks, deployments, environments, and audit logs. Alternative to Terraform with full-language programming model.

#### Tags

- Cloud
- Infrastructure as Code
- IaC
- Open Source
- Provisioning
- Pulumi

#### Properties

- [Documentation](https://www.pulumi.com/docs/)
- [Git Hub](https://github.com/pulumi/pulumi)
- [API Reference](https://www.pulumi.com/docs/pulumi-cloud/cloud-rest-api/)
- [Pricing](https://www.pulumi.com/pricing/)
- [Postman Collection](collections/scalable-infrastructure.postman_collection.json) — [Postman Collection 2.1](https://schema.getpostman.com/json/collection/v2.1.0/collection.json)
- [Open Collection](collections/scalable-infrastructure.opencollection.json) — [Open Collection 1.0](https://schema.opencollection.com/opencollection/v1.0.0.json)

## Common Properties

- [GitHub Organization](https://github.com/hashicorp)
- [C N C F  Landscape](https://landscape.cncf.io/card-mode?category=provisioning)
- [Blog](https://www.cncf.io/blog/)
- [JSON Schema](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/main/json-schema/scalable-infrastructure-compute-instance-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON Schema](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/main/json-schema/scalable-infrastructure-kubernetes-cluster-schema.json) — [JSON Schema](https://json-schema.org/specification)
- [JSON-LD](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/main/json-ld/scalable-infrastructure-context.jsonld) — [JSON-LD](https://www.w3.org/TR/json-ld11/)
- [Vocabulary](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/main/vocabulary/scalable-infrastructure-vocabulary.yml)

## Maintainers

**FN:** API Evangelist
**Email:** kin@apievangelist.com
**URL:** https://apievangelist.com
