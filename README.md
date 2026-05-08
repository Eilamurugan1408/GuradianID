# GuardianID Tourist Safety Portal

GuardianID is a full-stack tourist safety platform with real-time SOS coordination, dual chat between tourist-authority-responder, WebRTC calling, geo-zone monitoring, eKYC-style onboarding, and AI chatbot support.

The project includes:
- A FastAPI backend with SQLite persistence and WebSocket messaging
- A Next.js frontend with role-based pages for tourist, authority/dashboard, and responder
- Supporting documentation for architecture, testing, and integration flows

## Key Features

- Tourist onboarding with profile registration and QR identity card generation
- SOS incident lifecycle: NEW -> CONFIRMED -> ASSIGNED -> RESOLVED
- Two-way and dual-thread chat flows for live emergency coordination
- Group room chat for tourist travel groups
- Optional WebRTC call controls in chat interfaces
- Geo-zone definitions and zone-based safety logic
- AI travel assistant/chatbot endpoints with session context

## Tech Stack

- Frontend: Next.js 15, React 18, TypeScript, Tailwind, Radix UI
- Backend: FastAPI, SQLAlchemy, SQLite, WebSockets
- AI/External: Groq API (optional), OpenWeather API (optional)

## Repository Structure

- backend: FastAPI app, APIs, DB models, websocket logic
- frontend: Next.js application
- docs: architecture, troubleshooting, testing, and integration guides
- test_chat_flow.py: flow test script

## Prerequisites

- Python 3.10+
- Node.js 18+
- npm or pnpm

## Quick Start

### 1) Clone

git clone https://github.com/Eilamurugan1408/GuradianID.git
cd GuradianID

### 2) Backend Setup

Open Terminal 1:

cd backend

Create and activate a virtual environment (recommended):

Windows PowerShell:
python -m venv .venv
.\.venv\Scripts\Activate.ps1

Install backend dependencies (if you do not yet have a requirements file):

pip install fastapi "uvicorn[standard]" sqlalchemy python-dotenv requests pydantic

Create backend/.env and set optional keys:

GROQ_API_KEY=your_key_here
OPENWEATHER_API_KEY=your_key_here

Run backend:

python main.py

Default backend URL: http://localhost:8000

### 3) Frontend Setup

Open Terminal 2:

cd frontend
npm install
npm run dev

If you prefer pnpm:

pnpm install
pnpm dev

Default frontend URL: http://localhost:3000

## Primary App Routes

- /tourist
- /dashboard
- /responder
- /admin
- /overview
- /pricing

## Typical Verification Flow

1. Open /tourist and register a tourist profile
2. Trigger an SOS event
3. Open /dashboard and confirm the incident
4. Open /responder and accept the confirmed ticket
5. Verify responder-tourist and authority-responder chat updates


## Notes

- The backend uses a local SQLite DB file (guardianid.db).
- The backend auto-loads environment variables from backend/.env.
- If ports 3000 or 8000 are already in use, stop the conflicting process or change port settings.

## Git Workflow

After making changes:

git add .
git commit -m "Update README and project docs"
git push
