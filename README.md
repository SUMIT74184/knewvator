# StudyNotion Edtech Project
Study Notion – EdTech Platform (MERN + JWT + Docker)

An end-to-end EdTech platform where creators can publish courses and learners can discover, purchase, and study them. Built with MongoDB, Express.js, React.js, Node.js and secured with JWT. Containerized with Docker for easy local/dev deployment.

Repo: https://github.com/SUMIT74184/knewvator (rename in README badges/links if you change the repo name)

✨ Features

Auth & Profiles

Email/password signup & login (JWT auth, HTTP-only cookies)

Role-based access: student, instructor, admin

Profile management: avatar, bio, social links

Course Management

Create/edit courses with sections & lectures (video, PDFs, text)

Draft/Publish workflows, pricing (free/paid), tags & categories

Rich content editor for lesson notes / descriptions

Learning Experience

Course landing page, curriculum viewer

Video player with timestamps, resources, notes

Progress tracking & “continue where you left off”

Payments (pluggable)

Abstraction for payment gateway (e.g., Razorpay/Stripe).
Implementor can drop in credentials; mock mode available for dev.

Search & Discovery

Keyword search, filters (category, price, level), trending/new

Reviews & Ratings

Student reviews (1–5), course Q&A (optional)

Admin Tools

Users/courses overview, creator approvals, reports

Developer Friendly

Clear API structure, request validation, error handling

Ready Docker setup for backend, frontend, and MongoDB

🧱 Tech Stack

Frontend: React.js (Vite/CRA), React Router, Zustand/Redux (choose one), Tailwind CSS

Backend: Node.js, Express.js, Mongoose, Zod/Joi validation

Database: MongoDB

Auth: JWT (access & refresh tokens), bcrypt

Other: Multer / cloud storage SDK for uploads, Nodemailer for emails

DevOps: Docker, docker-compose

📁 Monorepo Structure
knewvator/
├─ client/                 # React app
│  ├─ src/
│  │  ├─ components/
│  │  ├─ pages/
│  │  ├─ store/            # state (Zustand/Redux)
│  │  ├─ hooks/
│  │  ├─ utils/
│  │  └─ services/         # API clients
│  └─ vite.config.ts | package.json
│
├─ server/                 # Express API
│  ├─ src/
│  │  ├─ config/           # env, db, cloud, mail
│  │  ├─ models/           # Mongoose schemas
│  │  ├─ controllers/
│  │  ├─ routes/           # /auth /courses /payments ...
│  │  ├─ middlewares/      # auth, roles, rateLimit, errors
│  │  ├─ services/         # business logic (payments, mail)
│  │  ├─ utils/            # helpers, validators
│  │  └─ app.ts | server.ts
│  └─ package.json
│
├─ docker-compose.yml
├─ .env.example
└─ README.md

⚙️ Environment Variables

Copy .env.example to backend and frontend as needed.

server/.env

NODE_ENV=development
PORT=5000

MONGO_URI=mongodb://mongo:27017/knewvator      # when using docker-compose
# MONGO_URI=mongodb://127.0.0.1:27017/knewvator # local without docker

JWT_ACCESS_SECRET=replace_me_access
JWT_REFRESH_SECRET=replace_me_refresh
JWT_ACCESS_EXPIRES=15m
JWT_REFRESH_EXPIRES=7d

CLIENT_URL=http://localhost:5173
CORS_ORIGIN=http://localhost:5173

MAIL_FROM=no-reply@studynotion.example
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your_user
SMTP_PASS=your_pass

# Optional: cloud storage
CLOUD_STORAGE_PROVIDER=local
UPLOAD_DIR=/uploads


# Payments (set one or both; mock if empty)
PAYMENT_PROVIDER=mock
# RAZORPAY_KEY_ID=
# RAZORPAY_KEY_SECRET=
# STRIPE_SECRET_KEY=


client/.env

VITE_API_URL=http://localhost:5000/api

🐳 Quick Start with Docker

Requirements: Docker & docker-compose installed.

# from repo root
docker-compose up --build


Services:

Frontend: http://localhost:5173

Backend: http://localhost:5000/api

MongoDB: mongodb://localhost:27017 (container: mongo)

To run in detached mode:

docker-compose up -d


Stop:

docker-compose down


Tip: Add a volume to persist Mongo data between runs (example compose below).

docker-compose.yml (example)

version: "3.8"
services:
  mongo:
    image: mongo:7
    container_name: mongo
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  server:
    build: ./server
    container_name: api
    env_file: ./server/.env
    ports:
      - "5000:5000"
    depends_on:
      - mongo

  client:
    build: ./client
    container_name: web
    env_file: ./client/.env
    ports:
      - "5173:5173"
    depends_on:
      - server

volumes:
  mongo_data:

🧪 Local Development (without Docker)

Backend

cd server
cp .env.example .env
npm i
npm run dev   # nodemon


Frontend

cd client
cp .env.example .env
npm i
npm run dev   # vite/CRA dev server

🔐 Auth Flow (JWT)

Signup/Login → issue Access Token (short lived) + Refresh Token (long lived).

Access token returned in JSON; refresh token stored as HTTP-only cookie.

Protected routes use Authorization: Bearer <access_token>.

POST /auth/refresh rotates tokens; POST /auth/logout clears cookie.

🔌 API Overview

Base URL: /api

Auth

POST /auth/signup – { name, email, password, role? }

POST /auth/login – { email, password }

POST /auth/refresh

POST /auth/logout

GET /auth/me – current user profile (requires auth)

Users

GET /users/:id

PATCH /users/:id – update profile (self or admin)

Courses

GET /courses – query params: q, category, price, level, sort

GET /courses/:id

POST /courses – instructor only

PATCH /courses/:id – instructor/admin

DELETE /courses/:id – instructor/admin

Sections & Lectures

POST /courses/:id/sections

POST /sections/:id/lectures – upload video/resource (Multer/S3)

PATCH /lectures/:id

DELETE /lectures/:id

Enrollments & Progress

POST /enroll/:courseId – creates order (mock/real)

GET /progress/:courseId

POST /progress/:courseId/checkpoint – { lectureId, completedAt }

Reviews

POST /reviews/:courseId

GET /reviews/:courseId

Admin

GET /admin/metrics

PATCH /admin/courses/:id/status – approve/unlist

Each route is validated (Zod/Joi). Errors are returned as:

{ "success": false, "message": "Validation error", "errors": { "field": "issue" } }

🧰 Scripts

Backend (server/package.json)

dev – start with nodemon

start – production start

lint – eslint

test – unit tests (Jest/Vitest)

seed – seed sample data (see src/scripts/seed.ts)

Frontend (client/package.json)

dev – start dev server

build – production build

preview – preview build

lint – eslint

🗄️ Data Models (example)

User

{
  name: string,
  email: string, unique,
  passwordHash: string,
  role: "student" | "instructor" | "admin",
  avatarUrl?: string,
  bio?: string,
  socials?: { twitter?: string; linkedin?: string; github?: string },
  createdAt: Date
}


Course

{
  title: string,
  slug: string, unique,
  description: string,
  price: number,           // 0 for free
  level: "beginner" | "intermediate" | "advanced",
  thumbnailUrl?: string,
  category?: string,
  tags?: string[],
  instructor: ObjectId<User>,
  sections: ObjectId<Section>[],
  status: "draft" | "published",
  rating: number, reviewsCount: number,
  createdAt: Date, updatedAt: Date
}

🚀 Deployment Notes

Build frontend and serve via CDN or behind the API’s static handler / separate Nginx.

Set CLIENT_URL & CORS properly.

Use managed MongoDB (Atlas) in production.

Store secrets via your platform’s secret manager.

Configure object storage (S3, Cloudflare R2, etc.) for video & asset delivery.

Consider HLS/streaming for large videos.

🧩 Extensibility

Swap payment provider by implementing PaymentService interface.

Add more roles/permissions via roleGuard.

Plug analytics (PostHog/GA4) on the frontend.

Add caching (Redis) for catalog and metrics.

✅ Checklist Before Opening PRs

 Linted & tests pass

 Env vars documented/updated

 New endpoints have validation & role guards

 Errors use standard JSON shape

 If schema changed, add/update seed & migration notes

🤝 Contributing

Fork the repo

Create a feature branch: git checkout -b feat/awesome-thing

Commit changes with clear messages

Open a PR describing the change & testing steps

🧭 Quick Commands (copy-paste)
# clone
git clone https://github.com/SUMIT74184/knewvator.git
cd knewvator

# envs
cp server/.env.example server/.env
cp client/.env.example client/.env

# run with Docker
docker-compose up --build  


# OR run manually
(cd server && npm i && npm run dev) &
(cd client && npm i && npm run dev)


If you want, I can tailor this README to your exact folders, scripts, and current routes from your repo—just paste your server/routes and package.json scripts and I’ll wire them in.

C
