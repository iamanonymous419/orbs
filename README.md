# 🧩 CircleCI Orbs Collection

This repository contains a set of reusable, production-ready [CircleCI orbs](https://circleci.com/orbs/) built by [`@iamanonymous419`](https://github.com/iamanonymous419) to simplify Docker workflows, security scanning with Trivy, and static code analysis using SonarCloud.

## 📁 Project Structure

```
orbs/
├── docker/         # Docker-related build, tag, push, login commands
├── sonar/          # SonarCloud integration for static code analysis
├── trivy/          # Trivy CLI installer and scanner jobs
├── LICENSE         # MIT License
└── README.md       # You're here
```

Each folder contains:
- `src/` — Orb definitions (`commands`, `jobs`, `executors`, `examples`)
- `@orb.yml` — Metadata used for publishing to CircleCI Orb Registry


## 🔧 Available Orbs

### 📦 [`iamanonymous419/docker`](https://circleci.com/developer/orbs/orb/iamanonymous419/docker)
Reusable Docker automation orb to:
- Log in to DockerHub securely
- Build Docker images from custom paths
- Tag Docker images
- Push images to remote registry

**Usage Example**
```yaml
orbs:
  docker: iamanonymous419/docker@1.0.0

jobs:
  build-image:
    docker:
      - image: cimg/base:stable
    steps:
      - checkout
      - docker/login:
          username: DOCKERHUB_USERNAME
          password: DOCKERHUB_PASSWORD
      - docker/build:
          image-name: myuser/app
          tag: build-123
          dockerfile-path: ./Dockerfile
          build-context: .
      - docker/tag:
          image-name: myuser/app
          old-tag: build-123
          new-tag: latest
      - docker/push:
          image-name: myuser/app
          tag: latest
```

### 🔐 [`iamanonymous419/trivy`](https://circleci.com/developer/orbs/orb/iamanonymous419/trivy)
Security scanner orb using [Trivy](https://github.com/aquasecurity/trivy). Performs vulnerability scans on container images.

**Usage Example**
```yaml
orbs:
  trivy: iamanonymous419/trivy@1.0.0

jobs:
  scan-image:
    machine: true
    steps:
      - checkout
      - trivy/install
      - trivy/scan:
          image: myuser/app:latest
          output: trivy-report.txt
          format: table
          severity: HIGH,CRITICAL
```

### 🧠 [`iamanonymous419/sonar`](https://circleci.com/developer/orbs/orb/iamanonymous419/sonar)
Orb for static analysis with [SonarCloud](https://sonarcloud.io). Easily integrate quality gates into your pipeline.

**Usage Example**
```yaml
orbs:
  sonar: iamanonymous419/sonar@1.0.0

workflows:
  version: 2
  sonar-scan:
    jobs:
      - sonar/scan:
          project_key: iamanonymous419_mern-chat-app
          organization: iamanonymous419
          sources: .
          exclusions: "**/node_modules/**,**/dist/**,**/build/**"
          host_url: https://sonarcloud.io
          sonar_token: SONAR_TOKEN
```

## 🚀 Publishing a New Orb

1. **Pack orb source into `orb.yml`:**
   ```bash
   circleci orb pack src > orb.yml
   ```

2. **Validate the orb:**
   ```bash
   circleci orb validate orb.yml
   ```

3. **Publish a development version:**
   ```bash
   circleci orb publish orb.yml iamanonymous419/<orb-name>@dev:first
   ```

4. **Promote to a production version:**
   ```bash
   circleci orb publish promote iamanonymous419/<orb-name>@dev:first patch
   ```

## 📝 License

This project is open-sourced under the MIT License. See [LICENSE](./LICENSE) for details.

## 🙋‍♂️ Author

Made with 💻 by [`@iamanonymous419`](https://github.com/iamanonymous419)  
Feel free to contribute or open issues for improvements.
