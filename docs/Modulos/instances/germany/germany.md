# Módulo Instances – Germany

## gcp/modules/instances/germany/template/twingate_client.tftpl  
Script de inicialização para instalar o connector Twingate em VMs.  

```bash
#!/bin/bash
curl "https://binaries.twingate.com/connector/setup.sh" \
  | sudo TWINGATE_ACCESS_TOKEN="${accessToken}" \
         TWINGATE_REFRESH_TOKEN="${refreshToken}" \
         TWINGATE_URL="https://${tgnetwork}.twingate.com" bash
```

## gcp/modules/instances/germany/firewall.tf  
Contém definições de regras de firewall (comentadas), preparadas para HTTPS/SSH.  
- Exemplos para liberar portas 22, 80 e 443  
- Pode ser ativado ajustando `resource "google_compute_firewall"`

## gcp/modules/instances/germany/main.tf  
Configura provedores e gerencia credenciais via Secret Manager.  
- Google, Twingate, Cloudflare, UpTimeRobot  
- Data sources para segredos: `twingate_token`, `cloudflare_key`, etc.  
- Backend GCS e `required_providers`

```hcl
terraform {
  backend "gcs" {
    bucket = "4admins-tf-state"
    prefix = "terraform/modules/instances/germany"
  }
  required_providers {
    twingate    = { source = "twingate/twingate" }
    newrelic    = { source = "newrelic/newrelic" }
    cloudflare  = { source = "cloudflare/cloudflare", version = "~> 5" }
    uptimerobot = { source = "uptimerobot/uptimerobot" }
  }
}

data "google_secret_manager_secret_version" "twingate_token" {
  secret = "twingate_token"
}

provider "twingate" {
  api_token = data.google_secret_manager_secret_version.twingate_token.secret_data
  network   = "4admins"
}

# ...
```

## gcp/modules/instances/germany/subnets.tf  
Cria sub-redes em várias regiões alemãs.  

- **de_berlin_office**  
- **de_berlin_warehouse**  
- **de_frankfurt_office**  
- **de_munich_office**  
- **de_hamburg_office**  

Cada recurso usa `var.germany[...]` para rede, região e CIDR.

## gcp/modules/instances/germany/variables.tf  
Define projeto e configurações de rede para data centers na Alemanha.  

- **project_id**, **tf_state_bucket_name**  
- **germany**: objeto aninhado com chaves `berlin`, `frankfurt`, `munich`, `hamburg`  
  - Cada local tem `network`, `region`, `zone[]`, `subnetwork_name`, `cidr_range`, entre outros.

---