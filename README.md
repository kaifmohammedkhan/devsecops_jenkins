# DevSecOps CI/CD Pipeline Enhanced

This project demonstrates an enterprise-grade **DevSecOps CI/CD pipeline** built for the **Resume Matcher** platform, integrating security, quality, testing, supply-chain protection, automated reporting, and multi-architecture container distribution from development through production release.

The enhanced implementation extends the CI/CD lifecycle with **cryptographic commit signing and verification, secret detection with Gitleaks, static analysis with SonarCloud, vulnerability scanning with OWASP Dependency-Check and Trivy, automated unit and E2E testing, performance validation with k6, cryptographic container image signing with Cosign, SBOM generation with Syft, attestation verification, and automated HTML reporting**.

The final release workflow produces verifiable container artifacts for both **Docker Hub** and **GitHub Container Registry (GHCR)**.

---

## Access the Walkthrough

[![Resume Matcher DevSecOps Pipeline](https://img.youtube.com/vi/ApellkNGW-I/0.jpg)](https://www.youtube.com/embed/ApellkNGW-I?si=WgOHZJnp1O2Atu8q)

[Watch the Resume Matcher DevSecOps Pipeline Walkthrough](https://www.youtube.com/embed/ApellkNGW-I?si=WgOHZJnp1O2Atu8q)

---

## 🛠 Enhanced DevSecOps Strategy

<div align="center">
<img src="images/cicdenhanced/cicdenhanced.png" width="1000"/>
</div>


The enhanced pipeline is organized into three major implementation stages:

1. **Commit Signing Setup**
2. **Security Scanning, Secret Detection & Commit Verification**
3. **Image Build, Push, Signing & SBOM Generation**

These stages extend the existing CI/CD lifecycle by adding verifiable software supply-chain security and machine-readable security evidence to the production release process.

<div align="center">
<img src="images/cicdenhanced/CICD1/cicd1.1.png" width="1000"/>
</div>

---

# Step 1: Commit Signing Setup

The pipeline infrastructure is organized around dedicated GitHub branches, with **`pre-main`** used for security and validation workflows and **`main`** representing the production release path.

Automated notification secrets are configured using a dedicated **Google App Password**, allowing GitHub Actions to securely dispatch security and QA reports directly to Gmail.

Repository notification secrets include:

```text
# GitHub Repository Secrets Configuration
EMAIL_USER=your-gmail-address
EMAIL_PASS=your-google-app-password
```

In addition to automated reporting, this stage introduces **cryptographic Git commit signing using GPG**.

Signed commits provide a mechanism for verifying that commits originated from the expected signing identity and have not been altered after signing.

### Generate a GPG Key

```bash
gpg --full-generate-key
```

### List Secret Keys

```bash
gpg --list-secret-keys --keyid-format LONG
```

### Configure Git Commit Signing

```bash
git config --global user.signingkey <YOUR_KEY_ID>
git config --global commit.gpgsign true
```

### Export the Public Key

```bash
gpg --armor --export <YOUR_KEY_ID>
```

The exported public key is added to GitHub under:

```text
Settings → SSH and GPG keys
```

Once configured, commits pushed to the repository can display GitHub's **Verified** badge, providing an additional layer of authenticity for the source-code supply chain.

<div align="center">

<img src="images/cicdenhanced/CICD1/cicd1.1.png" width="250"/>
<img src="images/cicdenhanced/CICD1/cicd1.2.png" width="250"/>
<img src="images/cicdenhanced/CICD1/cicd1.3.png" width="250"/>
<img src="images/devsecopscicd/CICD1/gpg-generate.png" width="250"/>
<img src="images/devsecopscicd/CICD1/gpg-config.png" width="250"/>
<img src="images/devsecopscicd/CICD1/github-gpg.png" width="250"/>

</div>

---

# Step 2: Security Scanning, Secret Detection & Commit Verification

The enhanced security layer introduces a dedicated child workflow:

```text
ci-security-checks.yaml
```

This workflow is integrated into:

```text
ci-security-pipeline.yaml
```

The workflow adds two important supply-chain controls:

- **Gitleaks** for hardcoded-secret detection
- **GPG commit signature verification** for commit authenticity

These controls execute alongside the existing security and quality validation process.

### Security Workflow

The security lifecycle combines:

- Gitleaks secret detection
- GPG commit verification
- SonarCloud static analysis
- Trivy security scanning
- Automated HTML report generation
- Email notification

The goal is to ensure that source-code authenticity and secret hygiene are continuously validated alongside traditional vulnerability and code-quality analysis.

The local:

```text
premain.sh
```

automation triggers the validation workflow and prompts the configured GPG signing process during commits.

After execution, the workflow provides confirmation of:

- Commit authenticity
- Secret detection status
- Security scan results
- Automated HTML reporting

The resulting security reports are delivered to Gmail alongside the existing SonarCloud and Trivy results.

<div align="center">

<img src="images/cicdenhanced/CICD2/cicd2.1.png" width="250"/>
<img src="images/cicdenhanced/CICD2/cicd2.2.png" width="250"/>
<img src="images/cicdenhanced/CICD2/cicd2.3.png" width="250"/>
<img src="images/cicdenhanced/CICD2/cicd2.4.png" width="250"/>
<img src="images/cicdenhanced/CICD2/cicd2.5.png" width="250"/>
<img src="images/cicdenhanced/CICD2/cicd2.6.png" width="250"/>

</div>

---

# Step 3: Image Build, Push, Signing & SBOM Generation

The production pipeline on the **`main`** branch extends the existing image build and distribution process with additional software supply-chain security controls.

After the vulnerability-free `pre-main` branch is merged into `main`, the:

```text
docker-publish.yaml
```

workflow builds and publishes the production container images.

The enhanced workflow performs:

- Multi-architecture container builds
- Docker Hub publication
- GHCR publication
- Immutable image-digest identification
- Cosign image signing
- Syft SBOM generation
- Attestation verification
- Machine-readable evidence preservation
- Automated HTML reporting

### Multi-Architecture Build

Production images are built for:

```text
linux/amd64
linux/arm64
```

This allows the same production release to support multiple CPU architectures.

### Container Image Signing

After the images are published, **Cosign** cryptographically signs the container images.

The signing process anchors the image identity to its **immutable digest**, providing stronger supply-chain verification than relying only on mutable tags such as:

```text
latest
```

This allows the published artifact to be independently associated with the exact image digest that was built and released.

### SBOM Generation

**Syft** generates Software Bill of Materials (SBOM) documents for the published images.

The SBOMs provide machine-readable information about the software components contained within the production images.

The generated SBOMs use the **SPDX** format.

### Attestation Verification

The pipeline also preserves and verifies attestation records associated with the released images.

Machine-readable evidence is retained as GitHub Actions artifacts, allowing the generated security evidence to be independently inspected.

The preserved evidence includes:

```text
Cosign JSON
SBOM JSON
Attestation JSON
```

### Automated Release Reporting

The production workflow generates two HTML reports:

```text
Docker Build & Push Report
Docker Security Report
```

The reports provide visibility into:

- Multi-architecture image builds
- Docker Hub publication
- GHCR publication
- Image signing
- SBOM generation
- Attestation verification
- Release metadata

The reports are automatically delivered through email using the configured notification system.

<div align="center">

<img src="images/cicdenhanced/CICD3/cicd3.1.png" width="250"/>
<img src="images/cicdenhanced/CICD3/cicd3.2.png" width="250"/>
<img src="images/cicdenhanced/CICD3/cicd3.3.png" width="250"/>
<img src="images/cicdenhanced/CICD3/cicd3.4.png" width="250"/>
<img src="images/cicdenhanced/CICD3/cicd3.5.png" width="250"/>
<img src="images/cicdenhanced/CICD3/cicd3.6.png" width="250"/>
<img src="images/devsecopscicd/CICD3/main-sh.png" width="250"/>
<img src="images/devsecopscicd/CICD3/actions-success.png" width="250"/>
<img src="images/devsecopscicd/CICD3/commit-verified.png" width="250"/>
<img src="images/devsecopscicd/CICD3/email-report.png" width="250"/>
<img src="images/devsecopscicd/CICD3/docker-build-report.png" width="250"/>
<img src="images/devsecopscicd/CICD3/docker-security-report.png" width="250"/>

</div>

---

# 🔄 End-to-End Enhanced Pipeline Flow

```text
Developer Changes
       │
       ▼
   pre-main
       │
       ├── GPG Commit Signing
       │
       ├── Commit Signature Verification
       │
       ├── Gitleaks Secret Detection
       │
       ├── SonarCloud Analysis
       │
       ├── Trivy Security Scanning
       │
       ├── OWASP Dependency Scanning
       │
       ├── Jest Unit Testing
       │
       ├── Cypress E2E Testing
       │
       ├── k6 Smoke Testing
       │
       └── k6 Load Testing
              │
              ▼
       Security / Quality
             Gates
              │
              ▼
        Validation Passed
              │
              ▼
            main
              │
              ▼
     Multi-Architecture Build
        ┌───────────────┐
        │               │
        ▼               ▼
      GHCR          Docker Hub
        │               │
        └───────┬───────┘
                │
                ▼
       Immutable Digest
                │
                ▼
         Cosign Signing
                │
                ▼
          Syft SBOM
          Generation
                │
                ▼
       Attestation Verify
                │
                ▼
      Machine-Readable Evidence
                │
                ▼
       HTML Release Reports
                │
                ▼
        Email Notification
```

---

# 🔐 DevSecOps Security Controls

The enhanced pipeline introduces multiple layers of protection across the software supply chain.

| Security Layer | Technology | Purpose |
|---|---|---|
| Commit Authenticity | GPG | Cryptographically sign and verify Git commits |
| Secret Detection | Gitleaks | Detect hardcoded credentials and secrets |
| Static Analysis | SonarCloud | Identify code-quality and security issues |
| Dependency Security | OWASP Dependency-Check | Identify vulnerable third-party dependencies |
| Filesystem Security | Trivy | Scan source files and dependencies |
| Container Security | Trivy | Scan container images for vulnerabilities and misconfigurations |
| Image Signing | Cosign | Cryptographically sign production container images |
| SBOM | Syft | Generate software component inventories |
| Attestation | GitHub Actions / Cosign | Verify and preserve build provenance evidence |
| Registry Security | GHCR / Docker Hub | Distribute production container artifacts |
| Reporting | Nodemailer / Gmail | Deliver automated security and release reports |

---

# 🧰 DevSecOps Toolchain

| Category | Technology |
|---|---|
| Source Control | Git / GitHub |
| Branch Management | `pre-main` / `main` |
| Commit Signing | GPG |
| Secret Detection | Gitleaks |
| CI/CD | GitHub Actions |
| Static Analysis | SonarCloud |
| Dependency Security | OWASP Dependency-Check |
| Filesystem Security | Trivy |
| Container Security | Trivy |
| Unit Testing | Jest |
| E2E Testing | Cypress |
| Performance Testing | k6 |
| API Mocking | WireMock |
| Container Build | Docker Buildx |
| Image Signing | Cosign |
| SBOM Generation | Syft |
| Container Registry | GitHub Container Registry |
| Container Registry | Docker Hub |
| Release Reporting | Nodemailer / Gmail |
| Automation | Bash |

---

# 📦 Supply Chain Security

The enhanced pipeline strengthens the software supply chain across multiple stages.

### Source Integrity

GPG signing establishes cryptographic identity for Git commits and allows GitHub to display verified commits.

### Secret Hygiene

Gitleaks continuously checks the repository for accidentally committed credentials, tokens, and other sensitive values.

### Dependency & Container Security

SonarCloud, OWASP Dependency-Check, and Trivy provide multiple layers of vulnerability and quality analysis before production release.

### Artifact Integrity

Production images are identified using their immutable digests and cryptographically signed using Cosign.

### Software Transparency

Syft generates SPDX SBOMs describing the software components included in the production container images.

### Verifiable Evidence

Cosign verification, SBOM documents, and attestation records are preserved as machine-readable artifacts.

### Human-Readable Reporting

HTML security and release reports are generated and delivered through email, providing an accessible summary of the production release.

---

# 📊 Automated Evidence & Reporting

The pipeline produces both human-readable and machine-readable security evidence.

### HTML Reports

```text
Docker Build & Push Report
Docker Security Report
```

These reports summarize the production build, registry publication, signing, SBOM generation, and attestation verification.

### Machine-Readable Evidence

The pipeline preserves:

```text
Cosign JSON
SBOM JSON
Attestation JSON
```

These artifacts can be independently inspected rather than relying exclusively on the rendered HTML reports.

This provides a stronger evidence model for the production release because the underlying security metadata remains available for verification.

---

# 🚀 Production Release Flow

The production release is performed through the `main` branch after validation of the `pre-main` branch.

The release process follows the general flow:

```bash
git checkout main
git merge pre-main
git push origin main
```

The production workflow then performs:

```text
1. Build production images
2. Build linux/amd64 image
3. Build linux/arm64 image
4. Push images to GHCR
5. Push images to Docker Hub
6. Resolve immutable image digests
7. Sign images with Cosign
8. Generate SPDX SBOMs with Syft
9. Verify attestations
10. Preserve machine-readable evidence
11. Generate HTML reports
12. Send release reports through email
```

---

# 📝 Notes

- **GPG** provides cryptographic commit signing and source authenticity verification.
- **Gitleaks** detects hardcoded secrets before they enter the production supply chain.
- **SonarCloud** performs static analysis and code-quality/security analysis.
- **OWASP Dependency-Check** identifies vulnerable third-party dependencies.
- **Trivy** scans filesystems and container images for vulnerabilities and misconfigurations.
- **Jest** provides automated unit testing and coverage.
- **Cypress** validates end-to-end application behavior.
- **WireMock** isolates external API dependencies during QA.
- **k6** validates smoke-test reliability and sustained load performance.
- **Docker Buildx** enables multi-architecture container builds.
- **Cosign** provides cryptographic container image signing.
- **Syft** generates SPDX Software Bill of Materials.
- **Attestation verification** provides additional supply-chain evidence.
- **GHCR and Docker Hub** distribute the production container images.
- **Nodemailer/Gmail** delivers automated HTML security and release reports.
- Machine-readable security evidence is preserved as GitHub Actions artifacts.
- Production image identity is anchored to immutable image digests rather than relying solely on mutable tags.

---

# 🎯 Outcome

The enhanced Resume Matcher DevSecOps pipeline extends traditional CI/CD into a more verifiable software supply-chain workflow.

Instead of stopping at source-code testing and vulnerability scanning, the pipeline establishes controls across the complete delivery chain:

```text
Source
  ↓
Signed Commits
  ↓
Secret Detection
  ↓
Static Analysis
  ↓
Dependency Scanning
  ↓
Filesystem / Container Scanning
  ↓
Automated Testing
  ↓
Quality & Security Gates
  ↓
Production Build
  ↓
Multi-Architecture Images
  ↓
Immutable Image Digest
  ↓
Cosign Image Signing
  ↓
SBOM Generation
  ↓
Attestation Verification
  ↓
Machine-Readable Evidence
  ↓
HTML Release Reporting
  ↓
GHCR + Docker Hub
```

The result is a **security-focused, test-driven, auditable, and verifiable DevSecOps release process** in which source authenticity, secret hygiene, application quality, dependency security, container security, artifact integrity, software transparency, and release evidence are integrated into a single automated delivery lifecycle.

---

<div align="center">

**© 2026 Kaif. All rights reserved.**

</div>

_____________

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Jenkins CI/CD Pipeline Project</title>
  <link rel="stylesheet" href="/style.css" />
</head>
<body>
  <aside class="sidebar">
    <ul>
      <li><a href="/#about">About</a></li>
      <li><a href="/#projects">Projects</a></li>
      <li><a href="/#skills">Skills</a></li>
      <li><a href="/#contact">Contact</a></li>
    </ul>
  </aside>

<section class="hero">
  <h1>Jenkins DevSecOps CI/CD Pipeline Walkthrough</h1>
  <p>
    A comprehensive guide to orchestrating automated end-to-end continuous integration and deployment 
    using Jenkins. From agent node setup and SCM triggers to static analysis, secret scanning, containerization, 
    image signing, Kubernetes deployment, and live application validation.
  </p>
</section>

<section class="section">
  <h2>Access the walkthrough</h2>
  <div class="video-container">
    <iframe width="560" height="315"
      src="https://www.youtube.com/embed/qBbtWlOH5rg?si=y8lQeAAUDfd1_1QU"
      title="Jenkins CI/CD Pipeline Tutorial for Beginners | Node.js, Docker & GitHub Actions Cloud Agents" frameborder="0"
      allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
      referrerpolicy="strict-origin-when-cross-origin" allowfullscreen>
    </iframe>
  </div>
</section>
<section class="section">
  <h2>Deployment Strategy</h2>
<!-- JENKINS 1: Pipeline Architecture & Jenkins Initialization -->
  <div class="card">
    <h3>Pipeline Architecture & Jenkins Controller Setup</h3>
    <p>
      The DevSecOps pipeline architecture was designed to handle pre-main validation, multi-arch builds, supply chain signing, SBOM generation, and automated reporting.
      The environment was initialized by deploying the official Jenkins LTS Docker container on port 8080 with persistent home volume storage:
    </p>
    <pre><code># Pull official Jenkins LTS image and run container
docker pull jenkins/jenkins:lts

docker run -d \
  --name jenkins \
  -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Retrieve initial administrator password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
</code></pre>
    <p>
      After retrieving the initial admin secret via Docker execution, the web interface on <code>http://localhost:8080</code> was unlocked. 
      Suggested plugins were installed (including Git, Pipeline, SSH Build Agents, and Mailer), followed by configuring the primary admin user profile (<code>kaifmohammedkhan</code>).
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 1/jenkins1.1.png" alt="DevSecOps Architecture Diagram" />
      <img src="/images/jenkins/JENKINS 1/jenkins1.png" alt="Docker Pull & Container Run" />
      <img src="/images/jenkins/JENKINS 1/jenkins2.png" alt="Unlock Jenkins Getting Started Page" />
      <img src="/images/jenkins/JENKINS 1/jenkins3.png" alt="Retrieve Initial Admin Password in Terminal" />
      <img src="/images/jenkins/JENKINS 1/jenkins4.png" alt="Customize Jenkins Plugin Selection" />
      <img src="/images/jenkins/JENKINS 1/jenkins5.png" alt="Installing Suggested Plugins Progress" />
      <img src="/images/jenkins/JENKINS 1/jenkins6.png" alt="Create First Admin User Setup" />
    </div>
  </div>

<!-- JENKINS 2: Pipeline Initialization & SCM Checkout -->
  <div class="card">
    <h3>Pipeline Job Configuration & Source Control Integration</h3>
    <p>
      A new Pipeline job named <code>resume-matcher-jenkins-premain</code> was created in Jenkins and configured to pull its definition dynamically using <strong>Pipeline script from SCM</strong> pointing to <code>https://github.com/kaifmohammedkhan/resume-matcher-devops</code> on branch <code>*/pre-main</code>.
    </p>
    <p>
      To allow Jenkins to interact securely with GitHub, DockerHub, SonarQube, Cosign, and SMTP services, all required Personal Access Tokens (PATs) and credentials—such as <code>GITHUB_CRED</code>—were created in GitHub and registered securely inside the global Jenkins Credential Store.
    </p>
    <pre><code>pipeline {
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
</code></pre>
    <p>
      The SCM configuration was bound with the newly added credentials (<code>kaifmohammedkhan/******</code>) to ensure secure checkout of the <code>Jenkinsfile</code> from the targeted branch during pipeline runs.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 2/jenkins7.png" alt="Creating new Pipeline job resume-matcher-jenkins-premain" />
      <img src="/images/jenkins/JENKINS 2/jenkins8.png" alt="Configuring Pipeline script from SCM with GitHub repo URL" />
      <img src="/images/jenkins/JENKINS 2/jenkins9.png" alt="Specifying pre-main branch and Jenkinsfile script path" />
      <img src="/images/jenkins/JENKINS 2/jenkins10.png" alt="Generating GITHUB_CRED Personal Access Token on GitHub" />
      <img src="/images/jenkins/JENKINS 2/jenkins11.1.png" alt="Jenkins Global Credentials Store page 1" />
      <img src="/images/jenkins/JENKINS 2/jenkins11.2.png" alt="Jenkins Global Credentials Store page 2 showing GITHUB_CRED" />
      <img src="/images/jenkins/JENKINS 2/jenkins12.png" alt="Attaching kaifmohammedkhan credentials to Git SCM configuration" />
    </div>
  </div>


<!-- JENKINS 3: Jenkins Environment Tooling Setup -->
  <div class="card">
    <h3>Jenkins Controller Tooling & Environment Configuration</h3>
    <p>
      To enable container management, security scanning, and application testing directly inside the Jenkins controller, necessary runtime packages were installed via root shell access inside the running Jenkins container:
    </p>
    <pre><code># Access Jenkins container shell as root
docker exec -it -u root jenkins /bin/bash

# Update package repository and install Docker CLI dependencies
apt-get update
apt-get install -y docker.io

# Verify Docker CLI installation inside container
docker --version

# Install Node.js and NPM runtime environment
apt-get update &amp;&amp; apt-get install -y nodejs npm
</code></pre>
    <p>
      Installing <code>docker.io</code>, <code>nodejs</code>, and <code>npm</code> directly inside the controller container ensured that downstream pipeline stages—including container build workflows, unit tests, secret scanning, and static analysis tools—had all required CLI binaries available natively on the execution host.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 3/jenkins15.png" alt="Exec into Jenkins container as root and install docker.io via apt-get" />
      <img src="/images/jenkins/JENKINS 3/jenkins16.png" alt="Verify Docker CLI version 26.1.5 installation inside container" />
      <img src="/images/jenkins/JENKINS 3/jenkins17.png" alt="Install Node.js and NPM packages inside Jenkins container" />
    </div>
  </div>

<!-- JENKINS 4: GitHub Actions Cloud Agents Integration -->
  <div class="card">
    <h3>GitHub Actions Cloud Agent Provisioning Configuration</h3>
    <p>
      To enable dynamic build execution offloading, Jenkins was configured to provision GitHub Actions Cloud Agents on demand. The <strong>GitHub Actions Cloud Agents</strong> plugin was installed, fine-grained Personal Access Tokens (PAT) were stored in Jenkins Credentials, and cloud agent templates were mapped to the repository target branch.
    </p>
    <pre><code># Jenkins Cloud Configuration Details:
# Cloud Name:       github-cloud
# Provider Type:    GitHub Actions
# Repository:        kaifmohammedkhan/resume-matcher-devops
# Secret ID:         github-agent-token
# Agent Label:       gha-runner
# Remote FS Root:    /home/runner/agent
# Workflow File:     jenkins-agent.yml
# Git Ref:           pre-main
</code></pre>
    <p>
      This configuration allowed Jenkins pipelines to automatically spawn ephemeral, one-shot GitHub Actions runners on demand, providing isolated execution environments for container builds and security stages.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 4/jenkins18.png" alt="Install GitHub Actions Cloud Agents plugin in Jenkins Plugin Manager" />
      <img src="/images/jenkins/JENKINS 4/jenkins19.png" alt="Generate fine-grained GitHub Personal Access Token with Actions, Contents, and Workflows permissions" />
      <img src="/images/jenkins/JENKINS 4/jenkins20.png" alt="Add secret text credential github-agent-token in Jenkins Credentials Manager" />
      <img src="/images/jenkins/JENKINS 4/jenkins21.png" alt="Create new cloud provider instance named github-cloud using GitHub Actions type" />
      <img src="/images/jenkins/JENKINS 4/jenkins22.png" alt="Configure cloud details with repository path and github-agent-token credentials" />
      <img src="/images/jenkins/JENKINS 4/jenkins23.png" alt="Define cloud agent template gha-runner with jenkins-agent.yml workflow file" />
      <img src="/images/jenkins/JENKINS 4/jenkins24.png" alt="Set Git Ref target branch to pre-main for one-shot agent dispatch" />
    </div>
  </div>

<!-- JENKINS 5: Tunneling & Ingress Setup with zrok -->
  <div class="card">
    <h3>Secure Ingress & Tunneling Setup with zrok</h3>
    <p>
      To expose the local Jenkins instance securely to external webhooks and GitHub Actions agent callbacks, 
      <strong>zrok</strong> was installed and initialized on the host machine. An environment token was generated 
      via the zrok portal and activated using the zrok CLI[cite: 25, 27].
    </p>
    <pre><code># Download and extract zrok binary on Windows PowerShell
New-Item -ItemType Directory -Path "C:\zrok" -Force; Invoke-WebRequest -Uri "https://github.com/openziti/zrok/releases/download/v0.4.42/zrok_0.4.42_windows_amd64.tar.gz" -OutFile "C:\zrok\zrok.tar.gz"; tar -xf "C:\zrok\zrok.tar.gz" -C "C:\zrok"; Remove-Item "C:\zrok\zrok.tar.gz"

# Verify zrok CLI installation
C:\zrok\zrok.exe version

# Enable zrok environment with account token
C:\zrok\zrok.exe enable &lt;zrok-token&gt;
</code></pre>
    <p>
      Once enabled, zrok established a persistent secure tunnel, allowing cloud-hosted GitHub Actions workflows 
      to communicate back to the local Jenkins controller seamlessly during pipeline executions[cite: 25, 27].
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 5/jenkins25.1.png" alt="Successfully created zrok account credentials and environment enable command" />
      <img src="/images/jenkins/JENKINS 5/jenkins25.2.png" alt="Download zrok v0.4.42 release binary and verify CLI version via PowerShell" />
      <img src="/images/jenkins/JENKINS 5/jenkins25.3.png" alt="Execute zrok enable command to successfully initialize zrok environment" />
    </div>
  </div>

<!-- JENKINS 6: zrok Reserved Share Automation & Jenkins Location Binding -->
  <div class="card">
    <h3>zrok Reserved Share Automation &amp; Jenkins Location Binding</h3>
    <p>
      To ensure a persistent, static public URL for incoming GitHub webhooks and agent callbacks, a automation script 
      (<code>jenkins.sh</code>) was executed to establish a named zrok share (<code>my-jenkins-local</code>)[cite: 28, 29, 30]. The generated public endpoint 
      (<code>https://my-jenkins-local.shares.zrok.io</code>) was then bound as the primary Jenkins Location URL[cite: 30, 31, 32].
    </p>
    <pre><code># Execute zrok share reset &amp; reservation script
chmod +x jenkins.sh &amp;&amp; ./jenkins.sh

# Script output:
# Starting zrok share reset sequence...
# [INFO] Releasing reserved name...
# [INFO] Creating reserved name 'my-jenkins-local' in namespace 'public'
# [INFO] Starting named share at my-jenkins-local.shares.zrok.io

# Configure Jenkins Location URL via UI System Settings:
# Jenkins URL: https://my-jenkins-local.shares.zrok.io/
</code></pre>
    <p>
      This guaranteed that Jenkins could receive webhook events from GitHub and remain fully accessible over secure SSL HTTPS, even across container restarts or IP address changes[cite: 29, 31, 32].
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 6/jenkins25.4.png" alt="Repository workspace files showing jenkins.sh automation script" />
      <img src="/images/jenkins/JENKINS 6/jenkins25.5.png" alt="Execute jenkins.sh to trigger zrok reserved share creation for my-jenkins-local" />
      <img src="/images/jenkins/JENKINS 6/jenkins25.6.png" alt="zrok TUI proxy monitor active for my-jenkins-local.shares.zrok.io" />
      <img src="/images/jenkins/JENKINS 6/jenkins26.2.png" alt="Set Jenkins URL to https://my-jenkins-local.shares.zrok.io/ in System Configuration" />
      <img src="/images/jenkins/JENKINS 6/jenkins27.jpg" alt="Access Jenkins login portal securely over zrok HTTPS domain" />
    </div>
  </div>

<!-- JENKINS 7: Ephemeral GitHub Actions Agent Provisioning & Orchestration -->
  <div class="card">
    <h3>Ephemeral GitHub Actions Agent Provisioning &amp; Orchestration</h3>
    <p>
      To offload workload execution from the main Jenkins controller, dynamic agent provisioning was configured via 
      the <strong>github-cloud</strong> integration[cite: 33, 35]. Jenkins orchestrates build jobs by dynamically triggering 
      the <code>Jenkins Agent Runner</code> workflow hosted on GitHub Actions[cite: 33, 34].
    </p>
    <pre><code>// Jenkinsfile agent configuration snippet
agent {
    node {
        label 'github-actions-runner'
    }
}

// Workflow invocation process:
// 1. Jenkins triggers 'Jenkins Agent Runner' via GitHub API / Cloud plugin
// 2. Ephemeral GitHub Actions runner spins up and connects back to Jenkins controller
// 3. Pipeline stage execution offloaded to GitHub Actions runner environment
</code></pre>
    <p>
      The status and health of the cloud agent provisioner are tracked directly within Jenkins system statistics, 
      ensuring continuous execution capability across automated pipeline runs.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 7/jenkins30.1.png" alt="Jenkins test-gha-agent job stage view showing successful Verify Provisioning execution" />
      <img src="/images/jenkins/JENKINS 7/jenkins30.2.png" alt="GitHub Actions dashboard listing triggered Jenkins Agent Runner workflows" />
      <img src="/images/jenkins/JENKINS 7/jenkins31.png" alt="Jenkins Cloud statistics dashboard for github-cloud provider health" />
    </div>
  </div>

<!-- JENKINS 8: API Authentication, Secrets Configuration & Parameterized Pipeline Setup -->
  <div class="card">
    <h3>API Authentication, Secrets Configuration &amp; Parameterized Pipeline Setup</h3>
    <p>
      To enable secure bidirectional communication between GitHub Actions and the Jenkins controller, a dedicated API token 
      (<code>GitHub-Actions-PR</code>) was generated under user security settings[cite: 36, 37, 38]. Corresponding secrets—including 
      <code>JENKINS_URL</code>, <code>JENKINS_USER</code>, and <code>JENKINS_TOKEN</code>—were securely configured in the GitHub repository settings[cite: 39, 40, 41].
    </p>
    <pre><code>// GitHub Repository Actions Secrets:
// - JENKINS_URL   : https://my-jenkins-local.shares.zrok.io
// - JENKINS_USER  : kaifmohammedkhan
// - JENKINS_TOKEN : &lt;generated-api-token&gt;

// Jenkins Job Configuration: Parameterized Pipeline
// Parameters created:
// - String Parameter: PR_NUMBER
// - String Parameter: BRANCH_NAME
</code></pre>
    <p>
      Additionally, the Jenkins pipeline (<code>resume-matcher-main</code>) was parameterized to accept build metadata such as <code>PR_NUMBER</code> and <code>BRANCH_NAME</code>[cite: 42, 43]. This allows automated GitHub Actions workflow triggers (defined in <code>.github/workflows</code>) to programmatically invoke Jenkins builds with precise contextual arguments[cite: 39, 42, 44].
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 8/jenkins32.1.png" alt="Jenkins User Security page prior to API token creation" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.2.png" alt="Creating new non-expiring API token named GitHub-Actions-PR in Jenkins" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.3.png" alt="Generated Jenkins API token confirmation dialog" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.4.png" alt="Adding JENKINS_URL secret in GitHub repository Actions settings" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.5.png" alt="Adding JENKINS_USER secret in GitHub repository Actions settings" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.6.png" alt="Adding JENKINS_TOKEN secret in GitHub repository Actions settings" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.7.png" alt="Configuring parameterized build options in Jenkins job setup" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.8.png" alt="Defining PR_NUMBER and BRANCH_NAME String Parameters in job config" />
      <img src="/images/jenkins/JENKINS 8/jenkins32.9.png" alt="GitHub workflow directory listing showing jenkins-main.yaml and jenkins-premain.yaml triggers" />
    </div>
  </div>

<!-- JENKINS 9: Automated Script Trigger, Pipeline Stage Execution & Email Artifact Notifications -->
  <div class="card">
    <h3>Automated Script Trigger, Pipeline Stage Execution &amp; Email Artifact Notifications</h3>
    <p>
      The release process is initiated locally using a helper script (<code>./premain.sh</code>) to automatically reconcile working branch states, perform auto-stashing, merge changes into the target deployment branch, and push commits to GitHub. This remote push triggers the Jenkins pipeline (<code>resume-matcher-jenkins-premain</code>)[cite: 45, 46].
    </p>
    <pre><code># Helper Script Trigger Execution & Git Sync:
$ ./premain.sh
# Checks working directory, auto-stashes changes, switches branch, 
# merges pre-main changes, commits, and pushes to GitHub.

# Triggers Jenkins Pipeline Job: resume-matcher-jenkins-premain
# Executing stages: Parallel Jobs, CI Job, OWASP Job, QA Job, SonarCloud Analysis,
# Trivy FS/Image Scans, Gitleaks, and Send Email Job.
</code></pre>
    <p>
      Upon completion of all pipeline stages—including parallel test suites, OWASP Dependency-Check, SonarCloud analysis, and Trivy filesystem/container scans—the pipeline consolidates generated security and test artifacts and sends an automated email report with HTML attachments to stakeholders[cite: 46, 47, 48, 49, 50].
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 9/jenkins32.10.png" alt="Executing helper script in Git Bash terminal to sync and push changes to GitHub" />
      <img src="/images/jenkins/JENKINS 9/jenkins32.png" alt="Jenkins Pipeline Stage View showing build #58 status and initial stages" />
      <img src="/images/jenkins/JENKINS 9/jenkins33.png" alt="Jenkins Pipeline Stage View middle stages including OWASP Dependency Check and SonarCloud Analysis" />
      <img src="/images/jenkins/JENKINS 9/jenkins34.png" alt="Jenkins Pipeline Stage View final stages including Trivy scans, Gitleaks, and Send Email Job" />
      <img src="/images/jenkins/JENKINS 9/jenkins35.png" alt="Gmail notification for CI Pipeline Reports showing execution status for pre-main branch" />
      <img src="/images/jenkins/JENKINS 9/jenkins36.png" alt="Gmail notification attachments including test-summary, sonar-summary, trivy, security, and dependency-check reports" />
    </div>
  </div>

<!-- JENKINS 10: Production Job SCM Configuration, Cosign CLI Installation & Signing Credentials -->
  <div class="card">
    <h3>Production Job SCM Configuration, Cosign CLI Installation &amp; Signing Credentials</h3>
    <p>
      To support the main production release pipeline (<code>resume-matcher-main</code>), SCM settings were configured to point directly to the <code>*/main</code> branch with SCM-managed <code>Jenkinsfile</code> execution. On the build host, Cosign was installed and configured locally under user binaries to enable digital signing and verification of container images pushed during pipeline execution.
    </p>
    <pre><code># Jenkins Job SCM Configuration:
# - Repository URL : https://github.com/kaifmohammedkhan/resume-matcher-devops
# - Branch Specifier : */main
# - Script Path      : Jenkinsfile

# Cosign Binary Setup (Git Bash / Local Host):
$ curl -sSL "https://github.com/sigstore/cosign/releases/latest/download/cosign-windows-amd64.exe" -o cosign.exe
$ mkdir -p ~/bin && mv cosign.exe ~/bin/cosign
$ cosign version
</code></pre>
    <p>
      Additionally, security credentials required for non-interactive image signing—specifically <code>COSIGN_PASSPHRASE</code> (Secret text) and <code>COSIGN_PRIVATE_KEY</code> (Secret file referencing <code>cosign.key</code>)—were securely added to Jenkins Global Credentials.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 10/jenkins37.png" alt="Configuring Git repository SCM URL and credentials in resume-matcher-main job settings" />
      <img src="/images/jenkins/JENKINS 10/jenkins38.png" alt="Setting branch specifier to */main and Script Path to Jenkinsfile in job configuration" />
      <img src="/images/jenkins/JENKINS 10/jenkins39.png" alt="Downloading and installing Cosign CLI v3.1.3 binary in Git Bash terminal" />
      <img src="/images/jenkins/JENKINS 10/jenkins40.png" alt="Adding COSIGN_PASSPHRASE Secret text credential in Jenkins Manage Credentials" />
      <img src="/images/jenkins/JENKINS 10/jenkins41.png" alt="Uploading cosign.key as COSIGN_PRIVATE_KEY Secret file in Jenkins Manage Credentials" />
    </div>
  </div>
  
<!-- JENKINS 11: End-to-End Build Execution, Stage View & Email Security Reports -->
  <div class="card">
    <h3>End-to-End Build Execution, Stage View &amp; Email Security Reports</h3>
    <p>
      The <code>resume-matcher-main</code> production pipeline run (#40) was executed through to full completion[cite: 56, 57]. The build sequence encompassed multi-architecture Docker image compilation, multi-registry publishing (GHCR &amp; Docker Hub), Cosign container signing, SPDX SBOM generation, and independent attestation verification across both target registries[cite: 56, 60, 61, 62].
    </p>
    <pre><code># Jenkins Pipeline Artifacts & Security Evidence Archival
Artifacts:
  - docker-build-push-report.html
  - docker-security-report.html
  - cosign-verify-ghcr.json / cosign-verify-dockerhub.json
  - attestation-verify-ghcr.json / attestation-verify-dockerhub.json
  - sbom-ghcr.json / sbom-dockerhub.json

# Image Digest Anchored in Pipeline Report:
Repository : kaifmohammedkhan/resume-matcher-devops
Digest     : sha256:661afae7ba52164864492b48b5cfed3eb3890b854c65952b44ed364fae98a72f
</code></pre>
    <p>
      Upon successfully passing all verification checks, Jenkins generated and dispatched an automated HTML email report summarizing the build, image digests, Cosign signatures, and SBOM attestations, attaching raw security evidence files directly for compliance auditability.
    </p>
    <div class="image-gallery">
      <img src="/images/jenkins/JENKINS 11/jenkins42.png" alt="Jenkins job status showing Build #40 artifacts and archived security reports" />
      <img src="/images/jenkins/JENKINS 11/jenkins43.png" alt="Stage View showing initial checkout, setup, and registry authentication stages" />
      <img src="/images/jenkins/JENKINS 11/jenkins44.png" alt="Stage View displaying Docker build, push, and evidence preservation stages" />
      <img src="/images/jenkins/JENKINS 11/jenkins45.png" alt="Stage View rendering Cosign and Syft CLI tool initialization stages" />
      <img src="/images/jenkins/JENKINS 11/jenkins46.png" alt="Stage View covering image verification, Cosign signing, and SPDX SBOM generation" />
      <img src="/images/jenkins/JENKINS 11/jenkins47.png" alt="Stage View showing final SBOM attestation, report packaging, and email notification stages" />
      <img src="/images/jenkins/JENKINS 11/jenkins48.png" alt="Automated Docker Build, Push & Security Report email notification body" />
      <img src="/images/jenkins/JENKINS 11/jenkins49.png" alt="Email report footer displaying canonical image digest and security report attachments" />
    </div>
  </div>
</section>

  <footer class="footer">
    <p>&copy; 2026 Kaif. All rights reserved.</p>
  </footer>
</body>
</html>
