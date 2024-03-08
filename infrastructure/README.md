# Verification Academy Infrastructure

![GitHub Action badge](https://github.com/ombu/va-2023/workflows/build/badge.svg)



## :bulb: 10,000 foot Services Overview for each site

![Services Overview](https://github.com/benhartig/temp/blob/main/infrastructure/.images/flow-of-services.png?raw=true)

## :book: Table Of Contents

- [Installation](#toolbox-installation)
- [External Services](#diamond_shape_with_a_dot_inside-external-services)
- [Stack Architecture](#classical_building-stack-architecture)
    - [backup](#backup)
    - [cluster](#cluster)
    - [cms](#cms)
    - [cms-cert](#cms-cert)
    - [deploy-pipeline](#deploy-pipeline)
    - [deploy-user](#deploy-user)
    - [forum](#forum)
    - [legacy](#legacy)
    - [resources](#resources)
    - [support-center-bucket](#support-center-bucket)
    - [tracking](#tracking)
    - [web](#web)
    - [web-cert](#web-cert)
- [Notes](#speech_balloon-notes)
- [`stacked` Commands](#gear-stacked-commands)
- [MakeFile Commands](#gear-makefile-commands)
- [Code Linting & Formatting](#mag-code-linting--formatting)
- [Folder Structure](#file_folder-folder-structure)
- [Footnotes](#link-footnotes)


## :toolbox: Installation

The infrastructure management uses Python 3.11. This version requirement is
documented in the file .tool_versions and can be used with [asdf](https://asdf-vm.com/).

Install the dependencies:
 
```console
make install
```

> [!TIP]
> Install the [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
to be able to interact with EC2 compute and ECS containers resources that are
behind a private subnet in the VPC. Then add the following to your `.ssh/config`.

```console
# SSH over AWS Systems Manager Session Manager
host i-* mi-*
    ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
```




## :diamond_shape_with_a_dot_inside: External Services




## :classical_building: Stack Architecture

## :bulb: 10,000 foot Stack Overview

![Stack Overview](https://github.com/benhartig/temp/blob/main/infrastructure/.images/flow-of-stacks.png?raw=true)

> ### backup

> ### cms

> ### cms-cert

> ### cluster

> ### deploy-pipeline

> ### deploy-user

> ### forum

> ### legacy

> ### resources

> ### support-center-bucket

> ### tracking

> ### web-cert

> ### web




## :speech_balloon: Notes

> [!NOTE]  
> You don't have to use `assume-role` if you are not coming from another
AWS account.

> [!TIP]
> Optional information to help a user be more successful.


> [!WARNING]
> This infrastructure creates CloudFront Distrubutions in a different region
than `us-east-1`. CloudFront Distrubution Certificates created by the `cms-cert`
 and `web-cert` stack must be launched from `us-east-1` [^1], then provide the
`cms` stack and `web` stack with the ARN of the certificates.

> [!TIP]
> CMS Cloudfront S3 Bucket Policy sometimes has issues when deploying or
updating and will get an access deined to placing the policy for the S3 bucket.
Solution is to add this inline policy because the normal S3 policy doesn't seem
to allow for `s3:PutBucketPolicy`.

```console
{
    "Version": "2012-10-17",
    "Statement": {
        "Action": [
            "s3:*",
            "cloudformation:*"
        ],
        "Resource": "*",
        "Effect": "Allow"
    }
}
```




## :gear: `stackedup` Commands

Infrastructure makes extensive use of [OMBU Stackedup](https://github.com/ombu/stacks) for configurating, packaging, launching and updating
AWS CloudFormation templates in the form of stacks.

| Command                                                           | Options    | Description                                                      |
| ----------------------------------------------------------------- | ---------- | ---------------------------------------------------------------- |
| `assume-role <account>`                                           |            | Assume a role defined in the accounts section of `config.yaml`   |
| `stack-launch <service> <instance>`                               | `--config` | Launch a new stack from the `config.yaml`                        |
| `stack-update <service> <instance>`                               | `--config` | Update an exisitng stack                                         |
| `stack-details <service> <instance>`                              | `--config` | Get details of an existing stack                                 |
| `container-shell <instance> <service> <service-name> <container>` | `--config` | Shell into an ECS Serviced from a stack defined in `config.yaml` |




## :gear: MakeFile Commands

| Command                                  | Needed ENV | Need `assume-role`| Description                                               |
| ---------------------------------------- | ---------- | ----------------- | --------------------------------------------------------- |
| `make help`                              |            |                   | Show help and list make commands                          |
| `remote-install-s3-cms`                  | TYPE       |                   | Install sanity cli for cms support                        |
| `remote-build-s3-cms`                    | TYPE       |                   | Build the public sanity files                             |
| `remote-push-s3-cms`                     | TYPE       | YES               | Push files to s3                                          |
| `github-push-s3-cms`                     | TYPE       | YES               | Push files to s3 from Github                              |
| `remote-build-images-forum`              | TAG        |                   | Build the remote Docker images for the forum service      |
| `remote-push-image-forum`                | TAG        |                   | Push local images to the remote image registry            |
| `remote-push-images-authenticated-forum` | TAG        | YES               | Push local images to the remote image registry for Github |
| `remote-build-images-web`                | TAG        |                   | Build the remote Docker images for the web service        |
| `remote-push-images-web`                 | TAG        |                   | Push local images to the remote image registry            |
| `remote-push-images-authenticated-web`   | TAG        | YES               | Push local images to the remote image registry for Github |
| `backup-remote-build-image`              | TAG        |                   | Build the remote Docker images for the backup service     |
| `backup-remote-push-image`               | TAG        |                   | Push local images to the remote image registry            |
| `install`                                |            |                   | Install the requirements to manage infrastructure         |
| `fmt-check`                              |            |                   | All format checks.                                        |
| `fmt-check-infrastructure`               |            |                   | Run cnf and yaml lint on infrastructure aws code.         |
| `django/collectstatic`                   | ENV        | YES               | Run collect static on remote webhead.                     |
| `django/migrate`                         | ENV        | YES               | Run migrate on remote webhead.                            |
| `prod-console`                           |            | YES               | Container shell for private vpc instances.                |
| `log`                                    | INSTANCE   | YES               | Get the log for the web service.                          |




## :mag: Code Linting & Formatting

To maintain code consistency and ensure proper code formatting the following tools are used:

  * [cfn-lint](https://github.com/aws-cloudformation/cfn-lint)
  * [yamllint](https://github.com/adrienverge/yamllint)

To run lint and format checks:

```console
make fmt-check
```




## :file_folder: Folder Structure

    .
    └── infrastructure/
        ├── .images
        ├── backup-parts/
        │   ├── backup.sh
        │   ├── Dockerfile
        │   ├── README.md
        │   └── scheduled-task.yaml
        ├── backup.yaml
        ├── cluster-parts/
        │   ├── database.yaml
        │   ├── ecs.yaml
        │   ├── filesystem.yaml
        │   ├── lifecyclehook.yaml
        │   ├── load-balancer.yaml
        │   ├── security-groups.yaml
        │   └── vpc.yaml
        ├── cluster.yaml
        ├── cms-cert.yaml
        ├── cms.yaml
        ├── config.hls.yaml
        ├── config.yaml
        ├── deploy-pipeline-parts/
        │   └── deploy-notification/
        │       ├── deploy_notification.py
        │       └── deploy_notification_lambda.py
        ├── deploy-pipeline.yaml
        ├── deploy-user.yaml
        ├── docker/
        │   ├── Dockerfile.web.remote
        │   └── forum/
        │       ├── discourse/
        │       │   ├── bin
        │       │   ├── cids
        │       │   ├── containers/
        │       │   │   └── web_only.yml
        │       │   ├── launcher
        │       │   ├── LICENSE
        │       │   ├── plugins
        │       │   ├── README.md
        │       │   ├── shared
        │       │   └── templates/
        │       │       ├── web.ratelimited.template.yml
        │       │       └── web.template.yml
        │       ├── discourse-verification-academy-theme
        │       └── docker-compose-temp-build.yml
        ├── forum.yaml
        ├── legacy.yaml
        ├── Makefile
        ├── README.md
        ├── requirements.txt
        ├── resources.yaml
        ├── support-center-bucket.yaml
        ├── tracking-parts/
        │   ├── tracking-deploy-pipeline.yaml
        │   └── tracking-deploy-script/
        │       └── tracking-deploy-script.py
        ├── tracking.yaml
        ├── web-cert.yaml
        ├── web-parts/
        │   ├── cloudfront.yaml
        │   ├── queue.yaml
        │   ├── static-hosting.yaml
        │   ├── web-deploy-pipeline.yaml
        │   └── web-deploy-script/
        │       └── web-deploy-script.py
        └── web.yaml




## :link: Footnotes

[^1]:
    https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html#https-requirements-aws-region
