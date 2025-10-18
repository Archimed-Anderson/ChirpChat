# ChirpLocal

> **A complete local microblogging platform for Free Mobile users**

ChirpLocal is a Twitter-style microblogging application that allows users to post short messages, including complaints that automatically generate unique contact form links. Built with modern web technologies and fully containerized for easy local deployment.

![Version](https://img.shields.io/badge/version-1.0.0-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

## 🌟 Features

- **Microblogging Platform**: Post short "chirps" (tweets) up to 500 characters
- **Complaint System**: Special post type that generates unique complaint links
- **Auto-generated Forms**: Each complaint gets a unique URL with a contact form
- **External API Integration**: Complaint forms forward data to configurable contact API
- **Real-time Updates**: WebSocket support for live timeline updates
- **Authentication**: JWT-based signup/login system
- **Social Features**: Like posts, add comments
- **Responsive UI**: Twitter/X-inspired clean design
- **Comprehensive Logging**: Structured JSON logs for monitoring
- **Fully Dockerized**: Run entire stack with one command

## 📋 Table of Contents

- [Quick Start](#quick-start)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Installation](#installation)
- [Configuration](#configuration)
- [API Documentation](#api-documentation)
- [Testing](#testing)
- [Project Structure](#project-structure)
- [Development](#development)

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- Git

### Running ChirpLocal

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd ChirpLocal
   ```

2. **Start the application**
   ```bash
   # Windows
   dev.bat up

   # Linux/Mac
   chmod +x dev
   ./dev up
   ```

3. **Access the application**
   - Frontend: http://localhost:5173
   - Backend API: http://localhost:8000
   - API Docs: http://localhost:8000/docs

4. **Seed demo data** (optional, in a new terminal)
   ```bash
   # Windows
   dev.bat seed

   # Linux/Mac
   ./dev seed
   ```

5. **Login with demo credentials**
   - Username: any username from seed data (e.g., `jeanmartin0`)
   - Password: `password123`

## 🏗️ Architecture

ChirpLocal uses a modern microservices architecture:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│                 │     │                 │     │                 │
│  React/Vite     │────▶│  FastAPI        │────▶│  PostgreSQL     │
│  Frontend       │     │  Backend        │     │  Database       │
│                 │     │                 │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                         │
        │                       │                         │
        └───────────────────────┴─────────────────────────┘
                               │
                               ▼
                      ┌─────────────────┐
                      │                 │
                      │  Redis          │
                      │  (WebSocket)    │
                      │                 │
                      └─────────────────┘
```

### Key Components

- **Frontend**: React 18 + TypeScript + Vite
- **Backend**: FastAPI (Python 3.11)
- **Database**: PostgreSQL 15
- **Cache/PubSub**: Redis 7
- **WebSocket**: Native FastAPI WebSocket support

## 🛠️ Tech Stack

### Backend
- **FastAPI**: Modern Python web framework
- **SQLAlchemy**: ORM for database operations
- **Pydantic**: Data validation
- **Python-JOSE**: JWT token handling
- **Passlib**: Password hashing
- **HTTPX**: Async HTTP client for external API calls
- **Uvicorn**: ASGI server

### Frontend
- **React 18**: UI library
- **TypeScript**: Type-safe JavaScript
- **Vite**: Fast build tool
- **React Router**: Client-side routing
- **Zustand**: State management
- **Axios**: HTTP client
- **date-fns**: Date formatting

### Infrastructure
- **Docker**: Containerization
- **Docker Compose**: Multi-container orchestration
- **PostgreSQL**: Relational database
- **Redis**: WebSocket pub/sub and caching

## 📥 Installation

### Method 1: Docker (Recommended)

```bash
# Clone repository
git clone <repository-url>
cd ChirpLocal

# Copy environment file
cp .env.example .env

# Edit .env with your configuration (optional)
# nano .env  # or your preferred editor

# Start all services
docker-compose up --build

# In another terminal, seed demo data
docker-compose exec backend python -m app.scripts.seed_data
```

### Method 2: Local Development

**Backend Setup:**
```bash
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set environment variables
cp ../.env.example .env

# Run database migrations
alembic upgrade head  # If using Alembic

# Seed data
python -m app.scripts.seed_data

# Start server
uvicorn app.main:app --reload
```

**Frontend Setup:**
```bash
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```

## ⚙️ Configuration

### Environment Variables

Edit `.env` file to configure the application:

```bash
# Application
APP_NAME=ChirpLocal
ENVIRONMENT=development

# Backend
BACKEND_PORT=8000
SECRET_KEY=your-secret-key-change-this  # CHANGE THIS!
ACCESS_TOKEN_EXPIRE_MINUTES=30
REFRESH_TOKEN_EXPIRE_DAYS=7

# Database
POSTGRES_USER=chirplocal
POSTGRES_PASSWORD=chirplocal_password  # CHANGE THIS!
POSTGRES_DB=chirplocal_db
DATABASE_URL=postgresql://chirplocal:chirplocal_password@postgres:5432/chirplocal_db

# External Contact API
CONTACT_API_URL=http://localhost:5050/api/contact  # Configure your endpoint

# CORS
CORS_ORIGINS=http://localhost:5173,http://localhost:3000

# Logging
LOG_LEVEL=INFO
LOG_FORMAT=json
```

## 📚 API Documentation

### Authentication Endpoints

#### POST `/auth/signup`
Register a new user.

**Request:**
```bash
curl -X POST http://localhost:8000/auth/signup \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "email": "test@example.com",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "id": 1,
  "username": "testuser",
  "email": "test@example.com",
  "avatar_url": "/avatars/default.png",
  "created_at": "2025-10-17T10:30:00Z"
}
```

#### POST `/auth/login`
Authenticate and receive tokens.

**Request:**
```bash
curl -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "password123"
  }'
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer"
}
```

### Posts Endpoints

#### GET `/timeline`
Get paginated timeline of posts.

**Request:**
```bash
curl -X GET "http://localhost:8000/timeline?page=1&page_size=20" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

#### POST `/posts/`
Create a new post (normal or complaint).

**Request:**
```bash
curl -X POST http://localhost:8000/posts/ \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Network issues in my area!",
    "type": "complaint"
  }'
```

**Response (Complaint):**
```json
{
  "id": 42,
  "user_id": 1,
  "content": "Network issues in my area!",
  "type": "complaint",
  "link_uuid": "a1b2c3d4-e5f6-4789-a012-b3c4d5e6f789",
  "created_at": "2025-10-17T10:45:00Z",
  "user": {...},
  "comments_count": 0,
  "likes_count": 0,
  "is_liked": false
}
```

### Complaint Endpoints

#### GET `/complaints/{link_uuid}`
Get complaint details by UUID.

**Request:**
```bash
curl -X GET http://localhost:8000/complaints/a1b2c3d4-e5f6-4789-a012-b3c4d5e6f789
```

#### POST `/complaints/{link_uuid}/submit`
Submit complaint form data.

**Request:**
```bash
curl -X POST http://localhost:8000/complaints/a1b2c3d4-e5f6-4789-a012-b3c4d5e6f789/submit \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "Jean",
    "last_name": "Dupont",
    "phone_number": "0601020304"
  }'
```

### WebSocket Connection

**Connect:**
```javascript
const ws = new WebSocket('ws://localhost:8000/ws?token=YOUR_ACCESS_TOKEN');

ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log('Received:', message);
};
```

For complete API documentation, visit: http://localhost:8000/docs

## 🧪 Testing

### End-to-End Complaint Flow Test

1. **Create a complaint post:**
   - Login to the application
   - Create a new post with type "Complaint"
   - Note the generated link UUID

2. **Access the complaint form:**
   - Open the generated link in a new tab: `http://localhost:5173/complaint/{UUID}`
   - Fill in the form with test data

3. **Verify API forwarding:**
   - Check logs: `docker-compose logs backend | grep contact`
   - Verify request was sent to external contact API

### Check Logs

```bash
# Windows
dev.bat logs backend

# Linux/Mac
./dev logs backend

# View contact API logs specifically
docker-compose logs backend | findstr contact  # Windows
docker-compose logs backend | grep contact     # Linux/Mac
```

### Health Check

```bash
curl http://localhost:8000/health
```

**Expected Response:**
```json
{
  "status": "healthy",
  "timestamp": "2025-10-17T10:30:00Z",
  "version": "1.0.0",
  "service": "ChirpLocal API"
}
```

## 📁 Project Structure

```
ChirpLocal/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI application
│   │   ├── config.py            # Configuration
│   │   ├── database.py          # Database setup
│   │   ├── models.py            # SQLAlchemy models
│   │   ├── schemas.py           # Pydantic schemas
│   │   ├── auth.py              # Authentication utils
│   │   ├── logger.py            # Logging configuration
│   │   ├── utils.py             # Utility functions
│   │   ├── dependencies.py      # FastAPI dependencies
│   │   ├── websocket.py         # WebSocket manager
│   │   ├── routers/             # API routes
│   │   │   ├── auth.py
│   │   │   ├── posts.py
│   │   │   ├── comments.py
│   │   │   ├── likes.py
│   │   │   ├── users.py
│   │   │   ├── complaints.py
│   │   │   └── timeline.py
│   │   └── scripts/
│   │       └── seed_data.py     # Database seeding
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/
│   ├── src/
│   │   ├── main.tsx             # Entry point
│   │   ├── App.tsx              # Main app component
│   │   ├── index.css            # Global styles
│   │   ├── types.ts             # TypeScript types
│   │   ├── api.ts               # API client
│   │   ├── store.ts             # State management
│   │   ├── websocket.ts         # WebSocket manager
│   │   ├── components/
│   │   │   ├── Layout.tsx
│   │   │   ├── ProtectedRoute.tsx
│   │   │   ├── CreatePost.tsx
│   │   │   └── PostItem.tsx
│   │   └── pages/
│   │       ├── Login.tsx
│   │       ├── Signup.tsx
│   │       ├── Timeline.tsx
│   │       └── ComplaintForm.tsx
│   ├── Dockerfile
│   ├── package.json
│   └── vite.config.ts
├── logs/                        # Application logs
├── docker-compose.yaml
├── .env.example
├── .gitignore
├── dev                          # Development helper (Linux/Mac)
├── dev.bat                      # Development helper (Windows)
├── README.md
├── ARCHITECTURE.md
└── contracts.md
```

## 💻 Development

### Available Commands

```bash
# Start all services
dev up  # or dev.bat up on Windows

# Stop all services
dev down

# Clean up (remove volumes)
dev clean

# View logs
dev logs [service]

# Seed database
dev seed

# Open shell in service
dev shell [service]

# Run tests
dev test
```

### Adding New Features

1. **Backend**: Add routes in `backend/app/routers/`
2. **Frontend**: Add components in `frontend/src/components/` or pages in `frontend/src/pages/`
3. **Database**: Update models in `backend/app/models.py`
4. **Types**: Update schemas in `backend/app/schemas.py` and `frontend/src/types.ts`

## 📝 Example Log Output

The system generates structured JSON logs. Example entries:

```json
{"timestamp": "2025-10-17T10:30:15.123Z", "level": "INFO", "logger": "chirplocal.app", "message": "New user registered", "user_id": 42, "username": "testuser"}
{"timestamp": "2025-10-17T10:30:45.456Z", "level": "INFO", "logger": "chirplocal.contact", "message": "Complaint form submitted successfully", "link_uuid": "a1b2c3d4...", "post_id": 100}
{"timestamp": "2025-10-17T10:31:00.789Z", "level": "INFO", "logger": "chirplocal.access", "message": "HTTP request", "method": "POST", "path": "/posts/", "status_code": 201}
```

## 🐛 Troubleshooting

### Port Already in Use

```bash
# Change ports in .env file
BACKEND_PORT=8001
# Then restart: dev down && dev up
```

### Database Connection Issues

```bash
# Reset database
dev clean
dev up
```

### Frontend Can't Connect to Backend

Check CORS settings in `.env`:
```bash
CORS_ORIGINS=http://localhost:5173
```

## 📄 License

MIT License - See LICENSE file for details

## 👥 Contributors

Built for Free Mobile user community feedback and support.

## 📧 Contact

For issues or questions, please use the GitHub issues page.

---

**Made with ❤️ for Free Mobile users**
#   C h i r p C h a t  
 