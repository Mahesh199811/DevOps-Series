# 🐳 Phase 2 — Application Setup & Dockerization

> Part of the [DevOps End-to-End Project](../README.md) | Cloud-Native Application on AWS

---

## 📋 Overview

This phase covers building a real .NET Web API, containerizing it with Docker using a multi-stage build, and deploying it to the EC2 instance created in Phase 1. The GitHub repo was cloned directly onto the EC2 Linux instance and the Docker image was built there natively — avoiding Mac ARM64 vs EC2 AMD64 platform issues entirely.

---

## 🏗️ What Was Built

| Component | Details |
|-----------|---------|
| Application | .NET 8 Web API — Task Manager API |
| Containerization | Docker (multi-stage build) |
| Base Image (build) | `mcr.microsoft.com/dotnet/sdk:8.0` |
| Base Image (runtime) | `mcr.microsoft.com/dotnet/aspnet:8.0` |
| Exposed Port | `8080` |
| Deployment Method | Cloned GitHub repo directly on EC2, built image on Linux |
| Deployment Target | EC2 instance (`devops-server`) from Phase 1 |

---

## 📁 Repo Structure

```
app/
└── TaskManagerApi/
    ├── Controllers/
    │   └── WeatherForecastController.cs
    ├── Properties/
    │   └── launchSettings.json
    ├── TaskManagerApi.csproj
    ├── Program.cs
    ├── Dockerfile          ← Multi-stage build
    └── .dockerignore
```

---

## ✅ Day 3 — Build, Dockerize & Deploy

### 🎯 Goals
- Create a .NET Web API project locally and push to GitHub
- Write a production-grade multi-stage Dockerfile
- Clone the GitHub repo directly on EC2
- Build Docker image natively on EC2 (Linux AMD64)
- Run and verify the container is publicly accessible

---

### Step 1 — Create .NET API (Local Mac)

```bash
mkdir devops-project/app
cd devops-project/app

dotnet new webapi -n TaskManagerApi
cd TaskManagerApi

# Test locally before Dockerizing
dotnet run
# Visit: http://localhost:5000/weatherforecast
```

---

### Step 2 — Dockerfile (Multi-Stage Build)

```dockerfile
# Stage 1 — Build
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o /app/publish

# Stage 2 — Runtime only (smaller image)
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENV ASPNETCORE_URLS=http://+:8080
ENTRYPOINT ["dotnet", "TaskManagerApi.dll"]
```

**Why multi-stage?**

| Single Stage | Multi-Stage |
|---|---|
| ~900MB (includes SDK) | ~220MB (runtime only) |
| SDK tools exposed in image | No SDK in final image |
| Slower to push/pull | Faster deployments |
| Security risk | Smaller attack surface |

---

### Step 3 — Push Code to GitHub (from Mac)

Once the API and Dockerfile are ready locally, push everything to GitHub:

```bash
cd devops-project
git add app/
git commit -m "Add .NET TaskManager API with multi-stage Dockerfile"
git push origin main
```

---

### Step 4 — Install Git & Clone Repo on EC2

SSH into your EC2 instance:

```bash
ssh -i ~/.ssh/devops-key.pem ubuntu@<EC2-PUBLIC-IP>
```

Install Git and clone your repo directly on EC2:

```bash
# Install Git
sudo apt update
sudo apt install git -y

# Clone your GitHub repo
git clone https://github.com/<your-username>/devops-project.git

# Navigate to the app folder
cd devops-project/app/TaskManagerApi
```

> ✅ This is cleaner than `scp` — EC2 always pulls the latest code from GitHub. This also sets the foundation for CI/CD in Phase 3.

---

### Step 5 — Install .NET SDK on EC2

```bash
# Add Microsoft package repository
wget https://packages.microsoft.com/config/ubuntu/24.04/packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb

# Install .NET SDK
sudo apt update
sudo apt install dotnet-sdk-8.0 -y

# Verify
dotnet --version
```

---

### Step 6 — Build Docker Image on EC2

```bash
cd ~/devops-project/app/TaskManagerApi

# Build natively on EC2 (AMD64)
docker build -t taskmanager-api .
```

**Verify image was created:**
```bash
docker images
# Should show taskmanager-api with its size
```

---

### Step 7 — Run the Container

```bash
docker run -d -p 8080:8080 --name taskmanager taskmanager-api
```

**Verify container is running:**
```bash
docker ps
# STATUS should show: Up X seconds
```

**Test the API:**
```bash
curl http://localhost:8080/weatherforecast
# Should return JSON weather data
```

**Access publicly from browser:**
```
http://<EC2-PUBLIC-IP>:8080/weatherforecast
```

---

## 🐛 Issues Faced & Fixes

### ❌ Platform Mismatch Error
```
WARNING: The requested image's platform (linux/arm64) does not match
the detected host platform (linux/amd64/v4)
```

**Root Cause:** Built image on Mac M-series (ARM64), tried to run on EC2 t2.micro (AMD64).

**Fix:** Build the Docker image directly on EC2 instead of copying the image from Mac.

**Alternative fix for future** (if you need to build locally and push to ECR):
```bash
# Force AMD64 build on Mac
docker buildx build --platform linux/amd64 -t taskmanager-api .
```

---

## ✅ Day 3 Checklist

- [x] .NET API created with `dotnet new webapi`
- [x] API tested locally with `dotnet run`
- [x] Multi-stage `Dockerfile` written
- [x] Code pushed to GitHub from Mac
- [x] Git installed on EC2
- [x] GitHub repo cloned directly on EC2 (`git clone`)
- [x] .NET SDK installed on EC2
- [x] Docker image built natively on EC2 (AMD64)
- [x] Container running with `docker ps` showing `Up`
- [x] API accessible at `http://<EC2-IP>:8080/weatherforecast`

---

## 🔑 Key Learnings

**Clone on the target server, don't copy files** — using `git clone` on EC2 directly is cleaner than `scp`. It keeps EC2 in sync with GitHub and naturally leads into CI/CD automation in Phase 3.

**Multi-stage Docker builds** keep images lean — always separate the build environment from the runtime environment.

**Platform architecture matters** — Mac M-series is ARM64, AWS EC2 (t2.micro/t3) is AMD64. Building the image directly on EC2 after cloning the repo eliminates this problem entirely.

**Never include secrets in Docker images** — use environment variables, AWS Secrets Manager, or SSM Parameter Store instead.

---

## 🔗 Navigation

| | Phase |
|--|-------|
| ⬅️ Previous | [Phase 1 — AWS Foundation](../phase-1-aws/README.md) |
| ➡️ Next | [Phase 3 — CI/CD Pipeline](../phase-3-cicd/README.md) |

---

## 📌 Resources

- [.NET Docker Images (Microsoft)](https://mcr.microsoft.com/en-us/product/dotnet/aspnet/about)
- [Docker Multi-Stage Builds](https://docs.docker.com/build/building/multi-stage/)
- [Docker Buildx for Cross-Platform](https://docs.docker.com/buildx/working-with-buildx/)