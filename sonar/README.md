## 📡 Sonar Orb

A CircleCI orb for SonarQube or SonarCloud static code analysis.

---

## 🔍 Overview

This orb provides a reusable job for running [SonarQube/SonarCloud](https://www.sonarsource.com/) scans within CircleCI pipelines. It automates the installation and execution of the scanner and supports flexible project configurations.

---

## ✅ Prerequisites

* CircleCI CLI installed and configured
* Valid SonarCloud/SonarQube token
* Your project should already be set up in [SonarCloud](https://sonarcloud.io/) or your own SonarQube server
* Node.js project (scanner is installed via npm)

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

1. Paste your CircleCI API token ([Generate one here](https://app.circleci.com/settings/user/tokens))
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
  sonar: iamanonymous419/sonar@1.0.0
```

🧭 [View in Orb Registry](https://circleci.com/developer/orbs/orb/iamanonymous419/sonar)

---

## 🧪 Development

### 🔁 Set Up Dev Environment

```bash
git clone https://github.com/iamanonymous419/orbs.git
cd orbs/sonar
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
circleci orb publish orb.yml iamanonymous419/sonar@dev:first
```

#### Promote to Production

```bash
circleci orb publish promote iamanonymous419/sonar@dev:first patch
```

#### Publish a Specific Version

```bash
circleci orb publish orb.yml iamanonymous419/sonar@1.0.0
```

---

## 📁 Orb Structure

```
src/
├── jobs/
│   └── scan.yml        # Reusable job to scan with Sonar
├── @orb.yml            # Orb metadata
├── @examples/          # Usage example configs
├── executors/          # Usage executor configs
```

---

## 🔧 Job: `sonar/scan`

This job installs the `sonarqube-scanner` package and runs a static code scan based on the provided configuration.

| Parameter      | Type           | Default                                       | Description                                      |
| -------------- | -------------- | --------------------------------------------- | ------------------------------------------------ |
| `project_key`  | string         | **required**                                  | Your Sonar project key                           |
| `organization` | string         | **required**                                  | Your Sonar organization name                     |
| `sources`      | string         | `"."`                                         | Source directory to scan                         |
| `exclusions`   | string         | `"**/node_modules/**,**/dist/**,**/build/**"` | Glob patterns to exclude                         |
| `host_url`     | string         | `"https://sonarcloud.io"`                     | Host URL for SonarQube or SonarCloud             |
| `sonar_token`  | env\_var\_name | `SONAR_TOKEN`                                 | Environment variable containing your Sonar token |
| `executor`     | executor       | `default`                                     | Executor to use for the job                      |

---

## 🧰 Example Usage

```yaml
version: 2.1

orbs:
  sonar: iamanonymous419/sonar@1.0.0

workflows:
  scan-workflow:
    jobs:
      - sonar/scan:
          project_key: "iamanonymous419_mern-chat-app"
          organization: "iamanonymous419"
          sources: "."
          exclusions: "**/node_modules/**,**/dist/**,**/build/**"
```

---

## 💡 Recommended Executor

Use a simple Node-compatible Docker image:

```yaml
executors:
  default:
    docker:
      - image: cimg/node:18.20.2
    working_directory: ~/project
```

---

## 🔁 Full Example Workflow

```yaml
version: 2.1

orbs:
  sonar: iamanonymous419/sonar@1.0.0

executors:
  default:
    docker:
      - image: cimg/node:18.20.2
    working_directory: ~/project

workflows:
  sonar-analysis:
    jobs:
      - sonar/scan:
          project_key: "my-project-key"
          organization: "my-org"
          sources: "."
          exclusions: "**/node_modules/**,**/dist/**,**/build/**"
```

---

## 📌 Version History

| Version     | Description               |
| ----------- | ------------------------- |
| `1.0.0`     | Initial stable release    |
| `dev:first` | First development version |

---

## 📬 Support

* Open an issue on [GitHub Issues](https://github.com/iamanonymous419/orbs/issues)
* Refer to [CircleCI Orb Docs](https://circleci.com/docs/2.0/orb-intro/)
* Browse the [Orb Registry](https://circleci.com/developer/orbs)

---

## 📄 License

MIT License. See `LICENSE` file.
