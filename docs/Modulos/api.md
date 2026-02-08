
## gcp/modules/api/api.tf  
Este arquivo orquestra a ativação das APIs do GCP necessárias ao projeto.  
- Utiliza o módulo `project_services` do _terraform-google-modules_  
- Recebe `project_id` e lista de APIs em `activate_apis`  
- Facilita manutenção centralizada das dependências de serviços  

```hcl
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