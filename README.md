# Linker Application – Docker Build & DevSecOps Pipeline

A full-stack link-sharing application built with Next.js (frontend) and NestJS + Prisma (backend), containerised with Docker and secured with a DevSecOps CI/CD pipeline using GitHub Actions.

---

## Table of Contents
- [Application Stack](#application-stack)
- [Repository Structure](#repository-structure)
- [DevSecOps Pipeline](#devsecops-pipeline)
- [Security Tools](#security-tools)
- [Docker Images](#docker-images)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [Pipeline Setup](#pipeline-setup)

---

## Application Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js 15, TypeScript, Tailwind CSS |
| Backend | NestJS, Prisma ORM, TypeScript |
| Database | PostgreSQL |
| Containerisation | Docker, Docker Compose |
| CI/CD | GitHub Actions |
| Registry | Docker Hub |

---

## Repository Structure
```
linker-monorepo/
├── frontend/                        # Next.js application
│   ├── Dockerfile
│   └── next.config.mjs
├── backend/                         # NestJS + Prisma API
│   ├── Dockerfile
│   ├── entrypoint-script.sh
│   └── prisma/
│       └── schema.prisma
├── .github/
│   └── workflows/
│       ├── ci.yml                   # Main CI/CD pipeline
│       └── codeql.yml               # CodeQL SAST analysis
└── docker-compose.yml               # Local development stack
```

---

## DevSecOps Pipeline

The pipeline runs automatically on every push and pull request to `main`.
```
Push / PR to main
        │
        ▼
┌─────────────────┐
│  npm audit      │  ← Dependency vulnerability scan (SCA)
│  (SCA)          │    on both frontend and backend
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Docker Build   │  ← Builds frontend and backend images
│  + Trivy Scan   │  ← Scans images for CVEs (CRITICAL, HIGH)
└────────┬────────┘
         │
         ▼ (main branch only)
┌─────────────────┐
│  Push to        │  ← Pushes images to Docker Hub
│  Docker Hub     │    Tagged with :latest and :commit-sha
└─────────────────┘
```

### CodeQL (SAST)
Runs separately on push, pull request, and every Monday at 3am:
```
Checkout → Initialize CodeQL → Install Dependencies → Analyze
```

---

## Security Tools

| Tool | Type | Purpose |
|---|---|---|
| npm audit | SCA | Scans dependencies for known vulnerabilities |
| Trivy | Image Scan | Scans Docker images for OS and library CVEs |
| CodeQL | SAST | Static analysis of JavaScript/TypeScript source code |

---

## Docker Images

Images are automatically built and pushed to Docker Hub on every merge to `main`.

| Image | Docker Hub |
|---|---|
| Backend | `yourusername/linker-backend` |
| Frontend | `yourusername/linker-frontend` |

Each image is tagged with:
- `:latest` — always points to the most recent build
- `:commit-sha` — immutable tag tied to the exact commit

Pull the images:
```bash
docker pull yourusername/linker-backend:latest
docker pull yourusername/linker-frontend:latest
```

---

## Getting Started

### Prerequisites
- Docker and Docker Compose installed
- Node.js 20+

### Run locally with Docker Compose
```bash
# Clone the repo
git clone https://github.com/yourusername/linker-monorepo.git
cd linker-monorepo

# Copy environment file
cp .env.example .env

# Start all services
docker compose up -d --build

# Run database migrations
docker compose exec backend npx prisma migrate deploy
```

| Service | URL |
|---|---|
| Frontend | http://localhost:3000 |
| Backend API | http://localhost:3001/api |
| Database | localhost:5432 |

---

## Environment Variables

Copy `.env.example` to `.env` and fill in the values:

| Variable | Description |
|---|---|
| `POSTGRES_DB` | Database name |
| `POSTGRES_USER` | Database username |
| `POSTGRES_PASSWORD` | Database password |
| `DATABASE_URL` | Full PostgreSQL connection string |
| `JWT_SECRET` | JWT signing secret (min 32 chars) |
| `FRONTEND_ORIGIN` | Allowed CORS origin |
| `NEXT_PUBLIC_API_URL` | Backend API URL for the browser |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name (image uploads) |
| `CLOUDINARY_API_KEY` | Cloudinary API key |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret |

---

## Pipeline Setup

### GitHub Secrets Required

Add these secrets in your GitHub repo under **Settings → Secrets and variables → Actions**:

| Secret | Description |
|---|---|
| `DOCKERHUB_USERNAME` | Your Docker Hub username |
| `DOCKERHUB_TOKEN` | Docker Hub access token with Read & Write permissions |

### Creating a Docker Hub Access Token
1. Login at [hub.docker.com](https://hub.docker.com)
2. Go to **Account Settings → Personal Access Tokens**
3. Click **Generate New Token**
4. Set permissions to **Read & Write**
5. Copy the token and add it as `DOCKERHUB_TOKEN` in GitHub secrets

---

## Database Schema

Managed by Prisma with three models:

- **User** — authentication (email + hashed password)
- **UserProfile** — display name and photo URL (1-to-1 with User)
- **Link** — social/professional links per user (platform + URL)