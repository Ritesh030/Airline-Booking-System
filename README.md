# ✈️ Airline Booking System

A **production-style backend** built with a **microservices architecture** using Node.js. Each service is independently deployable, owns its own database, and communicates over REST APIs.

![Node.js](https://img.shields.io/badge/Node.js-339933?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Express.js](https://img.shields.io/badge/Express.js-000000?style=for-the-badge&logo=express&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Sequelize](https://img.shields.io/badge/Sequelize-52B0E7?style=for-the-badge&logo=Sequelize&logoColor=white)
![JWT](https://img.shields.io/badge/JWT-000000?style=for-the-badge&logo=JSON%20web%20tokens&logoColor=white)

---

## 🏗️ Architecture Overview

```
                        ┌─────────────────────────────────────────┐
                        │           CLIENT  (REST API)            │
                        └────┬──────────┬──────────┬─────────┬───┘
                             │          │          │         │
                             ▼          ▼          ▼         ▼
                     ┌──────────┐ ┌──────────┐ ┌───────┐ ┌────────┐
                     │  Auth    │ │ Flights  │ │Booking│ │Reminder│
                     │ Service  │ │& Search  │ │Service│ │Service │
                     │ :3001    │ │ Service  │ │ :3002 │ │ :3003  │
                     │          │ │  :3000   │ │       │ │        │
                     └────┬─────┘ └────┬─────┘ └───┬───┘ └───┬────┘
                          │            │            │         │
                          ▼            ▼            │         ▼
                       MySQL DB     MySQL DB        │      Nodemailer
                    (users/roles) (flights/airports)│      (Gmail)
                                                    │
                                    ┌───────────────┘
                                    │  calls Flight Service
                                    │  via Axios (REST)
                                    ▼
                                 MySQL DB
                               (bookings)
```

> **BookingService** communicates with **FlightsAndSearch** over HTTP using Axios to validate seats and update availability in real time.

---

## 📦 Services

### 1. 🔐 [Auth Service](https://github.com/Ritesh030/Auth-Service)
Handles all user identity — registration, login, and access control.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/signup` | Register a new user |
| POST | `/api/v1/signin` | Login and receive JWT token |
| GET | `/api/v1/isAuthenticated` | Validate a JWT token |
| GET | `/api/v1/isAdmin` | Check if user has admin role |

**Key Highlights:**
- Passwords hashed using **bcrypt** before storing
- **JWT** tokens issued on login for stateless auth
- User ↔ Role **many-to-many** relationship (User_Roles join table)
- Role seeder included for quick setup (`admin`, `customer`)

---

### 2. 🛫 [Airline Flight & Search Service](https://github.com/Ritesh030/Airline-Flight-And-Search-Service)
Master data service — manages all airline infrastructure and flight inventory.

| Route Group | Description |
|-------------|-------------|
| `/api/v1/city` | Add and manage cities |
| `/api/v1/airport` | Add airports, link to cities |
| `/api/v1/airplane` | Register airplanes with capacity |
| `/api/v1/flight` | Create, search, fetch, and update flights |

**Key Highlights:**
- **Layered architecture** — Controllers → Services → Repositories
- Flight seat count automatically initialized from airplane capacity
- Supports `PATCH /flight/:id` to update available seats after booking
- Seeder included to pre-populate airplane data

**Database Models:** `City` → `Airport` → `Airplane` → `Flights`

---

### 3. 🎟️ [Airline Booking Service](https://github.com/Ritesh030/Airline-Booking-Service)
Handles the entire booking lifecycle with real-time flight validation.

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/v1/booking` | Create a new booking |

**Booking Flow:**
```
1. Receive booking request (userId, flightId, totalSeats)
        ↓
2. Call FlightsAndSearch to fetch flight details
        ↓
3. Validate seat availability
        ↓
4. Calculate total price (seats × price per seat)
        ↓
5. Create booking with status: "InProcess"
        ↓
6. Update remaining seats in FlightsAndSearch
        ↓
7. Mark booking as status: "Booked" ✅
```

**Booking Status:** `InProcess` → `Booked` | `Cancelled`

---

### 4. 📧 [Reminder Service](https://github.com/Ritesh030/Reminder-Service)
Email notification service for sending booking reminders to users.

- Email transport configured via **Nodemailer + Gmail**
- Designed as a standalone service for easy extension
- Can be triggered by other services via REST

---

## 🛠️ Tech Stack

| Technology | Purpose |
|------------|---------|
| **Node.js** | Runtime environment |
| **Express.js** | REST API framework |
| **MySQL** | Relational database (one per service) |
| **Sequelize ORM** | Database modeling & migrations |
| **JWT** | Stateless authentication |
| **bcrypt** | Password hashing |
| **Axios** | Inter-service HTTP communication |
| **Nodemailer** | Email delivery |
| **dotenv** | Environment-based configuration |
| **Nodemon** | Auto-restart during development |

---

## 🚀 Getting Started

### Prerequisites
- Node.js (v16+)
- MySQL
- npm

### 1. Clone with submodules
```bash
git clone --recurse-submodules https://github.com/Ritesh030/Airline-Booking-System.git
cd Airline-Booking-System
```

### 2. Install dependencies (in each service folder)
```bash
npm install
```

### 3. Set up environment variables

Create a `.env` file inside each service folder:

**Auth Service** (port 3001)
```env
PORT=3001
JWT_KEY=your_long_random_jwt_secret
DB_SYNC=false
```

**Airline Flight & Search Service** (port 3000)
```env
PORT=3000
SYNC_DB=false
```

**Airline Booking Service** (port 3002)
```env
PORT=3002
DB_SYNC=false
FLIGHT_SERVICE_PATH=http://localhost:3000
```

**Reminder Service** (port 3003)
```env
PORT=3003
EMAIL_ID=your_email@gmail.com
EMAIL_PASS=your_gmail_app_password
```

### 4. Configure Sequelize (each service)

Add `src/config/config.json` in each service:
```json
{
  "development": {
    "username": "root",
    "password": "your_mysql_password",
    "database": "your_db_name",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

### 5. Run migrations & seeders
```bash
# In each service with migrations:
npx sequelize-cli db:migrate

# In Auth Service only (to seed roles):
npx sequelize-cli db:seed:all

# In FlightsAndSearch only (to seed airplanes):
npx sequelize-cli db:seed:all
```

### 6. Start services (in this order)

```bash
# Terminal 1 — Start first (BookingService depends on this)
cd Airline-Flight-And-Search-Service && npm start

# Terminal 2
cd Auth-Service && npm start

# Terminal 3
cd Airline-Booking-Service && npm start

# Terminal 4
cd Reminder-Service && npm start
```

---

## 📁 Folder Structure

Each service follows the same clean layered pattern:

```
ServiceName/
├── src/
│   ├── config/          # Environment & DB config
│   ├── controllers/     # Request handlers
│   ├── services/        # Business logic
│   ├── repository/      # Database queries (Sequelize)
│   ├── models/          # Sequelize models
│   ├── routes/          # Express route definitions
│   │   └── v1/          # Versioned API routes
│   ├── middlewares/     # Auth validators
│   ├── migrations/      # Sequelize DB migrations
│   ├── seeders/         # Seed data
│   └── utils/           # Error classes & helpers
├── .env.example
├── package.json
└── README.md
```

---

## 🔗 Individual Service Repositories

| Service | Repository |
|---------|------------|
| Auth Service | [Ritesh030/Auth-Service](https://github.com/Ritesh030/Auth-Service) |
| Flight & Search | [Ritesh030/Airline-Flight-And-Search-Service](https://github.com/Ritesh030/Airline-Flight-And-Search-Service) |
| Booking Service | [Ritesh030/Airline-Booking-Service](https://github.com/Ritesh030/Airline-Booking-Service) |
| Reminder Service | [Ritesh030/Reminder-Service](https://github.com/Ritesh030/Reminder-Service) |