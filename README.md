# Furabapps - Microservices Platform

Furabapps is a high-performance, modular microservices ecosystem designed for ride-hailing, food-delivery, payment orchestration, and supporting utility services. The platform is written in Go and uses a modern cloud-native stack including PostgreSQL, Redis, Kafka, RabbitMQ, and Kubernetes.

---

## 🚀 CI/CD Pipeline (Jenkins)

This repository includes a multi-stage Jenkins pipeline defined in the [Jenkinsfile](file:///d:/KULIAH/SEMESTER%204/CLOUD/furabapps-main/Jenkinsfile) that automates:
1. **Checkout**: Pulling the latest code from GitHub.
2. **Unit Tests**: Running unit tests across all microservices.
3. **Lint/Vet**: Running static analysis via `go vet`.
4. **Build Image**: Local building of Docker images.
5. **Functional Tests**: Spinning up Docker infrastructure (PostgreSQL, Redis, RabbitMQ, Kafka) sequentially and executing functional tests (using `//go:build functional`).
6. **Push Image**: Pushing versioned Docker images to the Docker registry (only on the `main` branch).
7. **Deploy to Kubernetes**: Automatic deployment to K8s using Helm charts.
8. **Verify**: Verifying rollout statuses of all deployments in the namespace.

---

## 🛠️ Troubleshooting: Git Fetch Error in Jenkins

### ❌ Problem
During a Jenkins build execution, you may see the following failure in the logs:
```
hudson.plugins.git.GitException: Command "git.exe fetch --tags --force --progress --prune -- origin +refs/heads/master:refs/remotes/origin/master" returned status code 128:
stdout: 
stderr: fatal: couldn't find remote ref refs/heads/master
Finished: FAILURE
```

### 🔍 Cause
This error occurs because **Jenkins is configured to pull the outdated `master` branch**, but this repository uses **`main` as its primary/default branch** (which is standard practice in modern Git repositories). Since the remote repository `https://github.com/nissfin4/Furabapps-baru.git` does not contain a `master` branch, Git throws a `fatal: couldn't find remote ref refs/heads/master` error and terminates the build before reading the `Jenkinsfile`.

### ✅ How to Fix (Step-by-Step in Jenkins UI)

To resolve this issue, you must update the branch specifier inside your Jenkins Job configuration:

#### A. For a Standard Pipeline Job:
1. Open your **Jenkins Dashboard**.
2. Click on your job/pipeline name (e.g., `Furabapps-baru`).
3. Click **Configure** on the left-side menu.
4. Scroll down to the **Pipeline** section.
5. In the **Definition** dropdown, select **Pipeline script from SCM** (if not already selected).
6. Under the **Git** repository settings, locate the **Branches to build** field.
7. Change the **Branch Specifier (blank for 'any')** from `*/master` to:
   ```
   */main
   ```
8. Click **Save** at the bottom.
9. Click **Build Now** to trigger the build. Jenkins will now successfully fetch and build using the `main` branch!

#### B. For a Multibranch Pipeline Job:
1. Open your **Jenkins Dashboard** and select your Multibranch Pipeline.
2. Click **Configure** on the left menu.
3. Under **Branch Sources** -> **Git**, look for **Discover branches** or **Filter by name**.
4. If you have specified manual branch inclusions under a filter, make sure `main` is listed and `master` is removed.
5. Save and click **Scan Multibranch Pipeline Now**. Jenkins will discover the `main` branch and trigger the build.

---

## 💻 Local Development & Testing

### Prerequisites
* **Docker & Docker Desktop** installed and running.
* **Go 1.22+** installed locally.

### 1. Spin up Infrastructure
Run the following command to start PostgreSQL, Redis, RabbitMQ, and Kafka sequentially:
```powershell
docker compose -f furab-backend/deploy/docker/docker-compose.yml up -d postgres redis rabbitmq kafka
```

### 2. Run Functional Tests
Ensure `GOWORK` is bypassed if you are running tests on individual services:
```powershell
# Bypass workspace file
$env:GOWORK="off"

# Navigate to service directory and run functional tests
cd furab-backend/services/location-service
go test ./test/functional/... -v -tags=functional
```