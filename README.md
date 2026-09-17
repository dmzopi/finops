# Google Kubernetes Engine (GKE) Cluster Terraform module

## Cost Estimation (Infracost)

This repository integrates **Infracost** to track cloud infrastructure costs and carbon footprint:

- **Integration**: Installed as a GitHub App ([Installed GitHub Apps settings](https://github.com/dmzopi/finops/settings/installations), see [configuration screenshot](infracost_config.png)).
- **Automated PR Reviews**: On every pull request, Infracost automatically calculates cost diffs against the base branch and posts a detailed breakdown comment directly to the PR (example: [Pull Request #1](https://github.com/dmzopi/finops/pull/1)).
- **Local CLI Commands**:
  ```bash
  # --- Scanning ---
  # Scan current directory for costs, policies, and carbon impact
  infracost scan

  # Scan with a specific currency (e.g. EUR, GBP) or output as JSON
  infracost scan --currency EUR
  infracost scan --json

  # --- Inspecting Results ---
  # Show summary (monthly cost, resources, policy checks)
  infracost inspect --summary

  # List resources sorted by highest monthly cost
  infracost inspect --group-by resource

  # Show only the top 5 most expensive resources
  infracost inspect --top 5

  # Group costs by resource type or file
  infracost inspect --group-by type
  infracost inspect --group-by file

  # --- FinOps Recommendations & Savings ---
  # Show total potential monthly savings across all FinOps recommendations
  infracost inspect --total-savings

  # List top 5 FinOps savings opportunities
  infracost inspect --top-savings 5 --fields address,monthly_savings,policy

  # --- Diagnostics & Troubleshooting ---
  infracost doctor
  ```


This module deploys a Kubernetes cluster on Google Cloud Platform (GCP) using the Google Kubernetes Engine (GKE) service. The GKE cluster is provisioned with a single node pool, and it comes with a generated Kubernetes certs credentials.

## Usage

```terraform
provider "google" {
  # Configuration options
  project = var.GOOGLE_PROJECT
  region  = var.GOOGLE_REGION
}

resource "google_container_cluster" "this" {
  name     = var.GKE_CLUSTER_NAME
  location = var.GOOGLE_REGION

  initial_node_count       = 1
  remove_default_node_pool = true
}

resource "google_container_node_pool" "this" {
  name       = var.GKE_POOL_NAME
  project    = google_container_cluster.this.project
  cluster    = google_container_cluster.this.name
  location   = google_container_cluster.this.location
  node_count = var.GKE_NUM_NODES

  node_config {
    machine_type = var.GKE_MACHINE_TYPE
  }
}

module "gke_auth" {
  depends_on = [
    google_container_cluster.this
  ]
  source               = "terraform-google-modules/kubernetes-engine/google//modules/auth"
  version              = ">= 24.0.0"
  project_id           = var.GOOGLE_PROJECT
  cluster_name         = google_container_cluster.this.name
  location             = var.GOOGLE_REGION
}

resource "local_file" "kubeconfig" {
  content  = module.gke_auth.kubeconfig_raw
  filename = "${path.module}/kubeconfig"
}

output "kubeconfig" {
  value = "${path.module}/kubeconfig"
}
```

## Inputs

|       Name       |            Description           |  Type  |     Default     | Required |
|:----------------:|:--------------------------------:|:------:|:---------------:|:--------:|
| GOOGLE_PROJECT   | GCP project name                 | string | no              |    no    |
| GOOGLE_REGION    | GCP region name                  | string | "us-central1-c" |    no    |
| GKE_MACHINE_TYPE | GKE node machine type            | string | "g1-small"      |    no    |
| GKE_NUM_NODES    | Number of nodes in the node pool | number | 2               |    no    |

## Outputs
kubeconfig - Generated Kubernetes configuration file

## Requirements
This module requires Terraform 0.12 or later, and the following provider:

hashicorp/google 4.52.0

## License
This module is licensed under the MIT License. See the LICENSE file for details.
