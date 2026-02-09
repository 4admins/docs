# Welcome to Test

For full document

## Commands

```hcl title=example.tf"
module "project_api_management" {
  source         = "terraform-google-modules/project-factory/google//modules/project_services"
  version        = "~> 14.2"
  project_id     = var.project_id
  activate_apis  = [
    "artifactregistry.googleapis.com",
    "compute.googleapis.com",
    "dns.googleapis.com",
    "secretmanager.googleapis.com",
    "workflows.googleapis.com",
    # ...
  ]
}
```