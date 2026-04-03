# Talent Match — Innovating the Recruitment System Using Artificial Intelligence and Machine Learning

## Introduction

Large institutions like DRDO receive thousands of applicants for their technical interviews. These are mostly boardroom interviews where candidates are manually matched with interviewers based on the interviewers' expertise and the candidates' experience, education, and job role — all to ensure that the right person is hired. A single mis-hire can cause significant loss to an organisation. However, this manual matching process is time-consuming, cost-inefficient, and highly prone to human error.

**Talent Match** solves this problem by leveraging advanced AI-driven candidate-to-interviewer mapping using:

- **Semantic search** and **NLP models** for best-in-class contextual and relevance matching
- **Large Language Models (LLMs)** — which have seen significant recent advancements — that now surpass traditional NLP models and even human judgment in precision, while operating at machine speed; this directly addresses the twin challenges of time and accuracy

### Fairness & Scoring

To ensure fairness in interviews, Talent Match does not only evaluate the candidate. It also analyses the **interviewer** across dimensions such as:

- Relevance, depth, and difficulty of questions asked
- Expertise level of the interviewer who asked each question

These factors are used to **normalise candidate scores**, ensuring a level playing field. Final scores are produced by a **complex weighted, priority-based scoring algorithm** that drives the ultimate hiring decision.

### Candidate Profile Validation

Candidate profiles are validated using AI to detect and prevent **fraudulent submissions**, ensuring the integrity of the entire process.

---

## Quantifiable Impact

### Impact on the Organisation

| Metric | Estimated Impact |
|---|---|
| **Time savings** | Up to **70–80% reduction** in manual matching time; what takes days can be completed in minutes |
| **Cost efficiency** | Reduction in HR operational costs by **30–50%** through automation of screening and matching |
| **Mis-hire reduction** | AI-driven precision matching can lower mis-hire rates by **40–60%**, avoiding costs that typically equal **1–3× the annual salary** of the role |
| **Interviewer utilisation** | Ensures interviewers are matched only to candidates within their domain, improving panel **utilisation efficiency by 50%+** |
| **Bias mitigation** | Normalised scoring reduces subjective bias, improving **diversity hiring outcomes** |
| **Fraud prevention** | AI profile validation can detect fraudulent credentials, protecting against costly bad hires |

### Market Opportunity

| Dimension | Data Point |
|---|---|
| **Global HR Tech market size** | ~**$40 billion** (2024), projected to exceed **$90 billion by 2030** (CAGR ~13%) |
| **AI in recruitment market** | Expected to grow from ~$590 million (2022) to **$1.1 billion by 2030** |
| **Target segments** | Government defence organisations (DRDO, ISRO), PSUs, large enterprises, IT firms, MNCs |
| **Enterprise recruitment spend** | Large organisations spend **$4,000–$10,000 per hire**; automation can cut this by up to **50%** |
| **Scalability** | SaaS model enables rapid scaling across industries with minimal incremental cost |

---

## 🛠️ Installation — Local Development

### Prerequisites

| Tool | Version | Notes |
|---|---|---|
| **Node.js** | `v20+` | LTS recommended |
| **npm** | `v9+` | Bundled with Node.js |
| **Git** | Any recent version | For cloning the repo |
| **Docker** *(optional)* | `v24+` | Required only for container-based runs |

### Step 1 — Clone the Repository

```bash
git clone https://github.com/abhinav761/Interview-Regulator-Frontend-main.git
cd Interview-Regulator-Frontend-main
```

### Step 2 — Install Dependencies

```bash
npm ci --legacy-peer-deps
```

> `--legacy-peer-deps` is required to resolve peer dependency conflicts between some Radix UI packages.

### Step 3 — Start the Development Server

```bash
npm run dev
```

The app will be available at **http://localhost:5173** (Vite default).

### Other Useful Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start local dev server with HMR |
| `npm run build` | Production build → outputs to `dist/` |
| `npm run build:dev` | Dev-mode build (un-minified) |
| `npm run lint` | Run ESLint checks |
| `npm run preview` | Locally preview the production build |

---

## 🐳 Running with Docker (Local)

### Option A — Docker Compose *(recommended)*

```bash
# Build and start the container
docker compose up --build -d

# App runs at http://localhost:80

# Stop the container
docker compose down
```

### Option B — Docker CLI

```bash
# Build the image
docker build -t interview-regulator-frontend:latest .

# Run the container
docker run -d \
  --name interview-regulator-app \
  --restart unless-stopped \
  -p 80:80 \
  interview-regulator-frontend:latest
```

The container uses a **multi-stage build**:
- **Stage 1 (builder)** — Node 20 Alpine: installs deps and runs `npm run build`
- **Stage 2 (production)** — Nginx 1.27 Alpine: serves the `dist/` folder with a hardened nginx config

A health check endpoint is available at **http://localhost/health** → returns `200 OK`.

---

## 🚀 CI/CD Pipeline — GitHub Actions

The pipeline is defined in [`.github/workflows/docker-ci-cd.yml`](.github/workflows/docker-ci-cd.yml) and runs automatically.

### Pipeline Overview

```
Push / PR → [ Job 1: Lint & Build ] → [ Job 2: Docker Build & Push ] → [ Job 3: Deploy to Production ]
```

| Job | Trigger | What it does |
|---|---|---|
| **Lint & Build** | Every push & PR to `main` / `develop` | Installs deps, runs ESLint, runs `npm run build`, uploads `dist/` as artifact |
| **Docker Build & Push** | Push to `main` or `develop` only | Builds multi-stage Docker image, pushes to **GitHub Container Registry (GHCR)** |
| **Deploy to Production** | Push to `main` only | SSHs into the production server, pulls the latest image, restarts the container |

### Image Tags Strategy

| Tag | When applied |
|---|---|
| `:latest` | Push to `main` branch |
| `:develop` | Push to `develop` branch |
| `:sha-<commit>` | Every push (short commit SHA) |
| `:<branch-name>` | Every push |

### Required GitHub Secrets & Variables

Set these in **GitHub → Settings → Secrets and Variables → Actions**:

| Secret / Variable | Type | Description |
|---|---|---|
| `DEPLOY_HOST` | Secret | IP address or hostname of your production server |
| `DEPLOY_USER` | Secret | SSH username on the production server |
| `DEPLOY_SSH_KEY` | Secret | Private SSH key (the server must have the matching public key in `~/.ssh/authorized_keys`) |
| `DEPLOY_PORT` | Secret *(optional)* | SSH port — defaults to `22` if not set |
| `PRODUCTION_URL` | Variable | Public URL of the deployed app (shown in GitHub environments UI) |
| `GITHUB_TOKEN` | Auto-provided | Used to authenticate with GHCR — no manual setup needed |

### Setting Up the Production Server

On your production server, ensure Docker is installed and the deploy user has Docker permissions:

```bash
# Install Docker (Ubuntu/Debian)
sudo apt update && sudo apt install -y docker.io
sudo systemctl enable --now docker

# Add the deploy user to the docker group
sudo usermod -aG docker <DEPLOY_USER>

# Allow GHCR images to be pulled (login once manually)
echo <GITHUB_TOKEN> | docker login ghcr.io -u <GITHUB_USERNAME> --password-stdin
```

### Trigger the Pipeline

```bash
# Trigger CI (lint + build + docker push) — push to develop
git push origin develop

# Trigger full pipeline including production deploy — merge/push to main
git push origin main
# or: open a PR → main and merge it
```

### Pulling & Running the Published Image Manually

```bash
# Pull the latest production image from GHCR
docker pull ghcr.io/abhinav761/interview-regulator-frontend-main:latest

# Run it
docker run -d \
  --name interview-regulator-app \
  --restart unless-stopped \
  -p 80:80 \
  ghcr.io/abhinav761/interview-regulator-frontend-main:latest
```

---

## 📁 Project Structure

```
Interview-Regulator-Frontend-Main/
├── .github/
│   └── workflows/
│       └── docker-ci-cd.yml   # GitHub Actions CI/CD pipeline
├── src/                       # React + TypeScript source code
├── public/                    # Static assets
├── dist/                      # Production build output (git-ignored)
├── Dockerfile                 # Multi-stage Docker build
├── docker-compose.yml         # Local container orchestration
├── nginx.conf                 # Production Nginx configuration
├── vite.config.ts             # Vite bundler config
├── tailwind.config.ts         # Tailwind CSS config
└── package.json               # Scripts and dependencies
```

---

## 🔗 Links

- **Live Project**: https://lovable.dev/projects/019da328-a341-4e2b-8a30-c494bee4a04e
- **GitHub Repository**: https://github.com/abhinav761/Interview-Regulator-Frontend-main
- **Issues / Bug Reports**: https://github.com/abhinav761/Interview-Regulator-Frontend-main/issues
