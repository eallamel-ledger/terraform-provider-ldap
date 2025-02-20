# Terraform LDAP 
```text
The only major difference from forked verison, is that within this fork, you can list what attributes within the object provider will skip reading.
```
## Installation

You can easily install the latest version with the following:

```shell
go get -u github.com/elastic-infra/terraform-provider-ldap
```

Then add the plugin to your local `.terraformrc`:

```shell
cat >> ~/.terraformrc <<EOF
providers {
  ldap = "${GOPATH}/bin/terraform-provider-ldap"
}
EOF
```

## Provider example

```hcl
provider "ldap" {
  ldap_host     = "ldap.example.org"
  ldap_port     = 389
  bind_user     = "cn=admin,dc=example,dc=com"
  bind_password = "admin"
}
```

## Resource LDAP Object example

```hcl
resource "ldap_object" "foo" {
  # DN must be complete (no RDN!)
  dn = "uid=foo,dc=example,dc=com"

  # classes are specified as an array
  object_classes = [
    "inetOrgPerson",
    "posixAccount",
  ]

  # attributes are specified as a set of 1-element maps
  attributes = [
    { sn              = "10" },
    { cn              = "bar" },
    { uidNumber       = "1234" },
    { gidNumber       = "1234" },
    { homeDirectory   = "/home/billy" },
    { loginShell      = "/bin/bash" },
    # when an attribute has multiple values, it must be specified multiple times
    { mail            = "billy@example.com" },
    { mail            = "admin@example.com" },
  ]
}
```

The Bind User must have write access for resource creation to succeed.

## Features

This provider is feature complete.
As of the latest release, it supports resource creation, reading, update, deletion
and importing.
It can be used to create nested resources at all levels of the hierarchy, 
provided the proper (implicit or explicit) dependencies are declared.
When updating an object, the plugin computes the minimum set of attributes that 
need to be added, modified and removed and surgically operates on the remote 
object to bring it up to date.
When importing existing LDAP objects into the Terraform state, the plugin can
automatically generate a .tf file with the relevant information, so that the 
following ```terraform apply``` does not drop the imported resource out of the
remote LDAP server due to it missing in the local ```.tf``` files.
In order to have the plugin generate this file, put the name of the output file
(which must *not* exist on disk) in the ```TF_LDAP_IMPORTER_PATH``` environment 
variable, like this:
```shell
$> export TF_LDAP_IMPORTER_PATH=a123456.tf 
$> terraform import ldap_object.a123456 uid=a123456,ou=users,dc=example,dc=com
```
and the plugin will create the ```a123456.tf``` file with the proper information.
Then merge this file into your existing ```.tf``` file(s).

## Limitations

This provider supports TLS, but certificate verification is not enabled yet; all
connections are through TCP, no UDP support yet.
