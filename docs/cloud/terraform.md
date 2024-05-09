## Infrastructure as Code (IaC)

Infrastructure as Code (IaC) is a method of managing and provisioning computing infrastructure through machine-readable definition files, rather than physical hardware configuration or interactive configuration tools. 

The IT infrastructure managed by this comprises both physical equipment, such as bare-metal servers, as well as virtual machines, and associated configuration resources. The definitions may be in a version control system. It can use either scripts or declarative definitions, rather than manual processes, but the term is more often used to promote declarative approaches.


### Pros

- Automatization of creation of an infrastructure;
- Standardization of platforms;
- Replication of infrastructure.

```
|- .github
|  |- workflows
|- s3-bucket-static
   |- main.tf
```

``` tf title="main.tf"

provider "aws" {
    region = "us-east-1"
}

variable "bucket_name" {
    type = string
}

resource "aws_s3_bucket" "static_site_bucket" {
    bucket = "static-site-${var.bucket_name}"

    website {
        index_document = "index.html"
        error_document = "404.html
    }

    tags = {
        Name = "Static Site Bucket"
        Environment = "Production"
    }
}

resource "aws_s3_public_access_block" "static_site_bucket" {
    bucket aws_s3_bucket.static_site_bucket.id

    block_public_acls       = false
    block_public_policy     = false
    ignore_public_acls      = false
    restrict_public_buckets = false
}
```

## Alternatives

- AWS CloudFormation
- Ansible
- Vagrant

## Additional Material

- <a href="https://youtu.be/BslJdgv_I2c" target="_blank">Fernanda Kipper - Criando Infra na AWS com Terraform (IaC)</a></i>

    [![](https://img.youtube.com/vi/BslJdgv_I2c/0.jpg){ width=80% }](https://youtu.be/BslJdgv_I2c){:target="_blank"}
