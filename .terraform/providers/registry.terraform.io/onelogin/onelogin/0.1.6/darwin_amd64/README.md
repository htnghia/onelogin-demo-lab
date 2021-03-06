# Onelogin Terraform Provider
[![Go Report Card](https://goreportcard.com/badge/github.com/onelogin/terraform-provider-onelogin)](https://goreportcard.com/report/github.com/onelogin/terraform-provider-onelogin)
<a href='https://github.com/dcaponi/gopherbadger' target='_blank'>![gopherbadger-tag-do-not-edit](https://img.shields.io/badge/Go%20Coverage-100%25-brightgreen.svg?longCache=true&style=flat)</a>

# Prerequisites
    1) Install Golang
    2) Install Terraform v0.12.24 or later
    3) Install code dependencies

# Getting Started
If you are sideloading this provider (i.e. not getting this via the Terraform store) You must clone this repository to run the following commands.

1) In the ./terraform-provider-onelogin directory run:
    ```
    make sideload
    ```

    If you are using Terraform v0.13.x or later you can use following Terraform configuration for sideloaded version of this provider:
    ```
    terraform {
      required_providers {
        onelogin = {
          source  = "onelogin.com/onelogin/onelogin"
          version = "0.1.0"
        }
      }
    }

    provider "onelogin" {
      # Configuration options
    }
    ```

2) You'll need admin access to a OneLogin account where you can create API credentials. Create a set of API credentials with _manage all_ permission. For applying the credentials, there are 2 ways

    * Export these credentials to your environment and the provider will read them in from there
    ```
    export ONELOGIN_CLIENT_ID=<your client id>
    export ONELOGIN_CLIENT_SECRET=<your client secret>
    export ONELOGIN_OAPI_URL=<the api url for your region>
    ```

3) You are ready to use the provider, just follow the terraform commands!

# Managing App Resources
Refer to [creating an App](https://developers.onelogin.com/api-docs/2/apps/create-app)
On create, omitted fields are ignored and set to their empty or default values.

On update, omitted fields are treated as if the intent is to clear the field and
an empty or zero value is sent to the API to clear the field. E.G. creating an app with
a description, then removing the description field in your HCL file, will result in
setting the description to `""`

### App Fields
Required fields are, well, required.
Computed fields are set by the API and cannot be set via Terraform

1) name [string] *required*
2) connector_id [int] *required*
1) description [string]
2) notes [string]
3) visible [bool] - Defaults to `true`
4) allow_assumed_signin [bool] - Defaults to `false`
5) parameters [set no limit] - see below
6) configuration [set limit 1] - see below
7) provisioning [set limit 1] - see below
8) auth_method *computed*
9) icon_url *computed*
10) policy_id *computed*
11) tab_id *computed*
12) updated_at *computed*
13) created_at *computed*

### Parameter Sub Field
*Known Issue* - Parameters are tracked as completely separate entities in the OneLogin API.
Currently only adding new/additional parameters permitted is permitted here as their lifecycle methods Have not yet been implemented. Removing or Changing a parameter will NOT affect the state of the App resource as of v0.0.1.

1) param_key_name [string] *required*
2) param_id [int] *computed*
3) label [string]
4) user_attribute_mappings [string]
5) user_attribute_macros [string]
6) attributes_transformations [string]
7) default_values [string]
8) skip_if_blank [bool] - Defaults to `false`
9) values [string]
10) provisioned_entitlements [bool] - Defaults to `false`
11) safe_entitlements_enabled [bool] - Defaults to `false`

### Provisioning Sub Field
1) Enabled [bool] - Defaults to `false`

### App Type Specific Sub Fields
Configuration and SSO depends on they app's authentication type e.g. SAML or OIDC and has different fields.

#### SAML Configuration
1) certificate_id *computed*
2) provider_arn [string]
3) signature_algorithm [string] *required* - one of `SHA-1`, `SHA-256`, `SHA-348`, `SHA-512`

#### SAML SSO
1) acs_url [string] *computed*
2) metadata_url [string] *computed*
3) issuer [string] *computed*
4) certificate [set limit 1]
    * name [string] *computed*
    * id [string]*computed*
    * value [string] *computed*
    * sls_url [string] *computed*

#### OIDC Configuration
1) redirect_uri [string]
2) refresh_token_expiration_minutes [int] - defaults to 1 minute
3) login_url [string]
4) oidc_application_type [int] - one of `0` (Web) or `1` (Native/Mobile)
5) token_endpoint_auth_method [int] - one of `0` (Basic) `1` (POST) `2` (Nonce/PKCE)
6) access_token_expiration_minutes [int] - defaults to 1 minute

#### OIDC SSO
1) client_id [string] *computed*
2) client_secret [string] *computed*

# Terraform
Install:
```
brew install terraform
```

Initialize:
 ```
terraform init
```

Plan:
```
terraform plan
```

Apply:
```
terraform apply
```

# Dependency Management
We use go mod for dependency management.

To add a package:

```
go get -u "package-name"
```

To re-install dependencies for this project:
```
rm go.sum
go mod download
```

To update dependencies for this project:
```
go mod -u ./...
```

# Running Tests
Standard Go Way:
```
go test ./... -v -cover
```

Including Terraform Acceptance Tests
```
TF_ACC=1 go test ./... -v -cover
```
