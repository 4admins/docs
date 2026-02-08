## gcp/modules/api/variables.tf  
Declara variáveis necessárias para configurar o módulo API.  

| Nome               | Tipo   | Default           |
|--------------------|--------|-------------------|
| project_id         | string | `"4admins-academy"` |
| tf_state_bucket    | string | `"4admins-tf-state"` |

---

# Módulo Global Settings

## gcp/modules/global_settings/main.tf  
Centraliza o provedor e o estado remoto para configurações globais.  
- Configura provedor Google  
- Define backend GCS para o módulo  

```hcl
provider "google" {
  project = var.project_id
  region  = "europe-west2"
  zone    = "europe-west2-b"
}

terraform {
  backend "gcs" {
    bucket = "project-lab-terraform"
    prefix = "terraform/modules/global_settings"
  }
}
```

## gcp/modules/global_settings/variables.tf  
Define variáveis de escopo global e outputs de configuração.  

- **project_id**: ID do projeto no GCP  
- **south_caroline**: mapeia VPC, subrede, região e zonas  
- Output `south_caroline`: seleciona uma zona específica  

```hcl
variable "project_id" {
  type    = string
  default = "project-lab"
}

variable "south_caroline" {
  type = map(object({
    vpc        = string
    subnetwork = string
    region     = string
    zone       = list(string)
  }))
  default = {
    "config" = {
      vpc        = "projects/project-lab/global/networks/vpc"
      subnetwork = "projects/project-lab/regions/us-east1/subnetworks/prd-us-south-caroline"
      region     = "us-east1"
      zone       = ["us-east1-a", "us-east1-b", "us-east1-c", "us-east1-d"]
    }
  }
}

output "south_caroline" {
  value = var.south_caroline["config"].zone[2]
}
```
---