
# 🏋️ AI-Powered Fitness Tracker

A full-stack, **microservices-based fitness tracking application** that leverages **Google Gemini AI** to analyze user workout activities and deliver personalized fitness recommendations — built with Java Spring Boot, Spring Cloud, RabbitMQ, and React.

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Microservices](#microservices)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Endpoints](#api-endpoints)

---

## 🧠 Overview

The AI-Powered Fitness Tracker allows users to log their fitness activities (runs, workouts, cycling, etc.) and instantly receive **AI-generated recommendations** covering:

- Overall performance analysis
- Pace & heart rate insights
- Calorie burn feedback
- Workout improvement suggestions
- Safety guidelines

Built using a **microservices architecture**, each service is independently deployable, scalable, and communicates via asynchronous messaging through **RabbitMQ**.

---

## 🏗️ Architecture

```
                        ┌─────────────────────────┐
                        │     React Frontend       │
                        │   (Vite + JavaScript)    │
                        └────────────┬────────────┘
                                     │ HTTP (JWT)
                        ┌────────────▼────────────┐
                        │      API Gateway         │
                        │  (Spring Cloud Gateway   │
                        │   + Keycloak OAuth2)     │
                        └──┬───────┬──────────┬───┘
                           │       │          │
              ┌────────────▼─┐  ┌──▼───────┐ ┌▼──────────────┐
              │ User Service │  │ Activity │ │   AI Service  │
              │  (PostgreSQL)│  │ Service  │ │   (MongoDB)   │
              └──────────────┘  │(MongoDB) │ └───────┬───────┘
                                └────┬─────┘         │
                                     │   RabbitMQ    │
                                     └───────────────┘
                        ┌────────────────────────────┐
                        │   Eureka Service Registry  │
                        └────────────────────────────┘
                        ┌────────────────────────────┐
                        │   Spring Cloud Config      │
                        │   Server                   │
                        └────────────────────────────┘
```

---

## 🔧 Microservices

### 1. 🔐 API Gateway (`gateway`)
- Entry point for all client requests
- Handles **OAuth2 / JWT authentication** via Keycloak
- Routes requests to appropriate downstream services
- Syncs authenticated Keycloak users with the User Service
- Built with **Spring WebFlux** (reactive, non-blocking)

### 2. 👤 User Service (`userservice`)
- Manages user registration and profile data
- Stores user data in **PostgreSQL** via JPA/Hibernate
- Exposed endpoints for user validation used by other services
- Supports roles: `USER`, `ADMIN`

### 3. 🏃 Activity Service (`activityservice`)
- Accepts and stores user fitness activity logs in **MongoDB**
- Validates the user via the User Service before saving
- Publishes activity events to **RabbitMQ** for AI processing
- Supports activity types: Running, Cycling, Swimming, etc.
- Tracks: duration, calories burned, heart rate, pace, additional metrics

### 4. 🤖 AI Service (`aiservice`)
- Listens to RabbitMQ for new activity events
- Sends structured prompts to **Google Gemini AI API**
- Parses AI JSON response into structured recommendation objects
- Stores recommendations in **MongoDB**
- Exposes REST endpoints to fetch per-user recommendations

### 5. ⚙️ Config Server (`configserver`)
- Centralized configuration management using **Spring Cloud Config**
- All microservices pull their configuration from this server at startup

### 6. 📋 Eureka Service Registry (`eureka`)
- Service discovery server using **Netflix Eureka**
- All microservices register here; Gateway uses it for routing

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Backend | Java 23, Spring Boot 3.4 |
| Microservices | Spring Cloud 2024, Eureka, Config Server |
| API Gateway | Spring Cloud Gateway (WebFlux) |
| Authentication | Keycloak, OAuth2, JWT |
| AI Integration | Google Gemini AI API |
| Messaging | RabbitMQ (AMQP) |
| Databases | PostgreSQL (User), MongoDB (Activity, AI) |
| ORM | Spring Data JPA, Spring Data MongoDB |
| Frontend | React, Vite, JavaScript |
| Utilities | Lombok, Jackson |
| Build Tool | Maven |

---

## ✨ Features

- ✅ **User Authentication** — Secure login/registration via Keycloak (OAuth2 + JWT)
- ✅ **Activity Logging** — Log workouts with type, duration, calories, heart rate, and custom metrics
- ✅ **AI Recommendations** — Real-time AI analysis of each activity using Google Gemini
- ✅ **Async Processing** — Activity events processed via RabbitMQ for non-blocking AI integration
- ✅ **Service Discovery** — Dynamic service registration and lookup via Eureka
- ✅ **Centralized Config** — All services share config via Spring Cloud Config Server
- ✅ **Polyglot Persistence** — PostgreSQL for relational user data, MongoDB for unstructured activity/AI data
- ✅ **React Frontend** — Clean UI to log activities and view AI-generated insights

---

## 📁 Project Structure

```
fitness-app-microservices/
│
├── gateway/                  # API Gateway + Keycloak Security
├── userservice/              # User registration & management (PostgreSQL)
├── activityservice/          # Fitness activity logging (MongoDB + RabbitMQ)
├── aiservice/                # Gemini AI recommendations (MongoDB + RabbitMQ)
├── configserver/             # Spring Cloud Config Server
├── eureka/                   # Netflix Eureka Service Registry
└── fitness-app-frontend/     # React + Vite Frontend
```

---

## 🚀 Getting Started

### Prerequisites

- Java 23+
- Maven
- Docker (for MongoDB, PostgreSQL, RabbitMQ, Keycloak)
- Node.js + npm (for frontend)

### 1. Start Infrastructure (Docker)

```bash
docker run -d --name mongodb -p 27017:27017 mongo
docker run -d --name postgres -e POSTGRES_PASSWORD=password -p 5432:5432 postgres
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
docker run -d --name keycloak -p 8080:8080 -e KEYCLOAK_ADMIN=admin -e KEYCLOAK_ADMIN_PASSWORD=admin quay.io/keycloak/keycloak:latest start-dev
```

### 2. Start Backend Services (in order)

```bash
# 1. Config Server
cd configserver && mvn spring-boot:run

# 2. Eureka Registry
cd eureka && mvn spring-boot:run

# 3. User Service
cd userservice && mvn spring-boot:run

# 4. Activity Service
cd activityservice && mvn spring-boot:run

# 5. AI Service
cd aiservice && mvn spring-boot:run

# 6. API Gateway
cd gateway && mvn spring-boot:run
```

### 3. Start Frontend

```bash
cd fitness-app-frontend
npm install
npm run dev
```

Frontend runs at: `http://localhost:5173`

---

## 🔑 Environment Variables

Set these in your `application.properties` or Config Server:

| Variable | Description |
|---|---|
| `gemini.api.url` | Google Gemini API base URL |
| `gemini.api.key` | Your Gemini API key |
| `spring.datasource.url` | PostgreSQL connection URL |
| `spring.data.mongodb.uri` | MongoDB connection URI |
| `spring.rabbitmq.host` | RabbitMQ host |
| `keycloak.auth-server-url` | Keycloak server URL |

---

## 📡 API Endpoints

### User Service
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/users/register` | Register a new user |
| GET | `/api/users/{id}` | Get user by ID |

### Activity Service
| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/activities` | Log a new activity |
| GET | `/api/activities/user/{userId}` | Get all activities for a user |

### AI Service
| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/recommendations/{activityId}` | Get AI recommendation for an activity |
| GET | `/api/recommendations/user/{userId}` | Get all recommendations for a user |

---

## 🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you'd like to change.

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).
