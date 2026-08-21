# Cortex CLI GitHub Actions Container Image Scan Demo

This repository demonstrates how to integrate Cortex CLI into a GitHub Actions CI/CD pipeline to scan a Docker container image before pushing it to Docker Hub.

## Pipeline

```text
Git Push
   |
   v
Build Docker Image
   |
   v
Install Cortex CLI
   |
   v
Scan Local Image with Cortex CLI
   |
   +--------------------+
   |                    |
 PASS                 FAIL
   |                    |
   v                    v
Push to Docker Hub   Pipeline Stops
```

The Docker image is pushed only when the Cortex CLI scan completes successfully.

## Repository Structure

```text
.
├── app.py
├── requirements.txt
├── Dockerfile
├── .dockerignore
├── README.md
└── .github/
    └── workflows/
        └── build-scan-push.yml
```

## Application

The sample Flask application exposes:

- `/` - sample application endpoint
- `/health` - health endpoint

## Build and Run Locally

Replace `<dockerhub-username>` with your Docker Hub username.

```bash
docker build -t <dockerhub-username>/cortex-cli-demo:local .
docker run -p 8080:8080 <dockerhub-username>/cortex-cli-demo:local
```

Test it:

```bash
curl http://localhost:8080/
curl http://localhost:8080/health
```

## Required GitHub Configuration

Go to:

```text
Repository Settings
→ Secrets and variables
→ Actions
```

### Repository Variable

Create:

```text
DOCKERHUB_USERNAME
```

Example:

```text
mydockerusername
```

### Repository Secrets

Create these secrets:

```text
CORTEX_URL
CORTEX_API_ID
CORTEX_API_KEY
DOCKERHUB_TOKEN
```

Do not commit credentials to the repository.

## Workflow Behavior

Each push to the `main` branch:

1. Builds the Docker image.
2. Installs Cortex CLI dependencies.
3. Downloads Cortex CLI using the Cortex API.
4. Scans the locally built Docker image.
5. Stops immediately if the scan command returns a non-zero exit code.
6. Logs in to Docker Hub only after a successful scan.
7. Pushes the image using the Git commit SHA.
8. Tags and pushes the same image as `latest`.

Example image names:

```text
mydockerusername/cortex-cli-demo:<git-sha>
mydockerusername/cortex-cli-demo:latest
```

## Manual Workflow Run

You can also run the workflow manually:

```text
GitHub Repository
→ Actions
→ Build Scan and Push Container Image
→ Run workflow
```

## Cortex CLI Scan Command

The workflow uses:

```bash
sudo cortexcli   --log-level debug   --api-base-url "$CORTEX_URL"   --api-key "$API_KEY"   --api-key-id "$API_ID"   image scan   --timeout 600   "$IMAGE_NAME"
```

The image scanned is the same local Docker image that is pushed to Docker Hub after a successful scan.
