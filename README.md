# serverless.tf - Doing serverless with Terraform

This repository / _[serverless.tf framework](https://serverless.tf)_ aims to provide information, solutions, and tools for building, deploying, and managing serverless applications and infrastructure using [Terraform](https://www.terraform.io/). 

serverless.tf has started as an organic response to the accidental complexity of many existing tools used by serverless developers.

serverless.tf is not an official AWS or HashiCorp product, and not to be confused with the [Serverless Framework](https://serverless.com/).


## Terms of Content

1. [Current Status](#current-status)

1. [Getting Started with "why?"](#getting-started-with-why)
    * [Existing tools for serverless developers](#existing-tools-for-serverless-developers)
    * [What does a serverless application consist of?](#what-does-a-serverless-application-consist-of)
    * [Lack of high-quality reusable components](#lack-of-high-quality-reusable-components)

1. [Development Workflows](#development-workflows)
    * [Build](#build)
    * [Package](#package)
    * [Test](#test)

1. [serverless.tf-compatible Terraform modules](#serverlesstf-compatible-terraform-modules)

1. [What is next?](#what-is-next)

<!-- @todo: finish this...
1. Comparisons - serverless.tf vs ...
1. Serverless Framework
1. AWS Serverless Application Model
1. AWS Cloud Development Kit (CDK)
1. AWS Chalice
1. Pulumi
1. Secure
1. Deploy
1. Monitoring, logging, alerting
1. Rollback
1. Orchestrate
1. Multiple environments
1. CI/CD
-->


## Current Status

This project is in the initial phase. Going through the development process with [Betajob](https://www.betajob.com/)'s products, and with external customers, serverless.tf's concepts will be verifying and updating.

We focus on AWS specific serverless features and services, but most of the information described here can also be applied to [Google Cloud Functions](https://cloud.google.com/functions), [Azure Functions](https://azure.microsoft.com/en-us/services/functions/), and any other provider with decent support for the resources via [Terraform provider](https://www.terraform.io/docs/providers/).


### Getting Started with "why?"

Most likely, the first question you are wondering - _Why do you do this?_ Or, if you know me and have been following my projects for some time, you may think: _Yes, we can do a lot with Terraform, but wrong is with existing solutions available already?_

Before answering what is wrong, let's set the stage by highlighting the good parts of existing toolset available for serverless developers.

There are plenty of tools and frameworks with overlapping functionality, which is excellent - developers now have a choice, if they want.

Assuming that working with serverless would automatically bring developers **everything better** is one of the misconceptions many developers have after starting playing with it. There is [Law of conservation of complexity](https://en.wikipedia.org/wiki/Law_of_conservation_of_complexity) that can be applied to serverless architectures, too. In simple words, it means that the complexity is not changing, but the complexity can be allocated differently between application developers, tools developers, and cloud providers.

As serverless application developers, we shouldn't be exposed to accidental complexity enforced by tools we have to use. Complexity should be simplified as much as possible (always).

There is flexibility at the cost of accidental complexity developers have to deal with.


### Existing tools for serverless developers

When creating a serverless application, at the minimum least, developers have to deal with:

1. Serverless application frameworks ([Serverless Framework](https://www.serverless.com/), [AWS Chalice](https://chalice.readthedocs.io/en/latest/), [Zappa](https://github.com/Miserlou/Zappa)) to develop, test, and deploy code and serverless infrastructure resources.
2. Infrastructure management ([Terraform](https://www.terraform.io/), [AWS CloudFormation](https://aws.amazon.com/cloudformation/)) manages traditional resources which are not related to serverless, and sometimes to create resources for serverless applications, too.
3. Application deployment (Shell scripts, Makefile, [AWS CodeDeploy](https://aws.amazon.com/codedeploy/), AWS CLI) to orchestrate non-trivial builds of dependencies and do deployments, also.

As you can see, there is an overlap in the functionality of these tools.


### What does a serverless application consist of?

Let's list some parts a typical serverless application has. An unordered list of items include:

1. Code of your serverless function
1. Code dependencies
1. Shell scripts, Makefile, or similar, what installs dependencies and create a build package
1. Serverless infrastructure resources (AWS Lambda Function, Lambda Layers, related IAM roles, and policies)
1. Infrastructure resources which can be used by serverless functions (e.g., S3 buckets, SQS queues, SNS notifications, etc.)
1. Traditional infrastructure resources typically used and managed without related serverless applications (VPC, ALB, Security Groups, etc.)
1. Event mapping for your serverless resources
1. Monitoring metrics and logs (e.g., CloudWatch logs)
1. Deployment, rollback, scaling policies

All of these items are mostly _nodes with dependencies_ developers should describe declaratively (see [Infrastructure as Code: Imperative vs. Declarative](https://tech.ovoenergy.com/imperative-vs-declarative/)). We can treat all of these entities using a somewhat similar approach as we do with the [Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code) and [DevOps automation](https://en.wikipedia.org/wiki/DevOps) at large.

Serverless.tf's approach is to use [Terraform](https://www.terraform.io/), one of the most popular and powerful infrastructures as a code management tool.

Additionally, developers can use [Terragrunt](https://terragrunt.gruntwork.io/) as an orchestration tool for Terraform. It is not required but highly recommended since it reduces the complexity of working with Terraform configurations and keeping them DRY as the number of resources increases.


### Lack of high-quality reusable components

Some of the existing solutions support plugins that extend the functionality of existing frameworks and simplify infrastructure services.

In reality, often, developers still have to dive into those to learn internals and archive what they need. Adding functionality to those plugins usually requires writing Javascript code.

Using serverless.tf approach developers rely on open-source [Terraform AWS modules](https://github.com/terraform-aws-modules), which have been developing by [Betajob](https://www.betajob.com/) and huge community during several years, you get to build your serverless project on top of the verified, reusable components.

Using existing modules allows serverless developers to focus on their primary tasks instead of learning the internals of Terraform or googling the right piece of AWS CloudFormation snippet. Developers can see and execute working examples in each of the modules, integrate modules into the project, and get to know the modules' source code when necessary.

See the whole list of [Supported AWS Serverless Platform Services](https://serverless.tf/#serverless-services) on serverless.tf. More non-serverless modules listed on the [Terraform Registry](https://registry.terraform.io/modules/terraform-aws-modules/).


## Development Workflows

serverless.tf does not restrict any workflow, but shows how to build, package, test, deploy, monitor, and some other steps can be implemented using Terraform.

serverless.tf's approach advises management of all infrastructure resources equally, independently of their nature or provider - build or deploy of the code of the serverless function, manage VPC resources, manage GitHub users - all of this should be managed using the same commands - `terraform apply`.


### Build

Lambda functions usually have dependencies (libraries and binaries) built locally, in Docker, or by using external tools or services (e.g., [AWS CodeBuild](https://aws.amazon.com/codebuild/)).

[Terraform AWS Lambda module](https://github.com/terraform-aws-modules/terraform-aws-lambda) supports all of these ways of building dependencies (see [examples](https://github.com/terraform-aws-modules/terraform-aws-lambda/tree/master/examples) there).

Lambda layers have the same build and package process, so building the Lambda layer and Lambda functions is an identical process.

Using commands like [sam build](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-using-build.html) provided by AWS SAM can be a feasible option if you are using AWS SAM already and want to perform gradual migration towards serverless.tf's approach and start using Terraform AWS Lambda module where extra features like exclude files by masks, configurable storage, and conditional creation already supported.


### Package

Creation of Lambda deployment package (for a function or a layer) supported by [Terraform AWS Lambda module](https://github.com/terraform-aws-modules/terraform-aws-lambda) and can be customized, or completely disabled in favor of using external tool or script to do that (see [examples](https://github.com/terraform-aws-modules/terraform-aws-lambda/tree/master/examples) there).


### Test

Running any tests required for serverless application with Terraform efficiently is rather tricky simply because Terraform was not designed to run tests. There are several options developers can use:

1. [null_resource](https://www.terraform.io/docs/providers/null/resource.html) to run shell scripts without worrying about output. 
1. [Shell provider](https://github.com/scottwinkler/terraform-provider-shell) to manage Shell scripts as fully-fledged resources and have full control of a resource lifecycle handled by Terraform.
1. [External provider](https://www.terraform.io/docs/providers/external/data_source.html) to run any command or script.

By using Terragrunt or other tools which allow developers to orchestrate Terraform code, developers can run extra commands (e.g., shell scripts) that are not part of the infrastructure code itself (see [Before and After Hooks](https://terragrunt.gruntwork.io/docs/features/before-and-after-hooks/) there) to perform tests without putting _irrelevant_ code into main infrastructure repository, for example.


<!--
@todo: finish this...
### Secure

### Deploy

### Monitoring, logging, alerting

### Rollback

### Orchestrate

### Integrations

### Multiple environments

### CI/CD


## Comparisons - serverless.tf vs ...

### Serverless Framework

### AWS Serverless Application Model

### AWS Cloud Development Kit (CDK)

### AWS Chalice

### Pulumi
-->

## serverless.tf-compatible Terraform modules

All Terraform modules listed on [serverless.tf](https://serverless.tf/#aws-serverless) plus all modules in [Terraform AWS Modules GitHub Organization](https://github.com/terraform-aws-modules) available in the [Terraform Registry](https://registry.terraform.io/modules/terraform-aws-modules) were also designed and implemented with serverless.tf-compatibility in mind (e.g., naming, features, dependencies, quality, etc.).
 

## What is next?

Open-ended feedback is welcome through [Github issues](https://github.com/antonbabenko/serverless.tf/issues). You can also reach out to me via email - [anton@antonbabenko.com](mailto:anton@antonbabenko.com) . I am particularly interested in hearing about these topics:

1. What is **your** biggest challenge working with serverless?
1. Why do you use or don't Terraform with serverless?
1. What kind of content is missing or wrong in serverless.tf? Remember, `serverless.tf` is currently determining and revising its concepts and approaches.

Follow [AWS Serverless Heroes](https://aws.amazon.com/developer/community/heroes/?community-heroes-all.sort-by=item.additionalFields.sortPosition&community-heroes-all.sort-order=asc&awsf.filter-hero-category=heroes%23serverless) to learn about serverless from the experts.


### Authors

[serverless.tf](https://serverless.tf) project managed by [Anton Babenko](https://github.com/antonbabenko), and is not affiliated with AWS or HashiCorp.


### Like this? Please follow me and share it with your network!

[![@antonbabenko](https://img.shields.io/twitter/follow/antonbabenko.svg?style=flat&label=Follow%20@antonbabenko%20on%20Twitter)](https://twitter.com/antonbabenko)
[![@antonbabenko](https://img.shields.io/github/followers/antonbabenko?style=flat&label=Follow%20@antonbabenko%20on%20Github)](https://github.com/antonbabenko)

Consider support my work on [GitHub Sponsors](https://github.com/sponsors/antonbabenko), [Buy me a coffee](https://www.buymeacoffee.com/antonbabenko), or [PayPal](https://www.paypal.me/antonbabenko).


## License

MIT licensed. See LICENSE for full details.
