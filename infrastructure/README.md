# Verification Academy Infrastructure

![GitHub Action badge](https://github.com/ombu/va-2023/workflows/build/badge.svg)



## :bulb: 10,000 foot Services Overview

![Services Overview](https://github.com/benhartig/temp/blob/main/infrastructure/.images/flow-of-services.png?raw=true)

## :book: Table Of Contents

- [Installation](#toolbox-installation)
- [Notes](#speech_balloon-notes)
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
- [Code Linting & Formatting](#mag-code-linting--formatting)
- [`stacked` Commands](#gear-stacked-commands)
- [MakeFile Commands](#gear-makefile-commands)
- [Footnotes](#link-footnotes)


## :toolbox: Installation


## :speech_balloon: Notes

> [!WARNING]
> This stack creates CloudFront Distrubutions in a different region than
`us-east-1`. CloudFront Distrubution Certificates created by the `cms-cert`
stack must be launched from `us-east-1` [^1] and provide the `cms`
stack the ARN of the certificate.




## :classical_building: Stack Architecture

## :bulb: 10,000 foot Stacks Overview

![Stacks Overview](https://github.com/benhartig/temp/blob/main/infrastructure/.images/flow-of-stacks.png?raw=true)

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




## :mag: Code Linting & Formatting

To maintain code consistency and ensure proper code formatting the following tools are used:

  * [cfn-lint](https://github.com/aws-cloudformation/cfn-lint)
  * [yamllint](https://github.com/adrienverge/yamllint)

To run lint and format checks:

```console
make fmt-check
```

## :gear: `stackedup` Commands


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




## :link: Footnotes

[^1]:
    https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html#https-requirements-aws-region
