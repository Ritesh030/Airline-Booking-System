# Airline Management System

This project is a Node.js microservices-based backend for an airline management platform. It is organized as separate services for authentication, flight and search operations, bookings, and reminders so each domain can evolve independently.

The codebase uses Express for APIs, Sequelize for database access, MySQL for persistence, and environment-based configuration for local development.

## About The Project

The goal of this project is to provide the backend building blocks needed for an airline platform, including:

- user authentication and role checks
- flight, airport, city, and airplane management
- flight search and seat-aware booking creation
- reminder and email service support

The repository follows a service-oriented structure, where each service owns its own routes, controllers, models, and configuration.

## Main Features

- User signup and signin with hashed passwords
- JWT-based authentication validation
- Admin-role verification support
- CRUD APIs for cities and airports
- Airplane creation support
- Flight creation, fetching, updating, and search APIs
- Automatic flight seat initialization from airplane capacity
- Booking creation with live flight validation
- Seat availability checks before confirming bookings
- Automatic total fare calculation during booking
- Email reminder capability through Nodemailer and Gmail

## Services

### 1. `Auth_Service`

Handles authentication and authorization-related flows.

Key features:

- create user accounts
- sign in users
- verify JWT tokens
- check admin access

Important routes:

- `POST /api/v1/signup`
- `POST /api/v1/signin`
- `GET /api/v1/isAuthenticated`
- `GET /api/v1/isAdmin`

### 2. `FlightsAndSearch`

Handles master data and flight management.

Key features:

- manage cities
- manage airports
- create airplanes
- create and update flights
- fetch single flights and search multiple flights

Important route groups:

- `/api/v1/city`
- `/api/v1/airport`
- `/api/v1/airplane`
- `/api/v1/flight`

### 3. `BookingService`

Handles booking creation by integrating with the flight service.

Key features:

- fetch flight details before booking
- validate available seats
- calculate total booking price
- update remaining seats after booking
- mark booking status as `Booked`

Important route:

- `POST /api/v1/booking`

### 4. `ReminderService`

Provides the foundation for sending reminder emails.

Key features:

- email transport setup with Nodemailer
- Gmail-based sender configuration
- reusable email sending service

## Project Structure

```text
AirLine Managment/
|-- Auth_Service/
|-- BookingService/
|-- FlightsAndSearch/
|-- ReminderService/
`-- README.md
```

Each service contains its own `src/` folder with controllers, routes, models, config, and service logic.

## Tech Stack

- Node.js
- Express.js
- MySQL
- Sequelize
- JWT
- bcrypt
- Axios
- Nodemailer

## Local Setup

### Prerequisites

- Node.js
- npm
- MySQL

### 1. Install dependencies

Run this inside each service folder:

```bash
npm install
```

### 2. Configure environment variables

Each service expects its own `.env` file.

Common variables used in this repository:

#### `Auth_Service`

```env
PORT=3001
JWT_KEY=your_jwt_secret
DB_SYNC=false
```

#### `FlightsAndSearch`

```env
PORT=3000
SYNC_DB=false
```

#### `BookingService`

```env
PORT=3002
DB_SYNC=false
FLIGHT_SERVICE_PATH=http://localhost:3000
```

#### `ReminderService`

```env
PORT=3003
EMAIL_ID=your_email@gmail.com
EMAIL_PASS=your_app_password
```

### 3. Configure Sequelize database connection

Each service that uses Sequelize also expects a local `src/config/config.json` file for database credentials.

Example:

```json
{
  "development": {
    "username": "root",
    "password": "your_mysql_password",
    "database": "service_database_name",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
}
```

### 4. Run migrations and seeders

Run these in the service folder where migrations exist:

```bash
npx sequelize-cli db:migrate
```

For the auth service, seed roles as well:

```bash
npx sequelize-cli db:seed:all
```

### 5. Start the services

Run this inside each service directory:

```bash
npm start
```

Suggested local startup order:

1. `FlightsAndSearch`
2. `Auth_Service`
3. `BookingService`
4. `ReminderService`

## How The Services Work Together

- `Auth_Service` manages user login and access validation.
- `FlightsAndSearch` stores airline, airport, city, airplane, and flight data.
- `BookingService` calls `FlightsAndSearch` to verify a flight and update seat counts during booking.
- `ReminderService` can be used to send notification or reminder emails.

## Current Scope

This repository currently focuses on the backend service layer. It is well suited for learning and extending concepts such as:

- microservice-style backend design
- REST API development
- Sequelize model management
- service-to-service communication
- authentication with JWT

## Notes

- The repository name uses `AirLine Managment` as the project folder name.
- Some services already include their own service-level README files for deeper detail.
- `BookingService` depends on `FlightsAndSearch` being available at the configured `FLIGHT_SERVICE_PATH`.