# CareerCompass

A personalized career planning platform that helps you plan, follow, and
track a learning roadmap for the career you want.

> Plan your career like a trail — one waypoint at a time.

CareerCompass is a full-stack JavaScript application (React on the
frontend, Node/Express on the backend). You pick or build a roadmap for
the career you're aiming at, break it into waypoints, log your progress
day by day, and watch a dashboard turn that activity into streaks and a
skill profile.

---

## Table of contents

1. [Overview](#overview)
2. [Features](#features)
3. [Tech stack](#tech-stack)
4. [Project structure](#project-structure)
5. [Getting started](#getting-started)
6. [Environment variables](#environment-variables)
7. [Trying out the app](#trying-out-the-app)
8. [API reference](#api-reference)
9. [Data model](#data-model)
10. [Pushing this project to GitHub](#pushing-this-project-to-github)
11. [Author](#author)

---

## Overview

The core idea behind CareerCompass is that a career goal ("become a
Data Analyst", "become a Backend Developer") is easier to pursue when
it's broken into a visible, ordered path — a **trail** — rather than a
vague to-do list. Each stop on that trail is a **waypoint** (a
milestone: a skill, a project, a certification). You either:

- follow a **curated roadmap** the app already ships with,
- **customize** one of those curated roadmaps once it's yours, or
- **build your own** from a blank trail.

As you work, you log short daily notes and a completion percentage
against each waypoint. The dashboard aggregates that activity into a
streak counter, an overall completion percentage, and a skill profile
that grows automatically as you check things off.

## Features

| Area | What it does |
|---|---|
| **Public home page** | Explains the platform to visitors before they sign in. |
| **Accounts** | Register/login with JWT-based sessions; passwords hashed with bcrypt. |
| **Profile** | Edit your headline, bio, location, interests, and a skill base with per-skill levels. |
| **Roadmaps — three ways** | Build your own from scratch, follow a curated roadmap as-is, or clone a curated roadmap onto your profile and customize it freely. |
| **Curated roadmaps included** | Cybersecurity Analyst, Data Analyst, Frontend Developer, Backend Developer. |
| **Daily progress tracking** | Leave a short note and a completion % against any waypoint, any day. |
| **Dashboard** | Streaks, overall completion, and an auto-built skill profile. |
| **Trail visualization** | A signature winding SVG trail connects each roadmap's waypoints in order. |

## Tech stack

| Layer | Technology |
|---|---|
| Frontend | React 18, React Router, Vite, plain CSS using a design-token system |
| Backend | Node.js, Express |
| Auth | JWT (`jsonwebtoken`) + bcrypt password hashing |
| Storage | A structured JSON-file datastore (`backend/src/utils/db.js`) — the controller layer talks to a small data-access module, so swapping in MongoDB or Postgres later doesn't require touching route/controller logic |

## Project structure

```
CareerCompass/
├── backend/
│   ├── server.js                 # Express entrypoint
│   └── src/
│       ├── config/                # App-level configuration
│       ├── controllers/           # Route handlers (auth, profile, roadmaps, progress)
│       ├── data/db.json           # JSON datastore (seeded on boot)
│       ├── middleware/            # Auth guard, central error handler
│       ├── models/                # User / Roadmap / UserRoadmap shape factories
│       ├── routes/                # Express routers
│       ├── seed/                  # Curated roadmap seed data
│       └── utils/                 # db.js (datastore), jwt.js (token helpers)
└── frontend/
    └── src/
        ├── api/                   # Axios client
        ├── components/            # Navbar, TrailPath, MilestoneItem, ProgressRing, ...
        ├── context/                # AuthContext
        └── pages/                 # Landing, Login, Register, Dashboard, Explore,
                                    # RoadmapBuilder, RoadmapDetail, Profile
```

## Getting started

### Prerequisites

- Node.js 18+ and npm
- Two terminal windows/tabs (one for backend, one for frontend), since both need to run at once

### 1. Backend

```bash
cd backend
npm install
cp .env.example .env      # edit JWT_SECRET before deploying anywhere real
npm run dev                # starts on http://localhost:5000
```

On first boot, the backend automatically seeds four curated roadmap
templates into `src/data/db.json` (see `src/seed/seedRoadmaps.js`). You
don't need to run any separate seed command.

### 2. Frontend

In a second terminal:

```bash
cd frontend
npm install
cp .env.example .env      # points at http://localhost:5000/api by default
npm run dev                 # starts on http://localhost:5173
```

Open **http://localhost:5173**. The Vite dev server proxies `/api`
requests to the backend, so both servers need to be running at the same
time for the app to work.

## Environment variables

| File | Variable | Purpose |
|---|---|---|
| `backend/.env` | `JWT_SECRET` | Signing secret for auth tokens — change this before any real deployment. |
| `backend/.env` | `PORT` | Port the Express server listens on (defaults to `5000`). |
| `frontend/.env` | `VITE_API_URL` | Base URL the frontend calls for the API (defaults to `http://localhost:5000/api`). |

Both `.env.example` files document the exact keys expected — copy them
to `.env` and adjust as needed rather than committing real secrets.

## Trying out the app

A quick end-to-end walkthrough once both servers are running:

1. **Register** an account — you're taken straight to your profile.
2. Fill in a headline and a couple of skills, then save.
3. Go to **Explore** and either follow a curated roadmap as-is,
   customize one, or use **+ New roadmap** to build your own.
4. Open the roadmap, expand a waypoint, and log today's progress with a
   note and a completion %. Mark it complete when it's done.
5. Check the **Dashboard** — your streak, overall completion, and skill
   profile update automatically from that activity.

## API reference

All authenticated routes require a valid JWT in the `Authorization`
header (`Bearer <token>`), issued at login/register.

| Method | Route | Auth | Purpose |
|---|---|:---:|---|
| POST | `/api/auth/register` | – | Create an account |
| POST | `/api/auth/login` | – | Sign in |
| GET | `/api/auth/me` | ✓ | Get the current user |
| GET / PUT | `/api/profile` | ✓ | View / edit profile |
| GET | `/api/roadmaps/templates` | – | Browse curated roadmaps |
| GET | `/api/roadmaps/templates/:id` | – | Get one curated roadmap |
| GET / POST | `/api/my-roadmaps` | ✓ | List / create your own roadmaps |
| GET / PUT / DELETE | `/api/my-roadmaps/:id` | ✓ | View, edit, or delete one roadmap |
| POST / DELETE | `/api/my-roadmaps/:id/milestones[/:milestoneId]` | ✓ | Add / remove a waypoint |
| PATCH | `/api/my-roadmaps/:id/milestones/:milestoneId` | ✓ | Toggle a waypoint's completion |
| POST | `/api/my-roadmaps/:id/milestones/:milestoneId/logs` | ✓ | Add a daily progress log |
| GET | `/api/my-roadmaps/dashboard/summary` | ✓ | Get the streak / progress / skill summary |

## Data model

A rough shape of the three core records, for context when reading the
controllers/models:

- **User** — account credentials, plus profile fields (headline, bio,
  location, interests, skills with levels).
- **Roadmap template** (curated) — a named career path with an ordered
  list of waypoints; read-only, browsable via `/api/roadmaps/templates`.
- **UserRoadmap** (owned) — a roadmap that belongs to a user, either
  built from scratch or cloned from a template. Each waypoint tracks a
  completion flag and a list of dated logs (`note`, `percent`).

The dashboard summary endpoint derives streaks, overall completion, and
the skill profile from the set of a user's `UserRoadmap` records — none
of that is stored separately, it's computed on request.

## Pushing this project to GitHub

See [`GIT_PUSH_PLAN.md`](./GIT_PUSH_PLAN.md) for a day-by-day commit
plan — exactly which files to stage each day, plus ready-to-use commit
messages and descriptions, so the repository history reads as a
believable, steady build.

## Author

Built by -
Abir — DIU Cyber Security Centre.
Junaied - DIU Admission Office