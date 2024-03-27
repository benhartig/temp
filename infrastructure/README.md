# Verification Academy Infrastructure

![GitHub Action badge](https://github.com/ombu/va-2023/workflows/build/badge.svg)

The Verification Academy is a valuable resource for individuals looking to
enhance their understanding of advanced functional verification. It offers
various learning tools and resources to help organizations develop their
verification process capabilities. The following is the infrastructure to run
the https://verificationacademy.com/ and subsequent sites.

## :bulb: 10,000 foot Services Overview for each site

![Services Overview](https://github.com/ombu/va-2023/blob/develop/infrastructure/.images/flow-of-services.png?raw=true)

## :book: Table Of Contents

- [Installation](#toolbox-installation)
- [External Services](#diamond_shape_with_a_dot_inside-external-services)
- [Building and Deploying Services](#rocket-building-and-deploying-services)
    - [cms (Sanity Studio)](#cms)
    - [forum (Discourse)](#cms)
    - [tracking (Go)](#cms)
    - [web (Django)](#cms)
- [Stack Architecture](#classical_building-stack-architecture)
- [Stack Types](#beginner-stack-types)
    - [backup](#backup)
    - [cluster](#cluster)
    - [cms](#cms)
    - [cms-cert](#cms-cert)
    - [deploy-pipeline](#deploy-pipeline)
    - [deploy-user](#deploy-user)
    - [forum](#forum)
    - [legacy](#legacy)
    - [legacy-db](#legacy-db)
    - [resources](#resources)
    - [support-center-bucket](#support-center-bucket)
    - [tracking](#tracking)
    - [web](#web)
    - [web-cert](#web-cert)
- [Misc AWS Resources Deployed](#mag-misc-aws-resources-deployed)
- [Website Parameters](#key-website-parameters)
- [Notes](#speech_balloon-notes)
- [`stacked` Commands](#gear-stacked-commands)
- [MakeFile Commands](#gear-makefile-commands)
- [Code Linting & Formatting](#mag-code-linting--formatting)
- [Folder Structure](#file_folder-folder-structure)




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
>
> ```console
> # SSH over AWS Systems Manager Session Manager
> host i-* mi-*
>     ProxyCommand sh -c "aws ssm start-session --target %h --document-name AWS-StartSSHSession --parameters 'portNumber=%p'"
> ```




## :diamond_shape_with_a_dot_inside: External Services

The following external services are 3rd party integrations with the site that
need to be setup and configured before stacks are launched.

* [Adobe Marketo Engage](https://marketo.com) - marketing engagement
* [Algolia](https://www.algolia.com) - search backend
* [Cloudinary](https://cloudinary.com) - media hosting
* [Cookiebot](https://www.cookiebot.com) - GDPR popup
* [Docker](https://www.docker.com) - Image building
* [Github](https://github.com) (optional) - CICD 
* [Google Analytics](https://analytics.google.com) - analytics tracking
* [Sanity.io](https://www.sanity.io) - cms and content backend
* [Sentry.io](https://sentry.io) (optional) - error reporting
* [Slack](https://slack.com) (optional) - build notifications




## :rocket: Building and Deploying Services

> ### cms (Sanity Studio)

#### Controlling version
#### Building a release images
#### Updating a service with a release

> ### forum (Discourse)

#### Controlling version and configuration
#### Building a release images
#### Updating a service with a release

> ### tracking (Go)

#### Tagging a release

[!NOTE]
This is done in the `va-activity-tracking` repository

```console
git checkout main
git merge --no-ff --log=50 develop
git tag v1.0.0
git push origin v1.0.0
git push origin main
```

#### Building a release images

[!NOTE]
This is done in the `va-activity-tracking` repository

When in the release `main` branch navigate to `/infrastructure` to build and
push a ECR image with the following:

```console
TAG=v1.0.0 make remote-push-images-web
```

#### Updating a service with a release

[!NOTE]
This is done in main `va-2023` repository

After the image is pushed to ECR update `ImageTag` of the service you want
updated in the `config.yaml` file.

Example config excerpt:
```
instances:
  production:
    tracking:
      stack_name: va2023-tracking-production-20231211241
      parameters:
        ImageTag: 'v1.0.0'
```

Then issue the following command to udpate the service.

```console
stack-update <service> <environment>
stack-update tracking production
```

> [!NOTE]
> The `testing` environment have been configured with automatic deployments when
an ECR imaged tagged `latest` gets pushed. No need `stack-update` testing.

> ### web (Django)

#### Tagging a release

```console
git checkout main
git merge --no-ff --log=50 develop
git tag v1.0.0
git push origin v1.0.0
git push origin main
```

#### Building a release images

When in the release `main` branch navigate to `/infrastructure` to build and
push a ECR image with the following:

```console
TAG=v1.0.0 make remote-push-images-web
```

#### Updating a service with a release

After the image is pushed to ECR update `ImageTag` of the service you want
updated in the `config.yaml` file.

Example config excerpt:
```
instances:
  production:
    web:
      stack_name: va2023-web-production-20231211241
      parameters:
        ImageTag: 'v1.0.0'
```

Then issue the following command to udpate the service.

```console
stack-update <service> <environment>
stack-update web production
```

> [!NOTE]  
> The `testing` environment have been configured with automatic deployments when
an ECR imaged tagged `latest` gets pushed. No need `stack-update` testing.




## :classical_building: Stack Architecture

## :bulb: 10,000 foot Stack Overview

![Stack Overview](https://github.com/ombu/va-2023/blob/develop/infrastructure/.images/flow-of-stacks.png?raw=true)

## :beginner: Stack Types

> ### backup

Since sites are deployed as ECS containers files are stored centrally in EFS.
This stack will create a new S3 bucket and backup EFS to S3 each night. Stack
needs to only deployed one time per cluster.

Stack(s) currently deployed:

* va2023-cluster-backup-production-20231211241
* va2023-cluster-backup-development-20240326656
* hls-cluster-backup-production-20240228224
* hls-cluster-backup-development-20240326656

> ### cms

Stack creates an S3 bucket and Cloudfront Distribution to self host the 
Sanity.io Studio for an environment.

Stack(s) currently deployed:

* va2023-cms-testing-20240325429
* va2023-cms-beta-202403241256
* va2023-cms-production-20231211241
* hls-cms-testing-20240325429
* hls-cms-staging-20230724615
* hls-cms-production-20231211241

> ### cms-cert

Creates a certificate in the AWS Certificate Manager for use in a Cloudfront
Distribution for the hosted Sanity.io Studio cms stack.

Stack(s) currently deployed:

* va2023-cms-cert-production-20230107900
* va2023-cms-cert-production-20231211241
* hls-cms-cert-staging-20240228224
* hls-cms-cert-production-20240228224

> ### cluster

Main stack to create the VPC with an Internet Gateway and private subnets to
host non public EC2 ECS hosts, EFS, Postgres database and subgroups.

> [!IMPORTANT]
> CSO team runs a nightly bot to detect compliant AWS resources
and recommend that EC2 instances stay up to date with the latest AMI releases.
You should be getting emails that highlight the compliant window if EC2
instances are detected to have high or critical vulnerabilities. If not
remedied the EC2 instances will be automatically shutdown within the current
policy time of 30 days for critical and 60 days for high vulnerabilities. 
The cluster stack uses rolling AMI so to update just issue the following command
to force new EC2 instances and transfer ECS to the new instances. You can monitor
the status of instances also at
[AWS Inspector By instance](https://eu-west-1.console.aws.amazon.com/inspector/v2/home?region=eu-west-1#/findings/instance).
>
> ```console
> stack-update cluster <type>
> stack-update cluster development
> ```

Stack(s) currently deployed:

* va2023-cluster-development-20240324733
* va2023-cluster-production-20231211241
* hls-cluster-development-20240322709
* hls-cluster-production-20240228224

> ### deploy-pipeline

Notifies Slack about ECR image pushes. 

Stack(s) currently deployed:

* va2023-web-deploy-pipline-20230824740

> ### deploy-user

Creates a OIDCProvider for Github to push ECR images, upload files to the S3
CMS buckets and create CloudFront Invalidation.

Stack(s) currently deployed:

* va2023-web-deploy-user-20230824740

> ### forum

ECS task that deploys a redis and discourse container.

Stack(s) currently deployed:

* va2023-forum-testing-20240325558
* va2023-forum-beta-202403241256
* va2023-forum-production-20231211241
* hls-forum-testing-20230129741
* hls-forum-staging-20240228224
* hls-forum-production-20240228224

> ### legacy

This stack hosts the legacy verificationacademy.com 1.0 site and solr search.

Stack(s) currently deployed:

* va-legacy-web-202402241039

> ### legacy-db

This stack creates a MySQL database that the legacy verificationacademy.com can
be deployed onto.

Stack(s) currently deployed:

* va2023-legacy-database-202402241039

> ### resources

Creates a centralized S3 bucket for use in `Resource Type` content.

Stack(s) currently deployed:

* va2023-resources-production-202306220348
* hls-resources-production-20240228224

> ### support-center-bucket

Mimics the support center bucket and allows the Django app to store formatted
json files for the support center app.

Stack(s) currently deployed:

* va2023-support-center-bucket-20230731505
* hls-support-center-bucket-20240228224

> ### tracking

ECS task that deploys a tracking go application, data warehouse and S3 exports.

Stack(s) currently deployed:

* va2023-tracking-testing-20240325613
* va2023-tracking-beta-202403241256
* va2023-tracking-production-20231211241
* hls-tracking-testing-20240325613
* hls-tracking-staging-20240228224
* hls-tracking-production-20240228224

> ### web-cert

Creates a certificate in the AWS Certificate Manager for use in a Cloudfront
Distribution for the Django web stack.

Stack(s) currently deployed:

* va2023-web-static-cert-production-202308031108
* va2023-web-static-cert-production-20231211241
* hls-web-static-cert-staging-20240228224
* hls-web-static-cert-production-20240228224

> ### web

ECS task that deploys a the main Django application.

Stack(s) currently deployed:

* va2023-web-testing-20240325713
* va2023-web-beta-202403241256
* va2023-web-production-20231211241
* hls-web-testing-20240325713
* hls-web-staging-20240228224
* hls-web-production-20240228224




## :mag: Misc AWS Resources Deployed

The following are resources created manually, aren't deployed from this
infrastructure repo or aren't part of any stacks.

> ### CloudFormation Buckets

Buckets created for cloudformation templates

* cloudformation-hls-eu-729799859201
* cloudformation-va2023-eu-729799859201

> ### Forum Amazon SES key

Discoures can't use role SES so SMTP legacy for this user.

> [!WARNING]  
> These keys get rotated every 90 days (enforced by CSO), and you have to
> update the Parameter Store with those new keys and then force a new
> deployment of the forum to get emails going again.

* ses-smtp-va2023-forum-production-20240318-143303

> ### Route 53 Hosted zones

Both verificationacademy.com and hls.academy control the DNS but both are
configured to point the name servers to the following hosted zones.

* verificationacademy.com
* hls.academy

> ### Various Amazon SES Identities

* hls.academy
* verificationacademy.com
* info@hls.academy
* info@verificationacademy.com
* noreply@ombuweb.com

> ### Tracking access IAM user

This IAM user was generated to give Alteryx access to the S3 bucket exports
created by the `tracking` stack.

* TrackerBucketCustodian_va2023_production

> ### OMBU cross-acccount access role

* ombu-administrator

> ### EC2 ssh  key pair

Can be retrived from `EC2EUWest1SSHPairs-2GWE23g242` AWS Secrets Manager

* ombu-aws-eu-west
* hls-aws-eu-west-1
* va-aws-eu-west-1

> ### QuickSight

The following resources were created for use with Amazon QuickSight.

* aws-quicksight-secretsmanager-role-v0
* aws-quicksight-service-role-v0

> ### Legacy Assets

This is a list of assets that were transferred from the original AWS account.

S3 Buckets:

* legacy.verificationacademy.com

EBS Snapshots:

* snap-076b8790c2da565b4
* snap-006fb1e4701ed99d6
* snap-007b0b28d7151e8b2
* snap-01f35942392b0148c
* snap-0c414f945de0094b6
* snap-09957791f87328849
* snap-08870ff2fd8ea5850
* snap-09fb5482a66bac0f8
* snap-0410494f4849a7b49

RDS Snapshots:

* legacy-db-1-mysql-before-move
* prod-db-1-postgres-was-staging-before-relaunch
* va2023-prod-db-postgres-maual-before-cluster-update
* va2023-prod-db-postgres-before-tracking-1-0-3-update
* legacy-db-1-mysql-before-updatedb
* legacy-391166485564-manual-before-instance-upgrade
* legacy-391166485564-vdjji213hgsii0-2020-01-06-09-06-backup-final-snapshot
* legacy-391166485564-vdjji213hgsii0-2020-10-19-09-07-final-snapshot
* legacy-391166485564-vdjji213hgsii0-2020-01-13-09-07-backup-final-snapshot
* legacy-391166485564-vdjji213hgsii0-2021-03-04-23-43-manual-last-5-6
* legacy-391166485564-va-prod-before-encrypt
* legacy-391166485564-vdjji213hgsii0-snapshot-final
* manual-prod-db-1postgres-oct-2-2023




## :key: Website Parameters

For the sites to run the following need to be filled out in AWS Systems Manager
Parameter Store as SecureString.

```
/<site>/<environment>/VAATClientID
/<site>/<environment>/VAATClientSecret
/<site>/<environment>/algolia_api_key
/<site>/<environment>/cloudinary_api_key
/<site>/<environment>/cloudinary_api_secret
/<site>/<environment>/cloudinary_auth_token
/<site>/<environment>/discourse_api_key
/<site>/<environment>/discourse_basic_auth_password
/<site>/<environment>/discourse_basic_auth_username
/<site>/<environment>/discourse_sso_secret
/<site>/<environment>/forum_database_password
/<site>/<environment>/forum_smtp_password
/<site>/<environment>/forum_smtp_username
/<site>/<environment>/marketo_client_id
/<site>/<environment>/marketo_client_secret
/<site>/<environment>/marketo_munchkin_id
/<site>/<environment>/marketo_rest_root
/<site>/<environment>/slack_uri
/<site>/<environment>/support_center_access_key_id
/<site>/<environment>/support_center_secret_access_key
/<site>/<environment>/tracking_database_password
/<site>/<environment>/web_database_password
/<site>/<environment>/web_sanity_token
/<site>/<environment>/web_sentry_dsn
/<site>/<environment>/web_studio_secret
```

> [!TIP]
> All secrets for each environment are stored in System Managers Parameter Store and can retrieve 
with the following command.
>
> ```console
> export AWS_DEFAULT_REGION=eu-west-1
> aws ssm get-parameters-by-path --path /<application>/<environment> --with-decryption --recursive --output table
> aws ssm get-parameters-by-path --path /va2023/testing --with-decryption --recursive --output table
> ```




## :speech_balloon: Notes

> [!IMPORTANT]
> This infrastructure creates CloudFront Distributions in a different region
than `us-east-1`. CloudFront Distribution Certificates created by the `cms-cert`
 and `web-cert` stack must be launched from `us-east-1` [^1], then provide the
`cms` stack and `web` stack with the ARN of the certificates.


> [!NOTE]  
> You don't have to use `aws sts assume-role` or `assume-role` if you are not
coming from another AWS account and have direct AWS access keys.


> [!TIP]
> If you install the [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
from the [Installation](#toolbox-installation) section you can use the 
`instance id` of the EC2 instance to `ssh` and access the containers.
>
> ```console
> ssh -i ~/.ssh/<ssh key> ec2-user@<instance id>
> ```


> [!TIP]
> If you install the [SSM Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-working-with-install-plugin.html)
from the [Installation](#toolbox-installation) section you can use the following
command to start a remote port forward to access the database.
>
> ```console
> aws ssm start-session --region <region> --target <instance id> --document-name AWS-StartPortForwardingSessionToRemoteHost --parameters host="<db host>",portNumber="5432",localPortNumber="5432"
> ```


> [!TIP]
> CMS Cloudfront S3 Bucket Policy sometimes has issues when deploying or
updating and sometimes access denied when placing the policy for the S3 bucket.
Solution is to add this inline policy because the normal S3 policy doesn't seem
to allow for `s3:PutBucketPolicy`.
>
> ```console
> {
>     "Version": "2012-10-17",
>     "Statement": {
>         "Action": [
>             "s3:*",
>             "cloudformation:*"
>         ],
>         "Resource": "*",
>         "Effect": "Allow"
>     }
> }
> ```


> [!TIP]
> To migrate the Discourse forum service you can run the following inside the
forum container.
>
> ```console
> RAILS_ENV=production /usr/local/bin/bundle exec rake db:migrate
> ```


> [!TIP]
> To create a new admin user for the Discourse forum service you can run the
following inside the forum container.
>
> ```console
> RAILS_ENV=production /usr/local/bin/bundle exec rake admin:create
> ```


> [!TIP]
> To update a user to an admin user for the Discourse forum service you can run
the following inside the forum container if email services are down.
>
> ```console
> cd /var/www/discourse
> rails c
> user = User.find_by_email("<admin_email>")
> user.admin = true
> user.approved = true
> user.save
> EmailToken.confirm(user.email_tokens.first.token)
> ```


> [!CAUTION]
> When creating a new installation of the forum service, you might need to reset
database, use the following inside the container to do so, but warning it will
delete everything and reset.
>
> ```console
> DISABLE_DATABASE_ENVIRONMENT_CHECK=1 bundle exec rake db:drop db:create db:migrate
> ```




## :gear: `stackedup` Commands

Infrastructure makes extensive use of [OMBU Stackedup](https://github.com/ombu/stacks) for configuring, packaging, launching and updating
AWS CloudFormation templates in the form of stacks.

| Command                                                           | Options    | Description                                                      |
| ----------------------------------------------------------------- | ---------- | ---------------------------------------------------------------- |
| `assume-role <account>`                                           |            | Assume a role defined in the accounts section of `config.yaml`   |
| `stack-launch <service> <instance>`                               | `--config` | Launch a new stack from the `config.yaml`                        |
| `stack-update <service> <instance>`                               | `--config` | Update an existing stack                                         |
| `stack-details <service> <instance>`                              | `--config` | Get details of an existing stack                                 |
| `container-shell <instance> <service> <service-name> <container>` | `--config` | Shell into an ECS Serviced from a stack defined in `config.yaml` |

> [!TIP]
> This repo contains multiple Academy sites and config files. Currently deployed
are the va and hls sites. To switch between site configuration use the `--config`
flag. Example as follows:
>
> ```console
> stack-update --config config.hls.yaml cluster <type>
> ```




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
| `fmt-check-infrastructure`               |            |                   | Run cfn and yaml lint on infrastructure aws code.         |
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
        ├── .images                                         # images for this readme
        ├── backup-parts/
        │   ├── backup.sh                                   # backup shell script
        │   ├── Dockerfile                                  # docker file for backup task
        │   ├── README.md
        │   └── scheduled-task.yaml                         # scheduled task for backup
        ├── backup.yaml                                     # backup stack for efs of the cluster
        ├── cluster-parts/
        │   ├── database.yaml                               # cluster database
        │   ├── ecs.yaml                                    # ecs cluster
        │   ├── filesystem.yaml                             # efs for each cluster
        │   ├── lifecyclehook.yaml                          # autoscaling policies for cluster
        │   ├── load-balancer.yaml                          # application load balancer
        │   ├── security-groups.yaml                        # cluster security groups
        │   └── vpc.yaml                                    # vpc with internet gateway, private subnets
        ├── cluster.yaml                                    # ecs cluster stack
        ├── cms-cert.yaml                                   # cms cloudfront east-1 cert
        ├── cms.yaml                                        # sanity hosted cms
        ├── config.hls.yaml                                 # hls.academy configuration
        ├── config.yaml                                     # verificationacademy.com configuration
        ├── deploy-pipeline-parts/
        │   └── deploy-notification/
        │       ├── deploy_notification.py
        │       └── deploy_notification_lambda.py
        ├── deploy-pipeline.yaml                            # notifications for image builds
        ├── deploy-user.yaml                                # OIDC provider and role
        ├── docker/
        │   ├── Dockerfile.web.remote                       # web stack django docker file for ECR
        │   └── forum/
        │       ├── discourse/
        │       │   ├── bin
        │       │   ├── cids
        │       │   ├── containers/
        │       │   │   └── web_only.yml                    # discourse custom config
        │       │   ├── launcher                            # discourse build script
        │       │   ├── LICENSE
        │       │   ├── plugins
        │       │   ├── README.md
        │       │   ├── shared
        │       │   └── templates/
        │       │       ├── web.ratelimited.template.yml    # default discourse templete
        │       │       └── web.template.yml                # default discourse templete
        │       ├── discourse-verification-academy-theme
        │       └── docker-compose-temp-build.yml
        ├── forum.yaml                                      # discourse forum service stack
        ├── legacy.yaml                                     # configuration for the drupal vs 1 site
        ├── Makefile
        ├── README.md
        ├── requirements.txt
        ├── resources.yaml                                  # global resource type s3 bucket
        ├── support-center-bucket.yaml                      # mock support center bucket
        ├── tracking-parts/
        │   ├── tracking-deploy-pipeline.yaml               # notifications for deployments
        │   └── tracking-deploy-script/
        │       └── tracking-deploy-script.py
        ├── tracking.yaml                                   # go tracking service stack
        ├── web-cert.yaml                                   # web cloudfront east-1 cert
        ├── web-parts/
        │   ├── cloudfront.yaml                             # web cloudfront configuration
        │   ├── queue.yaml                                  # sqs service for django celery
        │   ├── static-hosting.yaml                         # s3 bucket creation for 
        │   ├── web-deploy-pipeline.yaml                    # notifications for deployments
        │   └── web-deploy-script/
        │       └── web-deploy-script.py
        └── web.yaml                                        # web django service stack




[^1]:
    https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cnames-and-https-requirements.html#https-requirements-aws-region
