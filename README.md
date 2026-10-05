# Jenkins DevSecOps CI/CD Pipeline

This project demonstrates an enterprise-grade **Jenkins-based DevSecOps CI/CD pipeline** for the **Resume Matcher** platform, integrating continuous integration, automated QA, security scanning, cloud-agent execution, containerization, multi-architecture publishing, cryptographic image signing, SBOM generation, supply-chain verification, and automated security reporting.

The implementation combines a locally hosted **Jenkins LTS controller**, **GitHub SCM**, **GitHub Actions Cloud Agents**, **zrok secure HTTPS ingress**, automated pre-main validation, production container publishing, **Cosign image signing**, **SPDX SBOM generation**, image and attestation verification, and automated HTML/email reporting.

The pipeline separates the software lifecycle into two primary paths:

```text
pre-main
   │
   ├── CI
   ├── OWASP
   ├── QA
   ├── SonarCloud
   ├── Trivy
   ├── Gitleaks
   └── Security / QA Reports
             │
             ▼
       Validation Passed
             │
             ▼
           main
             │
             ├── Multi-Arch Docker Build
             ├── GHCR Push
             ├── Docker Hub Push
             ├── Cosign Signing
             ├── SPDX SBOM
             ├── Attestation Verification
             └── Security Evidence
```

---

## Access the Walkthrough

[![Jenkins DevSecOps CI/CD Pipeline](https://img.youtube.com/vi/qBbtWlOH5rg/0.jpg)](https://www.youtube.com/embed/qBbtWlOH5rg?si=y8lQeAAUDfd1_1QU)

[Watch the Jenkins DevSecOps CI/CD Pipeline Walkthrough](https://www.youtube.com/embed/qBbtWlOH5rg?si=y8lQeAAUDfd1_1QU)

---

## 🏗️ Jenkins DevSecOps Strategy

<div align="center">
<img src="images/jenkins/JENKINS 1/jenkins1.1.png" width="1000"/>
</div>

The Jenkins implementation is organized into three major implementation stages:

1. **Jenkins Infrastructure, SCM & Cloud-Agent Setup**
2. **Secure Validation, QA, Security Scanning & Automated Reporting**
3. **Production Container Build, Signing, SBOM & Supply-Chain Verification**

The architecture combines a local Jenkins controller with dynamically provisioned GitHub Actions execution environments and a secure zrok ingress layer.

---

# Step 1: Jenkins Infrastructure, SCM & Cloud-Agent Setup

The first stage establishes the Jenkins control plane, source-control integration, execution environment, GitHub Actions cloud agents, and secure external connectivity.

The Jenkins controller is deployed using the official Jenkins LTS Docker image with persistent Jenkins home storage.

### Jenkins Installation

```bash
docker pull jenkins/jenkins:lts

docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Jenkins is then accessed locally through:

```text
http://localhost:8080
```

The initial setup includes:

- Jenkins LTS initialization
- Plugin installation
- Git integration
- Pipeline support
- SSH Build Agents
- Mailer
- Administrator account configuration

<div align="center">

<img src="images/jenkins/JENKINS 1/jenkins1.png" width="250"/>
<img src="images/jenkins/JENKINS 1/jenkins2.png" width="250"/>
<img src="images/jenkins/JENKINS 1/jenkins3.png" width="250"/>
<img src="images/jenkins/JENKINS 1/jenkins4.png" width="250"/>
<img src="images/jenkins/JENKINS 1/jenkins5.png" width="250"/>
<img src="images/jenkins/JENKINS 1/jenkins6.png" width="250"/>

</div>

---

## Jenkins Pipeline & GitHub SCM

A dedicated Jenkins Pipeline job is configured for the validation branch:

```text
Job:
resume-matcher-jenkins-premain

Repository:
https://github.com/kaifmohammedkhan/resume-matcher-devops

Branch:
*/pre-main

Script Path:
Jenkinsfile
```

The Jenkins pipeline obtains its definition directly from GitHub.

Example SCM configuration:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                git branch: 'pre-main',
                    credentialsId: 'GITHUB_CRED',
                    url: 'https://github.com/kaifmohammedkhan/resume-matcher-devops.git'
            }
        }
    }
}
```

GitHub authentication is handled through Jenkins Credentials rather than storing authentication material directly inside the pipeline.

<div align="center">

<img src="images/jenkins/JENKINS 2/jenkins7.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins8.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins9.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins10.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins11.1.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins11.2.png" width="250"/>
<img src="images/jenkins/JENKINS 2/jenkins12.png" width="250"/>

</div>

---

## Jenkins Controller Tooling

The Jenkins controller environment is prepared with the runtime tools required by the pipeline.

```bash
docker exec -it -u root jenkins /bin/bash

apt-get update
apt-get install -y docker.io

docker --version

apt-get update && apt-get install -y nodejs npm
```

The environment therefore provides:

```text
Docker
Node.js
npm
Application Test Runtime
Container Tooling
Security Tooling
```

<div align="center">

<img src="images/jenkins/JENKINS 3/jenkins15.png" width="250"/>
<img src="images/jenkins/JENKINS 3/jenkins16.png" width="250"/>
<img src="images/jenkins/JENKINS 3/jenkins17.png" width="250"/>

</div>

---

## GitHub Actions Cloud Agents

The Jenkins environment is extended with **GitHub Actions Cloud Agents** to provide dynamically provisioned execution environments.

The cloud configuration is:

```text
Cloud Name       : github-cloud
Provider         : GitHub Actions
Repository       : kaifmohammedkhan/resume-matcher-devops
Credential       : github-agent-token
Agent Label      : gha-runner
Remote FS        : /home/runner/agent
Workflow         : jenkins-agent.yml
Git Ref          : pre-main
```

The execution model is:

```text
Jenkins Controller
       │
       ▼
GitHub Actions Cloud Plugin
       │
       ▼
Jenkins Agent Runner Workflow
       │
       ▼
Ephemeral GitHub Actions Runner
       │
       ▼
Jenkins Agent
       │
       ▼
Pipeline Stage
```

<div align="center">

<img src="images/jenkins/JENKINS 4/jenkins18.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins19.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins20.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins21.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins22.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins23.png" width="250"/>
<img src="images/jenkins/JENKINS 4/jenkins24.png" width="250"/>

</div>

---

## Secure zrok Ingress

Because Jenkins is hosted locally, **zrok** provides an HTTPS ingress endpoint for external GitHub and GitHub Actions communication.

```text
GitHub / GitHub Actions
          │
          ▼
my-jenkins-local.shares.zrok.io
          │
          ▼
         zrok
          │
          ▼
Jenkins :8080
```

The zrok CLI is installed on the Windows host.

```powershell
New-Item -ItemType Directory -Path "C:\zrok" -Force

Invoke-WebRequest `
  -Uri "https://github.com/openziti/zrok/releases/download/v0.4.42/zrok_0.4.42_windows_amd64.tar.gz" `
  -OutFile "C:\zrok\zrok.tar.gz"

tar -xf "C:\zrok\zrok.tar.gz" -C "C:\zrok"

Remove-Item "C:\zrok\zrok.tar.gz"

C:\zrok\zrok.exe version

C:\zrok\zrok.exe enable <zrok-token>
```

<div align="center">

<img src="images/jenkins/JENKINS 5/jenkins25.1.png" width="250"/>
<img src="images/jenkins/JENKINS 5/jenkins25.2.png" width="250"/>
<img src="images/jenkins/JENKINS 5/jenkins25.3.png" width="250"/>

</div>

---

## Persistent zrok Share

A named zrok share provides the stable public Jenkins endpoint:

```text
my-jenkins-local.shares.zrok.io
```

The repository automation script:

```bash
chmod +x jenkins.sh
./jenkins.sh
```

handles the share lifecycle.

The resulting Jenkins URL is:

```text
https://my-jenkins-local.shares.zrok.io/
```

This endpoint is configured as the Jenkins Location URL.

<div align="center">

<img src="images/jenkins/JENKINS 6/jenkins25.4.png" width="250"/>
<img src="images/jenkins/JENKINS 6/jenkins25.5.png" width="250"/>
<img src="images/jenkins/JENKINS 6/jenkins25.6.png" width="250"/>
<img src="images/jenkins/JENKINS 6/jenkins26.2.png" width="250"/>
<img src="images/jenkins/JENKINS 6/jenkins27.jpg" width="250"/>

</div>

---

## Jenkins API & Parameterized Builds

The Jenkins environment also supports external API-triggered builds.

A dedicated Jenkins API token is configured for GitHub Actions communication.

GitHub repository secrets include:

```text
JENKINS_URL
JENKINS_USER
JENKINS_TOKEN
```

The production job is parameterized using:

```text
PR_NUMBER
BRANCH_NAME
```

The resulting integration is:

```text
GitHub Actions
      │
      │ Jenkins API
      ▼
Jenkins
      │
      ▼
Parameterized Pipeline
      │
      ├── PR_NUMBER
      └── BRANCH_NAME
```

<div align="center">

<img src="images/jenkins/JENKINS 8/jenkins32.1.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.2.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.3.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.4.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.5.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.6.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.7.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.8.png" width="250"/>
<img src="images/jenkins/JENKINS 8/jenkins32.9.png" width="250"/>

</div>

---

# Step 2: Secure Validation, QA, Security Scanning & Automated Reporting

The second stage implements the complete **pre-main validation lifecycle**.

The local automation entry point is:

```bash
./premain.sh
```

The script synchronizes the local working tree with the validation branch and pushes the resulting state to GitHub.

The push triggers:

```text
resume-matcher-jenkins-premain
```

The Jenkins pipeline then executes CI, security, QA, static analysis, container scanning, secret detection and reporting.

---

## Pre-main Validation Pipeline

```text
Developer Changes
       │
       ▼
./premain.sh
       │
       ▼
GitHub pre-main
       │
       ▼
Jenkins
       │
       ├───────────────┬───────────────┐
       │               │               │
       ▼               ▼               ▼
     CI Job         OWASP Job        QA Job
       │               │               │
       │               │               ├── Cypress
       │               │               ├── k6
       │               │               └── PostgreSQL
       │               │
       │               └── Dependency Check
       │
       ├── Tests
       ├── Lint
       └── Build
               │
               ▼
         SonarCloud
               │
               ▼
             Trivy
               │
               ├── Filesystem
               └── Image
               │
               ▼
           Gitleaks
               │
               ▼
       Report Consolidation
               │
               ▼
        Email Notification
```

---

## CI Validation

The CI stage validates the application using the project's automated test and build tooling.

The validation layer includes:

```text
Unit Tests
Linting
Application Build
Coverage Reporting
```

The generated CI evidence is preserved as Jenkins artifacts.

---

## OWASP Dependency Security

The OWASP stage performs dependency vulnerability analysis using:

```text
OWASP Dependency-Check
```

The resulting security evidence is consolidated into the pipeline reporting process.

---

## SonarCloud Static Analysis

SonarCloud provides static code analysis for the project.

The Jenkins pipeline executes SonarCloud after the test stage so that code-quality and security analysis can be associated with the current validated source state.

---

## Trivy Security Scanning

Trivy is used at multiple points in the pipeline.

```text
Trivy
 │
 ├── Filesystem Scan
 │
 └── Container Image Scan
```

The resulting reports are preserved for pipeline evidence.

---

## Gitleaks Secret Detection

Gitleaks provides source-level secret detection.

The purpose is to identify accidentally committed:

```text
API Keys
Tokens
Passwords
Credentials
Private Secrets
```

before the source state proceeds through the release lifecycle.

---

## QA Validation

The QA path includes:

```text
Cypress
k6
PostgreSQL
WireMock
```

Cypress performs browser-based end-to-end validation.

k6 provides smoke and load testing.

PostgreSQL supports the application/QA database workflow, while WireMock is used to isolate external API dependencies during testing.

---

## Automated HTML Reporting

The pre-main pipeline consolidates its results into multiple HTML reports.

The documented report set includes:

```text
test-summary-report
sonar-summary-report
trivy-fs-report
trivy-img-report
security-report
dependency-check-report
```

These reports are archived and distributed through automated email.

<div align="center">

<img src="images/jenkins/JENKINS 9/jenkins32.10.png" width="250"/>
<img src="images/jenkins/JENKINS 9/jenkins32.png" width="250"/>
<img src="images/jenkins/JENKINS 9/jenkins33.png" width="250"/>
<img src="images/jenkins/JENKINS 9/jenkins34.png" width="250"/>
<img src="images/jenkins/JENKINS 9/jenkins35.png" width="250"/>
<img src="images/jenkins/JENKINS 9/jenkins36.png" width="250"/>

</div>

---

# Step 3: Production Container Build, Signing, SBOM & Supply-Chain Verification

The third stage is the production release path.

The production Jenkins job:

```text
resume-matcher-main
```

is configured against:

```text
Branch:
*/main

Script Path:
Jenkinsfile
```

The production lifecycle extends container publication with image signing, SBOM generation, attestation verification and machine-readable security evidence.

---

## Production Pipeline

```text
pre-main Validation
        │
        ▼
       main
        │
        ▼
resume-matcher-main
        │
        ├── Checkout
        ├── Environment Setup
        ├── Docker Buildx
        ├── Multi-Architecture Build
        ├── GHCR Push
        ├── Docker Hub Push
        ├── Image Digest Capture
        ├── Cosign Signing
        ├── SPDX SBOM Generation
        ├── Attestation Verification
        ├── Security Evidence
        └── Email Report
```

---

## Multi-Architecture Docker Build

Production container images are built for:

```text
linux/amd64
linux/arm64
```

The resulting architecture manifest allows the published image to support both target platforms.

```text
                 Docker Buildx
                      │
          ┌───────────┴───────────┐
          │                       │
          ▼                       ▼
      linux/amd64             linux/arm64
          │                       │
          └───────────┬───────────┘
                      │
                      ▼
              Multi-Arch Image
                      │
             ┌────────┴────────┐
             ▼                 ▼
           GHCR            Docker Hub
```

---

## Docker Registry Publishing

The production image is published to both:

```text
GitHub Container Registry
Docker Hub
```

The pipeline captures image identity and verification information after publication.

---

## Immutable Image Digest

The production pipeline records the immutable image digest.

An example from the documented pipeline evidence is:

```text
Repository:
kaifmohammedkhan/resume-matcher-devops

Digest:
sha256:661afae7ba52164864492b48b5cfed3eb3890b854c65952b44ed364fae98a72f
```

The digest identifies the exact image artifact associated with that particular pipeline execution.

---

## Cosign Image Signing

Cosign is configured in Jenkins using:

```text
COSIGN_PRIVATE_KEY
COSIGN_PASSPHRASE
```

The private key is stored as a Jenkins Secret File and the passphrase is stored as Secret Text.

The signing process is:

```text
Published Image
      │
      ▼
Immutable Digest
      │
      ▼
Cosign Sign
      │
      ▼
Container Signature
      │
      ▼
Verification Evidence
```

Cosign therefore provides a cryptographic mechanism for associating the release signature with the published image identity.

---

## SPDX SBOM Generation

Syft is used to generate Software Bill of Materials documents for the production container images.

The generated SBOMs use the:

```text
SPDX
```

format.

The SBOM provides machine-readable information describing the software components present within the published image.

The pipeline preserves registry-specific SBOM evidence:

```text
sbom-ghcr.json
sbom-dockerhub.json
```

---

## Attestation Verification

The production pipeline also verifies and preserves attestation information associated with the released container artifacts.

The documented evidence includes:

```text
attestation-verify-ghcr.json
attestation-verify-dockerhub.json
```

This allows the underlying verification evidence to be retained alongside the human-readable reports.

---

## Machine-Readable Security Evidence

The production workflow preserves:

```text
Cosign Verification
SBOM
Attestation Verification
Image Metadata
Digest Information
```

Example artifacts include:

```text
cosign-verify-ghcr.json
cosign-verify-dockerhub.json

attestation-verify-ghcr.json
attestation-verify-dockerhub.json

sbom-ghcr.json
sbom-dockerhub.json
```

---

## Docker Build & Security Reports

The production pipeline generates HTML reporting for both the build/publishing process and the associated security evidence.

```text
docker-build-push-report.html
docker-security-report.html
```

The reports document information such as:

- Multi-architecture build status
- GHCR publication
- Docker Hub publication
- Image identity
- Image digest
- Cosign signing
- SBOM generation
- Attestation verification
- Security evidence

The reports and raw evidence are then included in the automated release notification.

<div align="center">

<img src="images/jenkins/JENKINS 10/jenkins37.png" width="250"/>
<img src="images/jenkins/JENKINS 10/jenkins38.png" width="250"/>
<img src="images/jenkins/JENKINS 10/jenkins39.png" width="250"/>
<img src="images/jenkins/JENKINS 10/jenkins40.png" width="250"/>
<img src="images/jenkins/JENKINS 10/jenkins41.png" width="250"/>

</div>

<div align="center">

<img src="images/jenkins/JENKINS 11/jenkins42.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins43.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins44.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins45.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins46.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins47.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins48.png" width="250"/>
<img src="images/jenkins/JENKINS 11/jenkins49.png" width="250"/>

</div>

---

# 🔄 End-to-End Jenkins DevSecOps Pipeline

```text
Developer Changes
       │
       ▼
   GitHub pre-main
       │
       ▼
    premain.sh
       │
       ▼
     Jenkins
       │
       ├── CI
       │    ├── Tests
       │    ├── Lint
       │    └── Build
       │
       ├── OWASP
       │    └── Dependency Check
       │
       ├── QA
       │    ├── Cypress
       │    ├── k6
       │    ├── PostgreSQL
       │    └── WireMock
       │
       ├── SonarCloud
       │
       ├── Trivy
       │    ├── Filesystem
       │    └── Image
       │
       └── Gitleaks
              │
              ▼
       Validation Evidence
              │
              ▼
          Validation
           Complete
              │
              ▼
             main
              │
              ▼
      Jenkins Production Job
              │
              ▼
        Docker Buildx
              │
       ┌──────┴──────┐
       ▼             ▼
    amd64           arm64
       │             │
       └──────┬──────┘
              │
              ▼
       Multi-Arch Image
              │
       ┌──────┴──────┐
       ▼             ▼
      GHCR       Docker Hub
       │             │
       └──────┬──────┘
              │
              ▼
       Immutable Digest
              │
              ▼
        Cosign Signing
              │
              ▼
         Syft / SPDX
              │
              ▼
     Attestation Verification
              │
              ▼
    Machine-Readable Evidence
              │
              ▼
       HTML Security Reports
              │
              ▼
        Email Notification
```

---

# 🔐 DevSecOps Security Controls

| Security Layer | Technology | Purpose |
|---|---|---|
| Source Control | GitHub | Version-controlled source |
| CI/CD Orchestration | Jenkins | Pipeline automation |
| Cloud Execution | GitHub Actions Cloud | Ephemeral pipeline agents |
| Secure Ingress | zrok | HTTPS access to local Jenkins |
| Unit Testing | Jest | Automated application testing |
| E2E Testing | Cypress | Browser-based validation |
| Performance Testing | k6 | Smoke and load testing |
| API Isolation | WireMock | External service mocking |
| Dependency Security | OWASP Dependency-Check | Dependency vulnerability analysis |
| Static Analysis | SonarCloud | Code quality and security analysis |
| Secret Detection | Gitleaks | Hardcoded secret detection |
| Filesystem Scanning | Trivy | Source/dependency scanning |
| Container Scanning | Trivy | Container vulnerability scanning |
| Container Build | Docker Buildx | Multi-platform images |
| Registry | GHCR | Container distribution |
| Registry | Docker Hub | Container distribution |
| Image Signing | Cosign | Cryptographic artifact signing |
| SBOM | Syft / SPDX | Software component inventory |
| Attestation | Verification Evidence | Supply-chain metadata verification |
| Reporting | HTML / JSON | Human and machine-readable evidence |
| Notification | Email | Automated report distribution |

---

# 🧰 Jenkins DevSecOps Toolchain

| Category | Technology |
|---|---|
| CI/CD | Jenkins |
| SCM | Git / GitHub |
| Cloud Agents | GitHub Actions |
| Secure Ingress | zrok |
| Application Runtime | Node.js |
| Package Manager | npm |
| Unit Testing | Jest |
| E2E Testing | Cypress |
| Performance Testing | k6 |
| Database | PostgreSQL |
| API Mocking | WireMock |
| Static Analysis | SonarCloud |
| Dependency Security | OWASP Dependency-Check |
| Filesystem Security | Trivy |
| Container Security | Trivy |
| Secret Detection | Gitleaks |
| Container Build | Docker Buildx |
| Image Signing | Cosign |
| SBOM | Syft / SPDX |
| Container Registry | GHCR |
| Container Registry | Docker Hub |
| Reporting | HTML / JSON |
| Notifications | SMTP / Email |
| Automation | Bash |

---

# 📦 Security & Reporting Evidence

## Pre-main Reports

```text
test-summary-report
sonar-summary-report
trivy-fs-report
trivy-img-report
security-report
dependency-check-report
```

## Production Reports

```text
docker-build-push-report.html
docker-security-report.html
```

## Supply-Chain Evidence

```text
cosign-verify-ghcr.json
cosign-verify-dockerhub.json

attestation-verify-ghcr.json
attestation-verify-dockerhub.json

sbom-ghcr.json
sbom-dockerhub.json
```

---

# 🔑 Jenkins Credential Model

The pipeline keeps authentication material outside the Jenkinsfile and accesses it through Jenkins Credentials.

The documented credential categories include:

```text
GitHub
├── GITHUB_CRED
└── github-agent-token

Docker
├── DOCKERHUB_USERNAME
└── DOCKERHUB_TOKEN

Supply Chain
├── COSIGN_PRIVATE_KEY
└── COSIGN_PASSPHRASE

Email
├── EMAIL_USER
├── EMAIL_PASS
├── QA_EMAIL_TO
└── QA_EMAIL_CC
```

Secret values are not intended to be committed to source control.

---

# 📊 Reporting Architecture

The reporting model separates pipeline execution from evidence preservation.

```text
Pipeline Execution
       │
       ├── CI
       ├── QA
       ├── OWASP
       ├── SonarCloud
       ├── Trivy
       ├── Gitleaks
       ├── Docker
       ├── Cosign
       ├── SBOM
       └── Attestation
              │
              ▼
       Evidence Collection
              │
       ┌──────┴──────┐
       ▼             ▼
     HTML           JSON
    Reports        Evidence
       │             │
       └──────┬──────┘
              ▼
       Jenkins Artifacts
              │
              ▼
       Email Notification
```

---

# 🛡️ Supply-Chain Security Model

The final production artifact follows this security chain:

```text
Source Code
     │
     ▼
GitHub Repository
     │
     ▼
Jenkins Validation
     │
     ├── CI
     ├── QA
     ├── OWASP
     ├── SonarCloud
     ├── Trivy
     └── Gitleaks
     │
     ▼
Production Build
     │
     ▼
Multi-Architecture Image
     │
     ▼
GHCR + Docker Hub
     │
     ▼
Immutable Digest
     │
     ▼
Cosign Signature
     │
     ▼
SPDX SBOM
     │
     ▼
Attestation Verification
     │
     ▼
Machine-Readable Evidence
     │
     ▼
HTML Security Report
     │
     ▼
Automated Email
```

---

# 📁 Project Structure

```text
resume-matcher-devops/
│
├── Jenkinsfile
├── jenkins.sh
├── premain.sh
├── main.sh
│
├── .github/
│   └── workflows/
│       ├── jenkins-main.yaml
│       ├── jenkins-premain.yaml
│       └── jenkins-agent.yml
│
├── scripts/
│   ├── ci-test.sh
│   ├── ci-sonarcloud.sh
│   ├── ci-trivy.sh
│   ├── ci-owasp.sh
│   ├── ci-email.sh
│   └── ...
│
├── cypress/
│   └── e2e/
│
├── tests/
│   └── load.js
│
├── reports/
│   ├── ci/
│   ├── security/
│   ├── qa/
│   └── docker/
│
└── application source
```

---

# 🚀 Execution Summary

## Pre-main Validation

```bash
./premain.sh
```

The validation path is:

```text
Local Changes
     │
     ▼
premain.sh
     │
     ▼
pre-main
     │
     ▼
GitHub
     │
     ▼
Jenkins
     │
     ├── CI
     ├── OWASP
     ├── QA
     ├── SonarCloud
     ├── Trivy
     └── Gitleaks
     │
     ▼
Security / QA Reports
     │
     ▼
Email
```

## Production Release

The main branch executes the production release workflow:

```text
main
 │
 ▼
Jenkins
 │
 ├── Checkout
 ├── Build
 ├── Docker Buildx
 ├── linux/amd64
 ├── linux/arm64
 ├── GHCR
 ├── Docker Hub
 ├── Digest
 ├── Cosign
 ├── SPDX SBOM
 ├── Attestation
 ├── Security Evidence
 └── Email
```

---

# 🎯 Outcome

The completed project demonstrates a Jenkins-centered DevSecOps lifecycle in which application validation, security analysis, QA, containerization, artifact integrity, software transparency, supply-chain verification, and reporting are integrated into a single automated workflow.

The final architecture provides:

```text
                 ┌──────────────────────────┐
                 │      Source Control       │
                 │          GitHub           │
                 └────────────┬─────────────┘
                              │
                              ▼
                 ┌──────────────────────────┐
                 │         Jenkins          │
                 │     CI/CD Orchestrator   │
                 └────────────┬─────────────┘
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
            CI             Security            QA
             │                │                │
             │         ┌──────┼──────┐         │
             │         │      │      │         │
             │       OWASP Sonar  Trivy      │
             │                       │         │
             │                    Gitleaks     │
             └────────────────┬───────────────┘
                              │
                              ▼
                     Validation Complete
                              │
                              ▼
                         Production
                              │
                              ▼
                       Docker Buildx
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
                  GHCR             Docker Hub
                    │                   │
                    └─────────┬─────────┘
                              │
                              ▼
                       Image Digest
                              │
                              ▼
                         Cosign Sign
                              │
                              ▼
                          SPDX SBOM
                              │
                              ▼
                    Attestation Verify
                              │
                              ▼
                       Evidence Store
                              │
                              ▼
                      HTML / JSON Reports
                              │
                              ▼
                       Email Notification
```

The result is an integrated **Jenkins DevSecOps CI/CD platform** covering the software lifecycle from source-code checkout and automated validation through production container publication and supply-chain evidence generation.

---

# 👤 Author

**Kaif Mohammed Khan**

DevOps / DevSecOps project demonstrating Jenkins CI/CD orchestration, GitHub SCM integration, ephemeral cloud-agent execution, secure ingress, automated QA, security scanning, multi-architecture container publishing, Cosign signing, SBOM generation, attestation verification and automated security reporting.

---

<div align="center">

**© 2026 Kaif. All rights reserved.**

</div>
