## gcp/modules/api/main.tf  
Define o provedor Google e configura o backend remoto em GCS.  
- `provider "google"` com projeto, região e zona  
- `terraform { backend "gcs" }` aponta para bucket de estado  

```hcl
provider "google" {
  project = var.project_id
  region  = "europe-west2"
  zone    = "europe-west2-b"
}

terraform {
  backend "gcs" {
    bucket = var.tf_state_bucket
    prefix = "terraform/modules/api"
  }
}
```