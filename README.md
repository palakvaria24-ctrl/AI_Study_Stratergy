# AI Study Strategy Generator

A full-stack study planning app that tracks study sessions per subject, calculates efficiency analytics, and generates AI-powered study strategies. It combines a **Node.js/Express + MongoDB** backend, a **vanilla HTML/CSS/JS** frontend, and a **Python (Flask) microservice** that uses a small scikit-learn regression model plus Google's Gemini API to analyze syllabi and recommend study strategies.

## Features

- 🔐 **Authentication** — register/login with hashed passwords (bcrypt) and JWT-based session tokens
- 📚 **Subjects & Study Logs** — add subjects (with difficulty, exam date, topic count) and log planned vs. actual study hours
- 📊 **Analytics Dashboard** — efficiency score, backlog count, and status (Good / Average / Needs Improvement) computed from study logs
- 🧠 **AI Insights** — a Flask microservice predicts study efficiency with a linear regression model, flags weak subjects, and calls the Gemini API to analyze syllabus text and suggest a personalized strategy
- 🖥️ **Frontend Pages** — login, register, dashboard, analytics, subjects, and AI insights pages served as static HTML/CSS/JS

## Architecture

```
Browser (frontend/*.html, script.js)
        │
        ▼
Express API (server.js, routes/, controllers/)  ──►  MongoDB (Mongoose models)
        │
        ▼
Flask AI service (ai-service/app.py, model.py)  ──►  Gemini API
```

The Express server exposes REST endpoints under `/api/auth`, `/api/study`, and `/api/analytics`, and also serves the static frontend. The analytics controller calls out to a separate Python service (`ai-service/app.py`, expected on `http://localhost:5001`) for ML predictions and AI-generated syllabus insights.

## Tech Stack

**Backend:** Node.js, Express 5, Mongoose/MongoDB, JWT, bcryptjs
**AI Service:** Python, Flask, scikit-learn, pandas, google-generativeai (Gemini)
**Frontend:** HTML, CSS, vanilla JavaScript

## Project Structure

```
.
├── server.js                  # Express app entry point
├── config/db.js               # MongoDB connection helper
├── models/                    # Mongoose schemas (User, Subject, StudyLog)
├── routes/                    # Express route definitions
├── controllers/                # Route handlers (auth, study logs, subjects, analytics, AI)
├── middleware/authMiddleware.js  # JWT verification middleware
├── utils/response.js          # Standardized JSON response helper
├── frontend/                  # Static pages (login, register, dashboard, analytics, insights, subjects)
└── ai-service/
    ├── app.py                 # Flask app: /predict endpoint, Gemini syllabus analysis
    └── model.py                # Linear regression model for study efficiency prediction
```

## Prerequisites

- Node.js (v18+ recommended) and npm
- Python 3.9+
- A MongoDB connection string (e.g. MongoDB Atlas)
- A Google Gemini API key

## Setup

### 1. Clone and install Node dependencies

```bash
npm install
```

### 2. Install Python dependencies for the AI service

```bash
cd ai-service
pip install flask flask-cors python-dotenv google-generativeai pandas scikit-learn
```

> Consider adding a `requirements.txt` to pin these versions.

### 3. Configure environment variables

Create a `.env` file in the project root (this file is **not** committed to version control):

```env
MONGO_URI=your_mongodb_connection_string
JWT_SECRET=your_jwt_signing_secret
GEMINI_API_KEY=your_gemini_api_key
PORT=5000
```

The `ai-service` reads `GEMINI_API_KEY` from the same or its own `.env` file via `python-dotenv`.

### 4. Run the services

Start the Express API and frontend:

```bash
npm run dev   # if using nodemon, or:
node server.js
```

In a separate terminal, start the AI microservice:

```bash
cd ai-service
python app.py
```

The Express server runs on `http://localhost:5000` by default, and the Flask AI service runs on `http://localhost:5001`.

### 5. Open the app

Visit `http://localhost:5000` in your browser to reach the login page.

## API Overview

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| POST | `/api/auth/register` | Create a new user | No |
| POST | `/api/auth/login` | Log in and receive a JWT | No |
| POST | `/api/study/add` | Log a study session | Yes |
| DELETE | `/api/study/clear` | Clear all study logs for the user | Yes |
| GET | `/api/analytics` | Get efficiency/backlog/status for the user | Yes |
| GET | `/api/ai/generate` | Generate an AI study plan from subjects | No |

Protected routes require an `Authorization: Bearer <token>` header with the JWT returned from login.

The Flask service exposes:

| Method | Endpoint | Description |
|---     |---|---                  |
| GET    | `/`      | Health check |
| POST   | `/predict`| Returns predicted efficiency, weak subjects, per-subject advice, and Gemini-generated syllabus insights |

## ⚠️ Security Notes

- **Rotate any credentials that were ever committed to this repo.** A `.env` file with a live MongoDB URI, JWT secret, and Gemini API key was found in this project — treat those as compromised and issue new ones.
- Add a `.gitignore` that excludes `.env`, `node_modules/`, and `__pycache__/` before pushing to GitHub.
- Several routes (e.g. `/api/ai/generate`, `GET /api/subjects`) are not protected by the auth middleware — review which endpoints should require a valid JWT.
- Debug `console.log` statements print auth headers and decoded tokens; remove these before deploying to production.
- The Flask app runs with `debug=True`; disable this in production.

## Roadmap Ideas

- Add a `requirements.txt` / `Pipfile` for the AI service
- Add input validation and rate limiting on public endpoints
- Containerize both services (Docker Compose) for easier local setup
- Add automated tests for controllers and the ML prediction endpoint

## License

ISC (as declared in `package.json`) — update as appropriate for your use case.
