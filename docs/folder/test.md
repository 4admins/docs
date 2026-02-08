# Módulo API

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

# Módulo Instances – Italy

## gcp/modules/instances/italy/template/twingate_client.tftpl  
Mesma função do script alemão, para instalar o connector em instâncias italianas.  

```bash
#!/bin/bash
curl "https://binaries.twingate.com/connector/setup.sh" \
  | sudo TWINGATE_ACCESS_TOKEN="${accessToken}" \
         TWINGATE_REFRESH_TOKEN="${refreshToken}" \
         TWINGATE_URL="https://${tgnetwork}.twingate.com" bash
```

## gcp/modules/instances/italy/backup_dr.tf  
Stub comentado para módulo _Backup & DR_ do GCP.  
- Parâmetros de appliance, rede, zona e credenciais  
- Pode ser ativado ajustando `module "backup_dr_appliance"`

## gcp/modules/instances/italy/cloud-nat.tf  
Configura Cloud NAT (todos os recursos comentados).  
- `google_compute_address`, `google_compute_router`, `google_compute_router_nat`  
- Permite saída de internet para sub-redes privadas

## gcp/modules/instances/italy/cloud_storage_buckets.tf  
Exemplo comentado de bucket GCS para armazenar artefatos.  
- `google_storage_bucket` com versão e políticas de ciclo de vida

## gcp/modules/instances/italy/firewall.tf  
Regras de firewall comentadas para instâncias italianas.  
- Exemplo de liberação completa (`protocol = "all"`)  
- SSH via IAP e HTTPS

## gcp/modules/instances/italy/it001srvfsw01.tf  
Configuração de máquina `it001srvfsw01` (comentada).  
- Definição de VM, disco de boot, tags e rótulos  
- Integração com Twingate resource

## gcp/modules/instances/italy/it001uptimekuma.tf  
Instância `it001uptimekuma` para monitoramento (comentada).  
- Endereço interno reservado  
- Recurso Twingate para exposição segura

## gcp/modules/instances/italy/it002srvfsw01.tf  
VM `it002srvfsw01` (comentada).  
- Configurações similares a `it001srvfsw01`  
- Recursos de rede e Twingate

## gcp/modules/instances/italy/it002srvfsw02.tf  
VM `it002srvfsw02` (comentada).  
- Suporte a instâncias _spot_ (comentado)  
- IP interno reservado

## gcp/modules/instances/italy/it002srvupk01.tf  
Instância de atualização `it002srvupk01` (comentada).  
- Endereço interno  
- Record-set DNS no Cloud DNS (comentado)

## gcp/modules/instances/italy/it_twingate_connectors_italy.tf  
Define redes e conectores Twingate (comentados).  
- `twingate_remote_network` para Turin e Berlin  
- `twingate_connector` e `twingate_connector_tokens`

## gcp/modules/instances/italy/main.tf  
Provê o módulo, backend GCS e providers Twingate/NewRelic.  
- Secret Manager para token Twingate  
- Estrutura similar ao módulo Germany

## gcp/modules/instances/italy/subnets.tf  
Sub-redes para escritórios e armazéns italianos.  

- **it_turin_office**, **it_turin_warehouse**  
- **it_rome_office**, **it_rome_warehouse**  
- **it_milan_office**, **it_milan_warehouse**  

Cada recurso usa `var.italy[...]` para configuração.

## gcp/modules/instances/italy/variables.tf  
Define o mapa `italy` com configurações de rede.  

- Cada local (`turin`, `rome`, `milan`) tem chaves `office` e `warehouse`  
- Parâmetros: `network`, `region`, `zone[]`, `subnetwork_name`, `cidr_range`, `country`, etc.

---

# Módulo Instances – Geral

## gcp/modules/instances/cloudflare_data.tf  
Busca o `zone_id` do Cloudflare via Secret Manager.

```hcl
data "google_secret_manager_secret_version" "cloudflare_zone_id_camilloaugusto_space" {
  secret = "cloudflare_zone_id_camilloaugusto_space"
}
```

## gcp/modules/instances/cloudflare_dns.tf  
Registros DNS no Cloudflare (comentados).  
- Exemplo de `cloudflare_dns_record` para “verde” e “azul”

## gcp/modules/instances/cloudflare_provider.tf  
Configuração do provider Cloudflare e seu token.

```hcl
terraform {
  required_providers {
    cloudflare = { source = "cloudflare/cloudflare", version = "~> 5" }
  }
}
provider "cloudflare" {
  api_token = data.google_secret_manager_secret_version.cloudflare_key.secret_data
}
```

## gcp/modules/instances/community_center_institucional.tf  
Instância de site institucional (comentada).  
- VM `srvpweb01` com tags e endereços

## gcp/modules/instances/community_center_institucional_old.tf  
Versão antiga do recurso institucional (comentada).  
- Uso de `locals` para definir parâmetros

## gcp/modules/instances/data.tf  
Pode conter data sources compartilhadas (não detalhado)

## gcp/modules/instances/main.tf  
Define provedor Google e backend GCS para todo o módulo de instâncias.

```hcl
provider "google" {
  project = var.project_id
  region  = "europe-west2"
  zone    = "europe-west2-b"
}

terraform {
  backend "gcs" {
    bucket = "4admins-tf-state"
    prefix = "terraform/modules/instances"
  }
}
```

## gcp/modules/instances/os_policy.tf  
Conteúdo comentado para políticas de patch via OS Config.

## gcp/modules/instances/srvpad01.tf  
Definição (comentada) da VM `srvpad01` e IP interno reservado.

## gcp/modules/instances/srvpweb02 copy.tf  
Cópia de configuração para `srvpweb02` (comentada).

## gcp/modules/instances/srvpweb03.tf  
VM `srvpweb03` com integração DNS e Twingate (comentada).

## gcp/modules/instances/srvpweb04.tf  
VM `srvpweb04` para UK/London (comentada).

## gcp/modules/instances/variables.tf  
Declara variáveis: `project_id`, `tf_state_bucket_name`, `london`, `south_caroline`, `sao_paulo`, `germany`, `italy`.  
- Cada variável define mapeamento de VPC, subrede, região e zonas.

---

# Módulo Network

## gcp/modules/network/cloud-nat.tf  
Cloud NAT para uma VPC compartilhada (comentado).

## gcp/modules/network/clouddns.tf  
Configurações (comentadas) de Cloud DNS e zonas privadas/públicas.

## gcp/modules/network/cloudflare_data.tf  
Busca `cloudflare_zone_id` de Secret Manager (comentado).

## gcp/modules/network/cloudflare_provider.tf  
Provider Cloudflare (comentado).

## gcp/modules/network/firewall.tf  
Regras globais de firewall na VPC principal.  
- Recurso ativo: libera todo o tráfego HTTPS (`allow_ingress_https`)

## gcp/modules/network/main.tf  
Configura provedor Google e backend GCS para rede.

## gcp/modules/network/secret_manager.tf  
Cria secrets no Secret Manager para Cloudflare.

## gcp/modules/network/subnets_one_vpc brazcompany.tf  
Sub-redes baseadas na VPC `brazcompany-network` (comentadas).

## gcp/modules/network/subnets_one_vpc cop.tf  
Sub-redes para VPC “cop” (comentadas).

## gcp/modules/network/subnets_one_vpc magic_city.tf  
Sub-redes para VPC “magic-city” (comentadas).

## gcp/modules/network/variables.tf  
Declara `project_id` e configurações de rede.

## gcp/modules/network/vpc.tf  
Cria três VPCs base: `vpc`, `magic-city` e `brazcompany-network`.

```hcl
resource "google_compute_network" "vpc" {
  name                    = "vpc"
  auto_create_subnetworks = false
  routing_mode            = "GLOBAL"
  project                 = var.project_id
}

resource "google_compute_network" "magic_city" {
  name                    = "magic-city"
  auto_create_subnetworks = false
  routing_mode            = "GLOBAL"
  project                 = var.project_id
}

resource "google_compute_network" "brazcompany_network" {
  name                    = "brazcompany-network"
  auto_create_subnetworks = false
  routing_mode            = "GLOBAL"
  project                 = var.project_id
}
```

---

# Módulo Twingate

## gcp/modules/twingate/template/twingate_client.tftpl  
Mesma lógica de bootstrap para conectores Twingate.

## gcp/modules/twingate/main.tf  
Provê o backend GCS e provedor Google para o módulo Twingate.

```hcl
provider "google" {
  project = var.project_id
  region  = "europe-west2"
  zone    = "europe-west2-b"
}

terraform {
  backend "gcs" {
    bucket = var.tf_state_bucket_name
    prefix = "terraform/modules/twingate"
  }
}
```

## gcp/modules/twingate/twingate.tf  
Exemplos comentados de rede e sub-rede próprias para testes Twingate.

## gcp/modules/twingate/twingate_provider.tf  
Define o provider Twingate e integra com o token recuperado.

## gcp/modules/twingate/twingate_gcp_connectors.tf  
Exemplos de criação de conectores e tokens (comentados).

## gcp/modules/twingate/twingate_groups.tf  
Cria grupos de acesso no Twingate:

```hcl
resource "twingate_group" "it" {
  name = "IT"
}
resource "twingate_group" "marketing" {
  name = "Marketing"
}
resource "twingate_group" "sales" {
  name = "Sales"
}
```

## gcp/modules/twingate/twingate_instance.tf  
Exemplo de instância Google exposta via Twingate (comentada).

## gcp/modules/twingate/twingate_resources.tf  
Define recursos/twingate_resource para VMs (comentado).

## gcp/modules/twingate/twingate_users.tf  
Exemplo de usuário Twingate (comentado).

## gcp/modules/twingate/variables.tf  
Variáveis `project_id` e `tf_state_bucket_name`.

## gcp/modules/twingate/x_twingate_gcp_network.tf  
Sub-redes e firewall específicas para testar Twingate (comentadas).

---

Cada módulo segue o padrão Terraform de:
- **Provider** e **Backend** bem definidos  
- Uso de **variables.tf** para parâmetros configuráveis  
- Estrutura modular que facilita integração em ambientes maiores  

Este panorama detalha o propósito e relação entre arquivos e módulos, permitindo navegar facilmente pela infraestrutura como código desenvolvida neste repositório. 🎯