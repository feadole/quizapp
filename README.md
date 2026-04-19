# ⚡ QuizApp — Real-Time Quiz Platform

A full-stack real-time quiz application (Kahoot-style) built with **Next.js 14**, **Node.js + Express**, **Socket.IO**, and **PostgreSQL via Prisma**.

---

## Architecture

```
quizapp/
├── backend/          Node.js + Express + Socket.IO
│   ├── routes/       REST API (auth, quizzes, sessions)
│   ├── socket/       Real-time game logic
│   ├── middleware/   JWT authentication
│   └── prisma/       Database schema
└── frontend/         Next.js 14 (App Router)
    ├── app/          Pages and routes
    ├── lib/          Socket.IO client + API helper
    └── styles/       Global CSS (Electric Arcade theme)
```

---

## Prerequisites

- Node.js 18+
- PostgreSQL 14+
- npm or yarn

---

## Setup

### 1. Backend

```bash
cd backend

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env with your DATABASE_URL and JWT_SECRET

# Generate Prisma client and run migrations
npx prisma generate
npx prisma migrate dev --name init

# Start server
node server.js        # production
# or
npm run dev           # with nodemon
```

Backend runs on `http://localhost:4000`

### 2. Frontend

```bash
cd frontend

# Install dependencies
npm install

# Configure environment
cp .env.local.example .env.local
# Edit NEXT_PUBLIC_BACKEND_URL if backend runs on different port

# Start dev server
npm run dev
```

Frontend runs on `http://localhost:3000`

---

## API Reference

### Authentication
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/auth/register` | Register (role: PARTICIPANT or ORGANIZER) |
| POST | `/api/auth/login` | Login, returns JWT |
| GET | `/api/auth/me` | Current user info |

### Quizzes
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/quizzes` | List quizzes (`?mine=true` for own) |
| GET | `/api/quizzes/:id` | Get quiz with questions |
| POST | `/api/quizzes` | Create quiz (organizer only) |
| PUT | `/api/quizzes/:id` | Update quiz |
| DELETE | `/api/quizzes/:id` | Delete quiz |

### Sessions
| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/sessions` | Create session, returns roomCode |
| GET | `/api/sessions/room/:code` | Look up room by code |
| GET | `/api/sessions/:id/leaderboard` | Final leaderboard |
| GET | `/api/sessions/history/me` | User's session history |

---

## Socket.IO Events

### Client → Server
| Event | Payload | Description |
|-------|---------|-------------|
| `join_room` | `{ roomCode, userId }` | Join a session |
| `start_quiz` | `{ sessionId, roomCode }` | Organizer starts quiz |
| `submit_answer` | `{ participantId, questionId, selectedOptions, sessionId, questionIndex }` | Submit answer |
| `next_question` | `{ sessionId, roomCode }` | Organizer advances |

### Server → Client
| Event | Payload | Description |
|-------|---------|-------------|
| `room_joined` | `{ sessionId, quizTitle, participants, status }` | Joined successfully |
| `participant_joined` | `{ userId, participants }` | Someone joined |
| `quiz_started` | `{ question, questionIndex }` | First question |
| `question_shown` | `{ question, questionIndex }` | Next question |
| `timer_tick` | `{ timeLeft }` | Every second |
| `answer_result` | `{ isCorrect, pointsEarned }` | Personal result |
| `answer_progress` | `{ answered, total }` | Answer count |
| `question_ended` | `{ correctOptions, leaderboard }` | After timeout/all answered |
| `quiz_ended` | `{ leaderboard }` | Final standings |

---

## Deployment

### Backend (Railway / Render)
```bash
# Set environment variables in dashboard:
DATABASE_URL=postgresql://...
JWT_SECRET=...
FRONTEND_URL=https://your-frontend.vercel.app
PORT=4000
```

### Frontend (Vercel)
```bash
# Set environment variables:
NEXT_PUBLIC_BACKEND_URL=https://your-backend.railway.app
```

---

## Scoring Algorithm

Points are awarded only for correct answers. The score is weighted by response speed:

```
points_earned = base_points × (0.3 + 0.7 × (1 - time_taken / time_limit))
```

- Fastest correct answer: ~100% of base points
- Slowest correct answer: 30% of base points
- Wrong answer: 0 points

---

## Tech Stack Justification

| Choice | Reason |
|--------|--------|
| **Next.js 14** | SSR + App Router + API routes in one framework; excellent DX |
| **Socket.IO** | Built-in rooms, reconnection, fallback transports; battle-tested |
| **PostgreSQL + Prisma** | Relational model fits quiz data; Prisma gives type-safe queries |
| **JWT** | Stateless auth; works across separate backend instances |
| **Vercel + Railway** | Zero-config deploy; generous free tiers for MVP |
