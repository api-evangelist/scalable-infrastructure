# Scalable Infrastructure

A subject-matter collection covering APIs, tools, and platforms for building and managing scalable cloud infrastructure. Encompasses compute, storage, networking, container orchestration, infrastructure as code (IaC), and the major cloud providers (AWS, Azure, GCP, DigitalOcean) that power modern scalable systems.

**URL:** [https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/refs/heads/main/apis.yml](https://raw.githubusercontent.com/api-evangelist/scalable-infrastructure/refs/heads/main/apis.yml)

## Tags

Cloud Infrastructure, Compute, DevOps, Infrastructure as Code, Kubernetes, Networking, Scalability, Storage

## Timestamps

- **Created:** 2024-01-15
- **Modified:** 2026-05-02

## APIs

### AWS EC2 API
Most widely used compute API; manages instances, AMIs, VPCs, security groups, and auto-scaling groups on AWS with 31% global cloud market share.

**Human URL:** [https://aws.amazon.com/ec2/](https://aws.amazon.com/ec2/)

#### Tags

Amazon Web Services, Cloud, Compute, EC2, Infrastructure, Instances

#### Properties

- [Documentation](https://docs.aws.amazon.com/ec2/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/ec2/2016-11-15/openapi.yaml)
- [Pricing](https://aws.amazon.com/ec2/pricing/)

### AWS VPC API
Foundational networking for AWS; manages subnets, route tables, internet gateways, NAT gateways, and VPC peering.

**Human URL:** [https://aws.amazon.com/vpc/](https://aws.amazon.com/vpc/)

#### Tags

Amazon Web Services, Cloud, Infrastructure, Networking, Security, VPC

#### Properties

- [Documentation](https://docs.aws.amazon.com/vpc/)
- [Pricing](https://aws.amazon.com/vpc/pricing/)

### AWS EKS API
Amazon managed Kubernetes service; API for clusters, node groups, Fargate profiles, and add-ons.

**Human URL:** [https://aws.amazon.com/eks/](https://aws.amazon.com/eks/)

#### Tags

Amazon Web Services, Cloud, Containers, EKS, Kubernetes, Managed Service

#### Properties

- [Documentation](https://docs.aws.amazon.com/eks/)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/amazonaws.com/eks/2018-08-23/openapi.yaml)
- [Pricing](https://aws.amazon.com/eks/pricing/)

### Google Compute Engine API
Google Cloud's VM service with powerful autoscaler and load balancing integration; 12% global market share.

**Human URL:** [https://cloud.google.com/compute](https://cloud.google.com/compute)

#### Tags

Cloud, Compute, Google Cloud, Infrastructure, Instances, Virtual Machines

#### Properties

- [Documentation](https://cloud.google.com/compute/docs)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/googleapis.com/compute/v1/openapi.yaml)
- [Pricing](https://cloud.google.com/compute/pricing)

### Google Kubernetes Engine API
GKE managed Kubernetes with auto-upgrade, cluster autoscaling, Workload Identity, and Autopilot mode.

**Human URL:** [https://cloud.google.com/kubernetes-engine](https://cloud.google.com/kubernetes-engine)

#### Tags

Cloud, Containers, GKE, Google Cloud, Kubernetes, Managed Service

#### Properties

- [Documentation](https://cloud.google.com/kubernetes-engine/docs)
- [OpenAPI](https://raw.githubusercontent.com/APIs-guru/openapi-directory/main/APIs/googleapis.com/container/v1/openapi.yaml)
- [Pricing](https://cloud.google.com/kubernetes-engine/pricing)

### Azure Resource Manager API
Azure's deployment and management layer; consistent REST API for creating and managing all Azure resources. ~28% global market share.

**Human URL:** [https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/overview)

#### Tags

Azure, Cloud, Infrastructure, Management, Microsoft, Resource Management

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/rest/api/resources/)
- [OpenAPI](https://raw.githubusercontent.com/Azure/azure-rest-api-specs/main/specification/resources/resource-manager/Microsoft.Resources/stable/2023-07-01/resources.json)

### Azure AKS API
Azure Kubernetes Service; manages clusters, node pools, and maintenance configurations for containerized workloads on Azure.

**Human URL:** [https://learn.microsoft.com/en-us/azure/aks/](https://learn.microsoft.com/en-us/azure/aks/)

#### Tags

AKS, Azure, Cloud, Containers, Kubernetes, Managed Service

#### Properties

- [Documentation](https://learn.microsoft.com/en-us/rest/api/aks/)
- [Pricing](https://azure.microsoft.com/en-us/pricing/details/kubernetes-service/)

### DigitalOcean API
Developer-friendly cloud platform API managing Droplets, Kubernetes, databases, Spaces (object storage), load balancers, and app platform.

**Human URL:** [https://www.digitalocean.com/](https://www.digitalocean.com/)

#### Tags

Cloud, Compute, Developer-Friendly, DigitalOcean, Infrastructure, Kubernetes

#### Properties

- [Documentation](https://docs.digitalocean.com/reference/api/)
- [OpenAPI](https://raw.githubusercontent.com/digitalocean/openapi/main/specification/DigitalOcean-public.v2.yaml)
- [Pricing](https://www.digitalocean.com/pricing)
- [SDK](https://github.com/digitalocean/doctl)

### Terraform Registry API
Access to IaC modules and providers for HashiCorp Terraform—the leading infrastructure as code tool for provisioning cloud resources declaratively.

**Human URL:** [https://developer.hashicorp.com/terraform](https://developer.hashicorp.com/terraform)

#### Tags

Infrastructure as Code, IaC, Open Source, Provisioning, Terraform

#### Properties

- [Documentation](https://developer.hashicorp.com/terraform/docs)
- [GitHub](https://github.com/hashicorp/terraform)
- [Registry](https://registry.terraform.io/)

### Pulumi Cloud API
Modern IaC platform using general-purpose programming languages (TypeScript, Python, Go, C#); Cloud API for stacks, deployments, and environments.

**Human URL:** [https://www.pulumi.com/](https://www.pulumi.com/)

#### Tags

Cloud, Infrastructure as Code, IaC, Open Source, Provisioning, Pulumi

#### Properties

- [Documentation](https://www.pulumi.com/docs/)
- [GitHub](https://github.com/pulumi/pulumi)
- [API Reference](https://www.pulumi.com/docs/pulumi-cloud/cloud-rest-api/)
- [Pricing](https://www.pulumi.com/pricing/)

## Schemas

| Artifact | Description |
|---|---|
| [Compute Instance Schema](json-schema/scalable-infrastructure-compute-instance-schema.json) | Normalized JSON Schema for cloud compute instances across AWS, GCP, Azure, and DigitalOcean. |
| [Kubernetes Cluster Schema](json-schema/scalable-infrastructure-kubernetes-cluster-schema.json) | JSON Schema for managed Kubernetes clusters across AWS EKS, GKE, AKS, and DigitalOcean DOKS, covering node pools, networking, and autoscaling. |

## Structures

| Artifact | Description |
|---|---|
| [Compute Instance Structure](json-structure/scalable-infrastructure-compute-instance-structure.json) | Hierarchical field documentation for cross-provider compute instance objects. |
| [Kubernetes Cluster Structure](json-structure/scalable-infrastructure-kubernetes-cluster-structure.json) | Hierarchical field documentation for managed Kubernetes cluster configurations including node pools, CNI networking, and cluster autoscaler settings. |

## Linked Data

| Artifact | Description |
|---|---|
| [Scalable Infrastructure Context](json-ld/scalable-infrastructure-context.jsonld) | JSON-LD context mapping infrastructure concepts to schema.org and cloud provider namespaces. |

## Examples

| Artifact | Description |
|---|---|
| [Compute Instance Example](examples/scalable-infrastructure-compute-instance-example.json) | Example AWS EC2 t3.xlarge instance with networking, storage, GPU, and tag configuration. |
| [Kubernetes Cluster Example](examples/scalable-infrastructure-kubernetes-cluster-example.json) | Example AWS EKS production cluster with general-purpose and GPU node pools, VPC CNI networking, and Cluster Autoscaler configuration. |

## Vocabulary

| Artifact | Description |
|---|---|
| [Scalable Infrastructure Vocabulary](vocabulary/scalable-infrastructure-vocabulary.yml) | Normative vocabulary for cloud providers, compute primitives, networking, storage, Kubernetes infrastructure, and IaC. |

## Common Properties

- [GitHub Organization](https://github.com/hashicorp)
- [CNCF Landscape](https://landscape.cncf.io/card-mode?category=provisioning)
- [Blog](https://www.cncf.io/blog/)

## Maintainers

**API Evangelist** — [kin@apievangelist.com](mailto:kin@apievangelist.com) — [https://apievangelist.com](https://apievangelist.com)
