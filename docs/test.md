# Welcome to Test

For full document

## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.


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
```