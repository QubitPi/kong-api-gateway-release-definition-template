Kong API Gateway Release Definition Template
============================================

[![Screwdriver CD badge][Screwdriver CD badge]][Screwdriver CD url]
[![HashiCorp Packer badge][HashiCorp Packer badge]][HashiCorp Packer url]
[![HashiCorp Terraform badge][HashiCorp Terraform badge]][HashiCorp Terraform url]
[![Apache License badge]][Apache License url]

A [Screwdriver CD template] that deploys an [immutable][Immutable Infrastructure] instance of [Kong API Gateway] to 
AWS. It uses the [screwdriver-template-main npm package] to assist with template validation, publishing, and tagging. 
This template tags the latest versions with the `latest` tag.

How to Use This Template
------------------------

> [!TIP]
> Before preceding, please note that it is assumed [Kong API gateway release definition template](./sd-template.yaml) 
> has already been installed in Screwdriver CD.
>
> If not, please see documentation on [publishing a template in Screwdriver]
> <img src="https://github.com/QubitPi/QubitPi/blob/master/img/8%E5%A5%BD.gif?raw=true" height="40px"/>

[Create a Screwdriver pipeline that uses this template](https://qubitpi.github.io/screwdriver-cd-guide/user-guide/templates#using-a-template). Here is an example:

```yaml
---
jobs:
  main:
    requires: [~pr, ~commit]
    template: QubitPi/kong_api_gateway_release_definition_template@latest
    secrets:
      - AWS_WS_PKRVARS_HCL
      - SSL_CERTIFICATE
      - SSL_CERTIFICATE_KEY
      - NGINX_CONFIG_FILE
      - AWS_WS_TFVARS
      - AWS_ACCESS_KEY_ID
      - AWS_SECRET_ACCESS_KEY
```

The following [Screwdriver Secrets] needs to be defined before running this template:

- [**AWS_ACCESS_KEY_ID**](https://qubitpi.github.io/hashicorp-aws/docs/setup#aws)
- [**AWS_SECRET_ACCESS_KEY**](https://qubitpi.github.io/hashicorp-aws/docs/setup#aws)

- [**SSL_CERTIFICATE**](https://qubitpi.github.io/hashicorp-aws/docs/setup#ssl)
- [**SSL_CERTIFICATE_KEY**](https://qubitpi.github.io/hashicorp-aws/docs/setup#ssl)
- [**NGINX_CONFIG_FILE**](https://qubitpi.github.io/hashicorp-aws/docs/setup#ssl)

- **AWS_WS_PKRVARS_HCL** - A [HashiCorp Packer variable values file] with the following variable values:

  ```hcl
  aws_image_region                 = "us-east-2"
  ami_name                         = "my-kong-ami"
  instance_type                    = "t2.small"
  aws_kong_ssl_cert_file_path        = "ssl.crt"
  aws_kong_ssl_cert_key_file_path    = "ssl.key"
  aws_kong_nginx_config_file_path    = "nginx.conf"
  ```

  - `aws_image_region` is the [image region][AWS regions] of [AWS AMI]
  - `ami_name` is the published AMI name; it can be arbitrary
  - `instance_type` is the recommended [AWS EC2 instance type] running this image
  - Please keep the values of `aws_kong_ssl_cert_file_path`, `aws_kong_ssl_cert_key_file_path`, and
    `aws_kong_nginx_config_file_path` as they are. They are used by [template](./sd-template.yaml) so that SSL configs
    are picked up from the right locations

- **AWS_WS_TFVARS** - A [HashiCorp Terraform variable values file] with the following variable values:

  ```hcl
  aws_deploy_region   = "us-east-2"
  ami_name            = "my-kong-ami"
  instance_type       = "t2.small"
  ec2_instance_name   = "My Kong API Gateway"
  ec2_security_groups = ["My Kong API Gateway Security Group"]
  route_53_zone_id    = "MBS8YLKZML18VV2E8M8OK"
  gateway_domain      = "gateway.mycompany.com"
  ```

  - `aws_deploy_region` is the [EC2 runtime region][AWS regions]
  - `ami_name` is the name of the published AMI; **it must be the same as the `ami_name` in AWS_WS_PKRVARS_HCL**
  - `instance_type` is the chosen [AWS EC2 instance type] at runtime
  - `ec2_instance_name` is the deployed EC2 name as appeared in the instance list of AWS console; it can be arbitrary
  - `ec2_security_groups` is the [AWS Security Group] _name_ (yes, not ID, but name...)
  - `gateway_domain` is the SSL-enabled domain that will serve [Kong manager UI]
  - `route_53_zone_id` is the AWS Route 53 hosted Zone ID that hosts the domain `gateway.mycompany.com`

> [!TIP]
> To find the zone ID in AWS Route 53, we can:
>
> 1. Sign in to the AWS Management Console
> 2. Open the Route 53 console at https://console.aws.amazon.com/route53/
> 3. Select Hosted zones in the navigation pane
> 4. Find the requested ID in the top level Hosted Zones summary in the Route 53 section

License
-------

The use and distribution terms for [Kong API gateway release definition template] are covered by the
[Apache License, Version 2.0].

<div align="center">
    <a href="https://opensource.org/licenses">
        <img align="center" width="50%" alt="License Illustration" src="https://github.com/QubitPi/QubitPi/blob/master/img/apache-2.png?raw=true">
    </a>
</div>

[Apache License badge]: https://img.shields.io/badge/Apache%202.0-F25910.svg?style=for-the-badge&logo=Apache&logoColor=white
[Apache License url]: https://www.apache.org/licenses/LICENSE-2.0
[Apache License, Version 2.0]: http://www.apache.org/licenses/LICENSE-2.0.html
[AWS AMI]: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html
[AWS EC2 instance type]: https://aws.amazon.com/ec2/instance-types/
[AWS regions]: https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.RegionsAndAvailabilityZones.html#Concepts.RegionsAndAvailabilityZones.Availability
[AWS Security Group]: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html

[HashiCorp Packer badge]: https://img.shields.io/badge/Packer-02A8EF?style=for-the-badge&logo=Packer&logoColor=white
[HashiCorp Packer url]: https://qubitpi.github.io/hashicorp-packer/packer/docs
[HashiCorp Packer variable values file]: https://qubitpi.github.io/hashicorp-packer/packer/guides/hcl/variables#from-a-file
[HashiCorp Terraform badge]: https://img.shields.io/badge/Terraform-7B42BC?style=for-the-badge&logo=terraform&logoColor=white
[HashiCorp Terraform url]: https://qubitpi.github.io/hashicorp-terraform/terraform/docs
[HashiCorp Terraform variable values file]: https://qubitpi.github.io/hashicorp-terraform/terraform/language/values/variables#variable-definitions-tfvars-files

[Immutable Infrastructure]: https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure

[Kong API Gateway]: https://qubitpi.github.io/docs.konghq.com/
[Kong API gateway release definition template]: https://github.com/QubitPi/kong-api-gateway-release-definition-template
[Kong manager UI]: https://qubitpi.github.io/docs.konghq.com/gateway/latest/kong-manager/

[publishing a template in Screwdriver]: https://qubitpi.github.io/screwdriver-cd-guide/user-guide/templates#publishing-a-template

[Screwdriver CD badge]: https://img.shields.io/badge/Screwdriver%20CD-1475BB?style=for-the-badge&logo=data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iaXNvLTg4NTktMSI/Pg0KPCEtLSBVcGxvYWRlZCB0bzogU1ZHIFJlcG8sIHd3dy5zdmdyZXBvLmNvbSwgR2VuZXJhdG9yOiBTVkcgUmVwbyBNaXhlciBUb29scyAtLT4NCjxzdmcgaGVpZ2h0PSI4MDBweCIgd2lkdGg9IjgwMHB4IiB2ZXJzaW9uPSIxLjEiIGlkPSJMYXllcl8xIiB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHhtbG5zOnhsaW5rPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5L3hsaW5rIiANCgkgdmlld0JveD0iMCAwIDUxMiA1MTIiIHhtbDpzcGFjZT0icHJlc2VydmUiPg0KPHBhdGggc3R5bGU9ImZpbGw6I2ZmZmZmZjsiIGQ9Ik01MDQuNzgzLDc3LjA5MWgtMC4wMDZIMzAzLjQ5NGMtMi4wMzYsMC0zLjk3NywwLjg1OS01LjM0NSwyLjM2Ng0KCWMtMS4zNjgsMS41MDgtMi4wMzgsMy41Mi0xLjg0Miw1LjU0OWwzLjkyNSw0MC42OTNjMC4zNDMsMy41NTIsMy4yMjgsNi4zMiw2Ljc5Miw2LjUxNmw0Mi4wNSwyLjMxNGwtODAuNzM2LDExMi4wMjNMMTgwLjI5Myw3MS4xNDINCglsNjMuNTYzLTIuNDNjMy44NDQtMC4xNDYsNi44OTYtMy4yNzYsNi45NDUtNy4xMjFsMC40NjQtMzYuMzkyYzAuMDIyLTEuOTMyLTAuNzI2LTMuNzkyLTIuMDgyLTUuMTY2DQoJYy0xLjM1Ni0xLjM3Mi0zLjIwNi0yLjE0Ny01LjEzNy0yLjE0N0g3LjIyYy0zLjk4OSwwLTcuMjIsMy4yMzEtNy4yMiw3LjIyMXY0MC41NThjMCwzLjk0NywzLjE3LDcuMTYzLDcuMTE1LDcuMjJsNjYuOTE3LDAuOTY0DQoJbDEyNy4yNTcsMjI0LjQ4OGwtMC41NjgsMTQwLjIwNWwtODguNDgsMy42MzFjLTMuODE3LDAuMTU1LTYuODUsMy4yNTktNi45MjMsNy4wNzdsLTAuNzE2LDM3LjUwNg0KCWMtMC4wMzcsMS45MzksMC43MDksMy44MSwyLjA2OSw1LjE5NmMxLjM1NiwxLjM4MywzLjIxMiwyLjE2MSw1LjE1MSwyLjE2MWgyNzYuNzYyYzEuOTgxLDAsMy44NzUtMC44MTMsNS4yMzktMi4yNDkNCgljMS4zNjMtMS40MzUsMi4wNzgtMy4zNjgsMS45NzQtNS4zNDZsLTEuOTMzLTM3LjEzMmMtMC4xOTItMy42OTItMy4xNC02LjY0MS02LjgzNS02LjgzNGwtNzYuMTI0LTMuOTkybC0xMS4wODItMTM2LjU3DQoJTDQzNi40NzksMTM3LjMzbDU1LjY0LTIuNTNjMy4wNjMtMC4xNDIsNS43MDQtMi4yMDIsNi41ODctNS4xMzlsMTIuOTA5LTQzLjAxOGMwLjI0OC0wLjcyOSwwLjM4NS0xLjUxNiwwLjM4NS0yLjMzMg0KCUM1MTIsODAuMzI1LDUwOC43NzIsNzcuMDkxLDUwNC43ODMsNzcuMDkxeiIvPg0KPC9zdmc+
[Screwdriver CD url]: https://qubitpi.github.io/screwdriver-cd-homepage/
[Screwdriver CD template]: https://qubitpi.github.io/screwdriver-cd-guide/user-guide/templates
[Screwdriver Secrets]: https://qubitpi.github.io/screwdriver-cd-guide/user-guide/configuration/secrets
[screwdriver-template-main npm package]: https://github.com/QubitPi/screwdriver-cd-template-main