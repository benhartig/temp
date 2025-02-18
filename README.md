# Navex Sanity Studio Infrastructure

[Sanity.io](https://www.sanity.io/) provides projects a single 
[Sanity Studio](https://www.sanity.io/studio) that they host and provides access
to a single dataset at a time. Recently Sanity.io has provided a new way to
access other datasets from their hosted Sanity Studio call
[Workspaces](https://www.sanity.io/docs/workspaces). However this doesn't 
provided the flexablity to deploy different configurations of the Sanity Studio
for  different datasets.

To expand the flexablity of providing different instances of the Sanity Studio
for different datasets this infrastructure stack will create a S3 backed
Cloudfront Distrubution and host a compiled version of the Sanity Studio
from `./studio`.




## :book: Table Of Contents

- [Installation](#toolbox-installation)
- [Notes](#speech_balloon-notes)
- [Stack Types](#beginner-stack-types)
    - [cms](#cms-1)
        - [Launch](#launch)
        - [Update](#update)
        - [Details](#details)
    - [cms-cert](#cms-cert)
        - [Launch](#launch-1)
        - [Update](#update-1)
        - [Details](#details-1)
- [Stack Architecture](#classical_building-stack-architecture)
    - [cms](#cms-2)
    - [cms-cert](#cms-cert-1)
- [Code Linting & Formatting](#mag-code-linting--formatting)
- [MakeFile Commands](#gear-makefile-commands)
- [Folder Structure](#file_folder-folder-structure)



## :sparkles: Quick Commands

> ### cms

```console
TYPE=testing make remote-push-s3-cms
```

```console
TYPE=staging make remote-push-s3-cms
```




## :toolbox: Installation


## :speech_balloon: Notes

> [!IMPORTANT]
> This infrastruture is for **testing** and **staging** instances.
See `./studio/README.md` for **production** deployments to Sanity hosted studio.

> [!WARNING]
> This stack creates CloudFront Distrubutions in a different region than
`us-east-1`. CloudFront Distrubution Certificates created by the `cms-cert`
stack must be launched from `us-east-1` [^1] and provide the `cms`
stack the ARN of the certificate.




## :beginner: Stack Types

> [!NOTE]
> This infrastructure does not contain any actual ECS clusters and uses the 
`clusters:` configuration to direct what region and which account CloudFormation
stacks are launched.

> ### cms-cert

#### Launch

#### Update

#### Details

> ### cms

#### Launch

#### Update

#### Details




## :classical_building: Stack Architecture

> ### cms-cert

    ┌──────────────┐
    │              │
    │ Certifica... │
    │              │
    │ Certificate  │
    │              │
    └──────────────┘

> ### cms

    ┌──────────────┐
    │              │
    │ Certifica... │
    │              │
    │ Certificate  │
    │              │
    └──────────────┘


    ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
    │              │   │              │   │              │   │              │
    │ S3BucketP... │   │   S3Bucket   │   │ Cloudfron... │   │ Route53Re... │
    │              ├──►│              │◄──┤              │◄──┤              │
    │ BucketPolicy │   │    Bucket    │   │ Distribution │   │ RecordSetGr. │
    │              │   │              │   │              │   │              │
    └──────┬───────┘   └──────────────┘   └──────┬───────┘   └──────────────┘
           │                                     │
           │                                     ▼
           │                              ┌──────────────┐
           │                              │              │
           │                              │ Cloudfron... │
           └─────────────────────────────►│              │
                                          │ CloudfontO.. │
                                          │              │
                                          └──────────────┘




## :mag: Code Linting & Formatting

To maintain code consistency and ensure proper code formatting the following tools are used:

  * [cfn-lint](https://github.com/aws-cloudformation/cfn-lint)
  * [yamllint](https://github.com/adrienverge/yamllint)

To run lint and format checks:

```console
make fmt-check
```




## :gear: MakeFile Commands

| Command                           | Needed ENV | Description                                                          |
| --------------------------------- | ---------- | -------------------------------------------------------------------- |
| `make help`                       |            | Show help and list make commands                                     |
| `make remote-build-s3-cms`        |    TYPE    | Build the public Sanity Studio files                                 |
| `make remote-push-s3-cms`         |    TYPE    | Push files to S3 and create invalidation for Cloudfront distrubution |
| `make github-push-s3-cms`         |    TYPE    | Special command when using Github Actions for `remote-push-s3-cms`   |
| `make install`                    |            | Install the requirements to manage infrastructure                    |
| `make fmt-check`                  |            | Run all format checks                                                |
| `make fmt-check-infrastructure`   |            | Run cfn-init and yamllint on infrastructure AWS code                 |




## :file_folder: Folder Structure

    .
    ├── infrastructure/
    │   ├── cms-cert.yaml          # CMS Cert Stack
    │   ├── cms.yaml               # CMS Cloudfront Hosted Stack
    │   ├── config.yaml            # Stack configurations
    │   ├── Makefile
    │   ├── README.md
    │   └── requirements.txt
    ├── preview/
    ├── README.md
    ├── studio/
    └── website/




[^1]:
    https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html#https-requirements-aws-region
