## 🛡️ Trivy Orb

A CircleCI orb for Trivy-related operations and workflows.

---

## 🔍 Overview

This orb provides reusable commands for installing and running [Trivy](https://github.com/aquasecurity/trivy) scans within CircleCI pipelines. It simplifies container image security scanning and promotes best practices for DevSecOps in CI/CD workflows.

---

## ✅ Prerequisites

* CircleCI CLI installed and configured
* Valid CircleCI account with orb publishing permissions
* Docker installed (on `machine` executor or remote Docker)

---

## 🛠️ Installing CircleCI CLI

### macOS (using Homebrew)

```bash
brew install circleci
```

### Linux/macOS (using curl)

```bash
curl -fLSs https://raw.githubusercontent.com/CircleCI-Public/circleci-cli/master/install.sh | bash
```

### Windows (using Chocolatey)

```bash
choco install circleci-cli
```

### Alternative: Download Binary

See the [CircleCI CLI releases](https://github.com/CircleCI-Public/circleci-cli/releases)

---

## 🔧 CircleCI CLI Setup

```bash
circleci setup
```

You'll be prompted to:

1. Paste your CircleCI API token (create one at [https://app.circleci.com/settings/user/tokens](https://app.circleci.com/settings/user/tokens))
2. Choose your host (`https://circleci.com`)

Verify:

```bash
circleci version
```

---

## 📦 Installation

To use this orb in your `.circleci/config.yml`:

```yaml
version: 2.1

orbs:
  trivy: iamanonymous419/trivy@1.0.0
```

🧭 [View in Orb Registry](https://circleci.com/developer/orbs/orb/iamanonymous419/trivy)

---

## 🧪 Development

### 🔁 Set Up Dev Environment

```bash
git clone https://github.com/iamanonymous419/orbs.git
cd orbs/trivy
```

### 🧹 Build and Validate

```bash
circleci orb pack src > orb.yml
circleci orb validate orb.yml
```

### 🚀 Publish the Orb

#### First Time Only (Create Namespace)

```bash
circleci orb namespace create iamanonymous419 iamanonymous419 github
```

#### Publish a Dev Version

```bash
circleci orb publish orb.yml iamanonymous419/trivy@dev:first
```

#### Promote to Production

```bash
circleci orb publish promote iamanonymous419/trivy@dev:first patch
```

#### Publish a Specific Version

```bash
circleci orb publish orb.yml iamanonymous419/trivy@1.0.0
```

---

## 📁 Orb Structure

```
src/
├── commands/
│   ├── install.yml     # Trivy installer
│   └── scan.yml        # Trivy scan logic
├── @orb.yml            # Orb metadata
├── @examples/          # Usage example configs
```

---

## 🔧 Commands

### `trivy/install`

Installs the latest Trivy CLI locally.

No parameters needed.

---

### `trivy/scan`

Scans a Docker image with Trivy and outputs a report.

| Parameter  | Type   | Default            | Description                                   |
| ---------- | ------ | ------------------ | --------------------------------------------- |
| `image`    | string | **required**       | Docker image to scan (e.g., `nginx:alpine`)   |
| `output`   | string | `trivy-report.txt` | Output file name                              |
| `format`   | string | `table`            | Report format: `table`, `json`, `sarif`, etc. |
| `severity` | string | `HIGH,CRITICAL`    | Severity levels to include in report          |

---

## 🧰 Example Job

```yaml
jobs:
  scan-app-image:
    machine: true
    steps:
      - checkout
      - run:
          name: Build App Docker Image
          command: docker build -t myuser/app:latest .
      - trivy/install
      - trivy/scan:
          image: myuser/app:latest
          output: app-scan.txt
          format: table
          severity: HIGH,CRITICAL
      - store_artifacts:
          path: trivy-reports/app-scan.txt
          destination: trivy-report
```

---

## 💡 Recommended Executor

Trivy works best in `machine` executors or Docker-enabled environments:

```yaml
executors:
  docker-executor:
    machine:
      image: ubuntu-2204:current
    working_directory: ~/repo
```

---

## 🔁 Full Example Workflow

```yaml
version: 2.1

orbs:
  trivy: iamanonymous419/trivy@1.0.0

jobs:
  trivy-full-scan:
    machine: true
    steps:
      - checkout
      - run:
          name: Build Docker Image
          command: docker build -t myorg/chat-server:latest ./server
      - trivy/install
      - trivy/scan:
          image: myorg/chat-server:latest
          output: server-report.txt
          format: table
          severity: HIGH,CRITICAL
      - store_artifacts:
          path: trivy-reports/server-report.txt
          destination: trivy-server-report

workflows:
  scan-workflow:
    jobs:
      - trivy-full-scan
```

---

## 📌 Version History

| Version     | Description               |
| ----------- | ------------------------- |
| `1.0.0`     | Initial stable release    |
| `dev:first` | First development version |

---

## 📬 Support

* Open an issue at [GitHub Issues](https://github.com/iamanonymous419/orbs/issues)
* Refer to [CircleCI Orb Docs](https://circleci.com/docs/2.0/orb-intro/)
* Explore the [Orb Registry](https://circleci.com/developer/orbs)

---

## 📄 License

MIT License. See `LICENSE` file.
