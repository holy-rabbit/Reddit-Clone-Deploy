# 🚀 Reddit Clone — CI Pipeline (Application Repo)

[![Next.js](https://img.shields.io/badge/Frontend-Next.js-000000?logo=nextdotjs&logoColor=white)](https://nextjs.org/)
[![Firebase](https://img.shields.io/badge/Backend-Firebase-FFCA28?logo=firebase&logoColor=black)](https://firebase.google.com/)
[![Jenkins](https://img.shields.io/badge/CI-Jenkins-D24939?logo=jenkins&logoColor=white)](https://www.jenkins.io/)
[![SonarQube](https://img.shields.io/badge/Code%20Quality-SonarQube-4E9BCD?logo=sonarqube&logoColor=white)](https://www.sonarsource.com/)
[![Trivy](https://img.shields.io/badge/Security-Trivy-1904DA)](https://aquasecurity.github.io/trivy/)
[![Docker](https://img.shields.io/badge/Container-Docker-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)

A Reddit clone built with **Next.js, Chakra UI, Recoil, and Firebase**, wired into a Jenkins CI pipeline that lints, scans, containerizes, and hands off deployment to a separate GitOps repo via ArgoCD.

This is the **application + CI** repo. Deployment manifests live in the companion repo:
👉 **[Reddit-Clone-Deploy-GitOps](https://github.com/holy-rabbit/Reddit-Clone-Deploy-GitOps)**

---

## 📐 CI/CD Flow

```
git push (main)
      │
      ▼
┌─────────────────────────── Jenkins: Reddit-Clone-CI ───────────────────────────┐
│ 1. Clean workspace                                                             │
│ 2. Checkout from GitHub                                                        │
│ 3. SonarQube static analysis (project: Reddit-Clone-CI)                        │
│ 4. Quality Gate check                                                          │
│ 5. Install dependencies (npm ci)                                               │
│ 6. Trivy filesystem scan  → trivyfs.txt                                        │
│ 7. Build Docker image → push to Docker Hub (holyrabbit/reddit-clone-pipeline)  │
│ 8. Trivy image scan (HIGH/CRITICAL) → trivyimage.txt                           │
│ 9. Clean up local Docker images                                                │
│ 10. Trigger downstream job: Reddit-Clone-CD (passes IMAGE_TAG)                 │
└──────────────────────────────────────────────────────────────────────────────┘
      │
      ▼  (remote trigger, build param: IMAGE_TAG)
Reddit-Clone-CD job in the GitOps repo → updates deployment.yaml → ArgoCD syncs to EKS
      │
      ▼
Email notification sent with build log + Trivy scan reports attached
```

---

## 🧰 Tech Stack

**Application**
- Next.js 12, React 17
- Chakra UI, Emotion
- Recoil (state management)
- Firebase (Auth, Firestore, Cloud Functions)

**Pipeline**
- Jenkins (JDK + Node 16 tool config)
- SonarQube — static code analysis + quality gate
- Trivy — filesystem scan and post-build image scan (HIGH/CRITICAL severities)
- Docker — image build and push to Docker Hub
- Email extension plugin — build notifications with attached scan reports

---

## 📦 Key Files

| File | Purpose |
|------|---------|
| `Jenkinsfile` | Full CI pipeline definition (stages listed above) |
| `Dockerfile` | Builds the app image (`node:19-alpine3.15`, runs `npm run dev` on port 3000) |
| `src/` | Next.js application source (pages, components, hooks, Firebase config) |
| `functions/` | Firebase Cloud Functions |

---

## ⚙️ Jenkins Pipeline Stages (from `Jenkinsfile`)

1. **Clean workspace**
2. **Checkout from Git** — `main` branch of this repo
3. **SonarQube Analysis** — runs `sonar-scanner` against project `Reddit-Clone-CI`
4. **Quality Gate** — waits on SonarQube's quality gate result
5. **Install Dependencies** — `npm ci` (memory-capped, offline-preferred, legacy peer deps)
6. **Trivy FS Scan** — scans the working directory, output saved to `trivyfs.txt`
7. **Build & Push Docker Image** — builds `holyrabbit/reddit-clone-pipeline`, pushes both a versioned tag (`1.0.0-<BUILD_NUMBER>`) and `latest`
8. **Trivy Image Scan** — scans the pushed image for HIGH/CRITICAL CVEs, output to `trivyimage.txt`
9. **Cleanup Artifacts** — removes local images to keep the build agent clean
10. **Trigger CD Pipeline** — calls the `Reddit-Clone-CD` job on the Jenkins server via the Jenkins API, passing the new `IMAGE_TAG`

**Post-build:** an email is always sent with the build result, job/build info, and both Trivy reports attached.

---

## 🚀 Running Locally

```bash
git clone https://github.com/holy-rabbit/Reddit-Clone-Deploy.git
cd Reddit-Clone-Deploy
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

You'll need a Firebase project and the relevant `NEXT_PUBLIC_FIREBASE_*` environment variables set up (see `src/firebase/clientApp.ts`).

### Build the Docker image
```bash
docker build -t reddit-clone-pipeline .
docker run -p 3000:3000 reddit-clone-pipeline
```

---

## 🔗 Related Repository

| Repo | Purpose |
|------|---------|
| [Reddit-Clone-Deploy](https://github.com/holy-rabbit/Reddit-Clone-Deploy) | This repo — app source + CI pipeline |
| [Reddit-Clone-Deploy-GitOps](https://github.com/holy-rabbit/Reddit-Clone-Deploy-GitOps) | Kubernetes manifests, updated by CD job and synced by ArgoCD |

---

## 👤 Author

**holy-rabbit**



