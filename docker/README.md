# Docker Orb

A CircleCI orb for Docker-related operations and workflows.

## Overview

This orb provides reusable commands, jobs, and executors for Docker operations in CircleCI pipelines. It simplifies common Docker tasks and promotes best practices for containerized CI/CD workflows.

## Prerequisites

- CircleCI CLI installed and configured
- Valid CircleCI account with orb publishing permissions
- Docker installed locally (for development and testing)
  
### Installing CircleCI CLI

#### macOS (using Homebrew)
```bash
brew install circleci
```

#### Linux/macOS (using curl)
```bash
curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash
```

#### Windows (using Chocolatey)
```bash
choco install circleci-cli
```

#### Alternative: Download Binary
Download the latest release from the [CircleCI CLI releases page](https://github.com/CircleCI-Public/circleci-cli/releases).

### Setting Up CircleCI CLI

After installation, configure the CLI with your CircleCI API token:

```bash
circleci setup
```

This will prompt you to:
1. Enter your CircleCI API token (create one at https://app.circleci.com/settings/user/tokens)
2. Choose your CircleCI host (usually `https://circleci.com`)

Verify the setup:
```bash
circleci version
```

## Installation

To use this orb in your CircleCI configuration:

```yaml
version: 2.1

orbs:
  docker: iamanonymous419/docker@1.0.0
```

View this orb in the CircleCI Orb Registry: https://circleci.com/developer/orbs/orb/iamanonymous419/docker

## Development

### Setting Up the Development Environment

1. Clone this repository
2. Install the CircleCI CLI
3. Navigate to the project directory

```bash
cd docker
```

### Building and Validating the Orb

Pack the orb source files into a single YAML file:

```bash
circleci orb pack src > orb.yml
```

Validate the orb configuration:

```bash
circleci orb validate orb.yml
```

### Publishing Workflow

#### First-Time Setup

Create a namespace (one-time setup):

```bash
circleci orb namespace create iamanonymous419 iamanonymous419 github
```

List available orbs to verify namespace creation:

```bash
circleci orb list
```

Create the orb in your namespace:

```bash
circleci orb create iamanonymous419/docker
```

#### Viewing Your Orbs

List all orbs in your namespace:

```bash
circleci orb list iamanonymous419
```

List uncertified orbs in your namespace:

```bash
circleci orb list iamanonymous419 --uncertified
```

#### Publishing Development Versions

Publish a development version for testing:

```bash
circleci orb publish orb.yml iamanonymous419/docker@dev:first
```

#### Publishing Production Versions

Promote a development version to production:

```bash
circleci orb publish promote iamanonymous419/docker@dev:first patch
```

Or publish a specific version directly:

```bash
circleci orb publish orb.yml iamanonymous419/docker@1.0.0
```

## Orb Structure

```
src/
├── commands/          # Reusable command definitions
└── @orb.yml           # Main orb configuration
└── @examples/         # examples configuration
```

## Commands

### Available Commands

- `login` - Authenticate with Docker registry (Docker Hub)
- `build` - Build Docker image with specified name, tag, and context
- `tag` - Tag existing Docker image with new tag
- `push` - Push Docker image to registry

### Command Parameters

#### `login`
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | Yes | Environment variable containing Docker registry username |
| `password` | env_var_name | Yes | Environment variable containing Docker registry password |

#### `build`
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `image-name` | string | Yes | Name of the Docker image to build |
| `tag` | string | Yes | Tag for the Docker image |
| `dockerfile-path` | string | Yes | Path to the Dockerfile |
| `build-context` | string | Yes | Build context directory |

#### `tag`
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `image-name` | string | Yes | Name of the Docker image |
| `old-tag` | string | Yes | Current tag of the image |
| `new-tag` | string | Yes | New tag to apply to the image |

#### `push`
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `image-name` | string | Yes | Name of the Docker image to push |
| `tag` | string | Yes | Tag of the image to push |

## Jobs

This orb is designed to be used with custom jobs that utilize the provided commands. The orb focuses on providing reusable commands rather than predefined jobs, allowing for maximum flexibility in your CI/CD workflows.

### Example Job Structure

```yaml
jobs:
  build-client:
    executor: docker-executor
    steps:
      - checkout
      - docker/login:
          username: $DOCKERHUB_USERNAME
          password: DOCKERHUB_PASSWORD
      - docker/build:
          image-name: $DOCKERHUB_USERNAME/chat-client
          tag: << pipeline.parameters.image_tag >>
          dockerfile-path: ./client/Dockerfile
          build-context: ./client
      - docker/tag:
          image-name: $DOCKERHUB_USERNAME/chat-client
          old-tag: << pipeline.parameters.image_tag >>
          new-tag: latest
      - docker/push:
          image-name: $DOCKERHUB_USERNAME/chat-client
          tag: latest
```

## Executors

### Recommended Executor

While this orb doesn't provide executors, it works best with machine executors for Docker operations:

```yaml
executors:
  docker-executor:
    machine:
      image: ubuntu-2204:current
    working_directory: ~/repo
```

## Usage Examples

### Complete Multi-Service Build

```yaml
version: 2.1

orbs:
  docker: iamanonymous419/docker@1.0.0

executors:
  docker-executor:
    machine:
      image: ubuntu-2204:current
    working_directory: ~/repo

parameters:
  image_tag:
    type: string
    default: build-<< pipeline.number >>

jobs:
  build-client:
    executor: docker-executor
    steps:
      - checkout
      - docker/login:
          username: $DOCKERHUB_USERNAME
          password: DOCKERHUB_PASSWORD
      - docker/build:
          image-name: $DOCKERHUB_USERNAME/chat-client
          tag: << pipeline.parameters.image_tag >>
          dockerfile-path: ./client/Dockerfile
          build-context: ./client
      - docker/tag:
          image-name: $DOCKERHUB_USERNAME/chat-client
          old-tag: << pipeline.parameters.image_tag >>
          new-tag: latest
      - docker/push:
          image-name: $DOCKERHUB_USERNAME/chat-client
          tag: latest

  build-server:
    executor: docker-executor
    steps:
      - checkout
      - docker/login:
          username: $DOCKERHUB_USERNAME
          password: DOCKERHUB_PASSWORD
      - docker/build:
          image-name: $DOCKERHUB_USERNAME/chat-server
          tag: << pipeline.parameters.image_tag >>
          dockerfile-path: ./server/Dockerfile
          build-context: ./server
      - docker/tag:
          image-name: $DOCKERHUB_USERNAME/chat-server
          old-tag: << pipeline.parameters.image_tag >>
          new-tag: latest
      - docker/push:
          image-name: $DOCKERHUB_USERNAME/chat-server
          tag: latest

workflows:
  build-all:
    jobs:
      - build-client
      - build-server
```

### Single Service Build

```yaml
version: 2.1

orbs:
  docker: iamanonymous419/docker@1.0.0

jobs:
  build-and-push:
    machine:
      image: ubuntu-2204:current
    steps:
      - checkout
      - docker/login:
          username: $DOCKERHUB_USERNAME
          password: DOCKERHUB_PASSWORD
      - docker/build:
          image-name: myapp
          tag: v1.0.0
          dockerfile-path: ./Dockerfile
          build-context: .
      - docker/push:
          image-name: myapp
          tag: v1.0.0

workflows:
  single-build:
    jobs:
      - build-and-push
```

## Configuration Parameters

### Environment Variables Required

| Variable | Description |
|----------|-------------|
| `DOCKERHUB_USERNAME` | Docker Hub username for authentication |
| `DOCKERHUB_PASSWORD` | Docker Hub password or access token |

### Pipeline Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `image_tag` | string | `build-<< pipeline.number >>` | Tag to apply to built images |

### Command Parameters

See the Commands section above for detailed parameter information for each command.

## Version History

- **1.0.0** - Latest stable version
- **dev:first** - Development version for testing

## Support

For issues and questions:

- Create an issue in this repository
- Check the [CircleCI Orb Documentation](https://circleci.com/docs/2.0/orb-intro/)
- Review the [CircleCI Orb Registry](https://circleci.com/developer/orbs)

## License

This orb is licensed under the MIT License. See the LICENSE file for details.

## Changelog

### [1.0.0] - Current
- Initial stable release
- Basic Docker operations support

### [dev:first] - Development
- Initial development version
- Core functionality implementation