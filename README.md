# Navex Sanity Studio Infrastructure

Sanity provides projects a single studio hosted solution. To expand hosting
Sanity Studios for different datasets this infrastructure stack will create S3
hosted Cloudfront Distrubutions then upload a compiled Sanity Studio.

## :book: Table Of Contents

- [Installation](#toolbox-installation)
- [Stack Types](#beginner-stack-types)
    - [cms](#cms)
    - [cms-cert](#cms-cert)
- [Stack Diagram](#classical_building-stack-architecture)
- [Clusters](#books-clusters)
- [Instances](#closed_book-instances)
    - [Launch](#launch-1)
    - [Update](#update-1)
    - [Details](#details-1)
- [Code Linting & Formatting](#mag-code-linting--formatting)
- [MakeFile Commands](#gear-makefile-commands)
- [Folder Structure](#file_folder-folder-structure)
- [Notes](#speech_balloon-notes)

## :toolbox: Installation

## :beginner: Stack Types

> ### cms

> ### cms-cert

## :classical_building: Stack Architecture

## :books: Clusters

## :closed_book: Instances

> ### Launch

> ### Update

> ### Details

## :mag: Code Linting & Formatting

## :gear: MakeFile Commands

| Command                           | ENV Needed | Description                                                          |
| --------------------------------- | ---------- | -------------------------------------------------------------------- |
| `make help`                       |            | Show help and list make commands                                     |
| `make remote-build-s3-cms`        | TYPE       | Build the public Sanity Studio files                                 |
| `make remote-push-s3-cms`         | TYPE       | Push files to S3 and create invalidation for Cloudfront distrubution |
| `make github-push-s3-cms`         | TYPE       | Special command when using Github Actions for `remote-push-s3-cms`   |
| `make install`                    |            | Install the requirements to manage infrastructure                    |
| `make fmt-check`                  |            | Run all format checks                                                |
| `make fmt-check-infrastructure`   |            | Run cfn-init and yamllint on infrastructure AWS code                 |

## :file_folder: Folder Structure

## :speech_balloon: Notes
