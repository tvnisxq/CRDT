# CRDT — Conflict-Free Replicated Data Type Editor

A real-time, multi-user collaborative code editor that uses **Conflict-Free Replicated Data Types (CRDTs)** to synchronize edits across clients without conflicts — similar in spirit to Google Docs, but for code.

Multiple users can join the same document, see each other in a live **Users** panel, and edit the same file simultaneously with automatic, conflict-free merging of changes.

<p align="left">
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=for-the-badge&logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Vite-Powered-B14CE8?style=for-the-badge&logo=vite&logoColor=white" alt="Vite" />
  <img src="https://img.shields.io/badge/Node.js-Backend-3C873A?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Docker-Supported-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
</p>
<p align="left">
  <img src="https://img.shields.io/badge/AWS%20ECR-Container%20Registry-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white" alt="AWS ECR" />
  <img src="https://img.shields.io/badge/AWS%20ECS-Orchestration-FF9900?style=for-the-badge&logo=amazonecs&logoColor=white" alt="AWS ECS" />
  <img src="https://img.shields.io/badge/AWS%20EC2-Compute-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white" alt="AWS EC2" />
</p>
<p align="left">
  <img src="https://img.shields.io/badge/Real--Time-WebSockets-06D6A0?style=for-the-badge&logo=socketdotio&logoColor=white" alt="WebSockets" />
  <img src="https://img.shields.io/badge/Sync-Conflict--Free-EF476F?style=for-the-badge" alt="Conflict-Free Sync" />
  <img src="https://img.shields.io/badge/License-MIT-FFD60A?style=for-the-badge" alt="License MIT" />
  <img src="https://img.shields.io/badge/Platform-Windows%20%7C%20macOS%20%7C%20Linux-9B5DE5?style=for-the-badge" alt="Platform" />
</p>

---

## Features

- Real-time collaborative text/code editing
- Conflict-free synchronization using CRDTs (no last-write-wins data loss)
- Live "Users" presence panel showing everyone connected to a session
- Fast, modern frontend built with **React + Vite**
- Lightweight Node.js backend for real-time sync (WebSockets)
- Fully containerized with Docker (multi-stage builds via Buildx)
- Deployable to AWS using **ECR**, **ECS**, and **EC2**

---

## Tech Stack

| Layer      | Technology                          |
|------------|--------------------------------------|
| Frontend   | React, Vite, JavaScript              |
| Backend    | Node.js, WebSockets, CRDT sync logic |
| Container  | Docker, Docker Buildx                |
| Cloud      | AWS ECR, ECS, EC2, Application Load Balancer |

---

## Project Structure

```
CRDT/
├── backend/          # Node.js server (WebSocket + CRDT sync logic)
├── frontend/          # React + Vite client application
├── dockerfile         # Multi-stage build for frontend + backend
├── .dockerignore
└── README.md
```

---

## Getting Started (Local Development)

### Prerequisites

- [Node.js](https://nodejs.org/) v18+
- npm (or yarn/pnpm)
- Git

### 1. Clone the repository

```bash
git clone https://github.com/tvnisxq/CRDT.git
cd CRDT
```

### 2. Set up the backend

```bash
cd backend
npm install
npm start
```

By default the backend server runs on `http://localhost:8080` (adjust to match your `server.js` configuration).

### 3. Set up the frontend

Open a new terminal window:

```bash
cd frontend
npm install
npm run dev
```

The Vite dev server will start (typically at `http://localhost:5173`). Open it in your browser, enter a username, and start collaborating. Open the same URL in another tab/browser to simulate a second user.

### 4. Environment variables

Create a `.env` file in `frontend/` and/or `backend/` as needed to point the client at your backend's WebSocket URL, e.g.:

```env
# frontend/.env
VITE_SERVER_URL=ws://localhost:8080
```

```env
# backend/.env
PORT=8080
```

---

## Running with Docker

The project ships with a single `dockerfile` that builds the frontend and backend into one deployable image.

### Build the image

```bash
docker build -t crdt-app:latest .
```

### Run the container

```bash
docker run -p 8080:8080 crdt-app:latest
```

Then visit `http://localhost:8080` in your browser.

### Build with Docker Buildx (multi-platform)

Buildx allows building images for multiple architectures (e.g., `amd64` for EC2/ECS Fargate, `arm64` for Graviton instances).

```bash
# Create and use a new builder instance (one-time setup)
docker buildx create --name crdt-builder --use

# Build and push a multi-platform image
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t crdt-app:latest \
  --push .
```

---

## Deploying to AWS (ECR → ECS/EC2)

### 1. Authenticate Docker to AWS ECR

```bash
aws ecr get-login-password --region <aws-region> | \
  docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com
```

### 2. Create an ECR repository (one-time)

```bash
aws ecr create-repository --repository-name crdt-app --region <aws-region>
```

### 3. Build and tag the image for ECR

```bash
docker buildx build \
  --platform linux/amd64 \
  -t <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/crdt-app:latest \
  --push .
```

### 4. Deploy on ECS (Fargate or EC2 launch type)

1. Create/update an **ECS Task Definition** referencing the pushed image:
   `<aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/crdt-app:latest`
2. Configure the container port (e.g. `8080`) and any required environment variables.
3. Create or update the **ECS Service**, attaching it to an **Application Load Balancer (ALB)** target group for public access.
4. Force a new deployment to roll out the latest image:

```bash
aws ecs update-service \
  --cluster <cluster-name> \
  --service <service-name> \
  --force-new-deployment \
  --region <aws-region>
```

### 5. Deploy on a plain EC2 instance (alternative to ECS)

```bash
# On the EC2 instance
aws ecr get-login-password --region <aws-region> | \
  docker login --username AWS --password-stdin <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com

docker pull <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/crdt-app:latest

docker run -d -p 80:8080 --restart unless-stopped \
  --name crdt-app \
  <aws-account-id>.dkr.ecr.<aws-region>.amazonaws.com/crdt-app:latest
```

Traffic is routed through an **Application Load Balancer** to the EC2 instance(s) or ECS service, giving you a public URL such as:

```
http://<your-alb-dns-name>.<aws-region>.elb.amazonaws.com
```

---

## Usage

1. Open the deployed (or local) URL in a browser.
2. Enter a username to join a session — it will appear in the **Users** panel.
3. Open the same URL in another browser/tab and join with a different username.
4. Start typing — edits from all connected users sync in real time, conflict-free.

---

## Roadmap / Ideas

- [ ] Syntax highlighting per language
- [ ] Persistent document storage
- [ ] Room/session-based document sharing via unique links
- [ ] Cursor position and selection presence indicators
- [ ] CI/CD pipeline for automated ECR push + ECS deployment

---

## Contributing

Contributions, issues, and feature requests are welcome. Feel free to open a pull request or an issue.

---

## License

This project is currently unlicensed. Add a `LICENSE` file to specify usage terms (e.g., MIT).

---

## Author

**Tanishq** ([@tvnisxq](https://github.com/tvnisxq))
