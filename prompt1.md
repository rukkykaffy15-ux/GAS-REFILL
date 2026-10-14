# GasTrack Backend Service — System Architecture & Folder Structure

## 📌 Overview & Core Entities

This document outlines the standard, production-ready directory structure for the **GasTrack Backend REST API**, built with FastAPI for an on-demand cooking gas refill and delivery service with consumption tracking.

### Core Domain Entities
1. **Gas Station:** Name, address, coordinates (latitude/longitude), price per kg, available cylinder sizes.
2. **Cylinder / Refill Log:** Cylinder size in kg, refill date, user ID, cost.
3. **Delivery Order:** User ID, station ID, driver ID, pickup address, delivery status, timestamp.
4. **User Consumption Profile:** Household size, cooking frequency per day, typical appliances used.

---

## 📁 Recommended Directory Structure

```text
gas_refill_api/
│
├── app/
│   ├── __init__.py
│   ├── main.py                     # Application entry point & FastAPI app instantiation
│   │
│   ├── core/                       # Core application configurations & security
│   │   ├── __init__.py
│   │   ├── config.py               # pydantic-settings BaseSettings (env vars, DB URLs)
│   │   ├── security.py             # Password hashing, JWT token generation & verification
│   │   └── deps.py                 # Common FastAPI dependencies (get_db, current_user)
│   │
│   ├── db/                         # Database initialization & session management
│   │   ├── __init__.py
│   │   ├── base.py                 # Declarative Base for SQLAlchemy ORM models
│   │   └── session.py              # Async / Sync SQLAlchemy Engine and SessionLocal
│   │
│   ├── models/                     # Database ORM models (SQLAlchemy / SQLModel)
│   │   ├── __init__.py
│   │   ├── gas_station.py          # GasStation table model
│   │   ├── refill_log.py           # RefillLog / Cylinder table model
│   │   ├── delivery_order.py       # DeliveryOrder table model
│   │   └── user_profile.py         # User & ConsumptionProfile table models
│   │
│   ├── schemas/                    # Data Validation & Serialization (Pydantic V2)
│   │   ├── __init__.py
│   │   ├── gas_station.py          # StationCreate, StationUpdate, StationResponse
│   │   ├── refill_log.py           # RefillLogCreate, RefillLogResponse
│   │   ├── delivery_order.py       # OrderCreate, OrderUpdateStatus, OrderResponse
│   │   └── user_profile.py         # ProfileCreate, ProfileUpdate, ProfileResponse
│   │
│   ├── services/                   # Business Logic & Core Algorithms
│   │   ├── __init__.py
│   │   ├── consumption_engine.py   # Consumption tracking & days-until-empty calculation
│   │   ├── dispatch_service.py     # Driver assignment & nearest gas station lookup
│   │   ├── order_service.py        # Order creation, total price estimation & status transitions
│   │   └── station_service.py      # Station filtering & inventory management
│   │
│   └── routers/                    # API Endpoints / Controllers (FastAPI APIRouters)
│       ├── __init__.py
│       ├── api.py                  # Master router aggregating all domain sub-routers
│       ├── gas_stations.py         # /api/v1/stations endpoints
│       ├── refill_logs.py          # /api/v1/refills endpoints
│       ├── delivery_orders.py      # /api/v1/orders endpoints
│       └── consumption.py          # /api/v1/consumption endpoints
│
├── alembic/                        # Database migration scripts (Alembic)
├── tests/                          # Test suite (pytest + httpx AsyncClient)
│   ├── conftest.py                 # Test DB fixtures and client setups
│   ├── test_stations.py
│   ├── test_orders.py
│   └── test_consumption.py
│
├── .env.example                    # Environment variable template
├── alembic.ini                     # Migration config file
├── requirements.txt / pyproject.toml
└── README.md
```

---

## 🌟 What This Architecture Does

This architecture enforces the **Layered Architecture (N-Tier)** design pattern, which separates concerns across distinct layers to maximize maintainability, scalability, and testability:

1. **HTTP / Routing Layer (`routers/`):** Handles web requests, endpoints, status codes, and returns HTTP responses.
2. **Validation / DTO Layer (`schemas/`):** Ensures incoming data from clients is strictly validated, correctly typed, and sanitized.
3. **Business Logic Layer (`services/`):** Encapsulates core algorithms (consumption estimation, order state validation, station pricing calculations).
4. **Data Access Layer (`models/` & `db/`):** Defines SQL database tables and handles persistent storage.
5. **Configuration & Security Layer (`core/`):** Manages environment variables, authentication, JWT tokens, and global dependencies.

---

## 📄 File-by-File Breakdown

### 1. Root Configuration Files
* **`.env.example`**: Template listing required environment variables (e.g., `DATABASE_URL`, `SECRET_KEY`, `ALGORITHM`).
* **`alembic.ini`**: Configuration file for Alembic database migrations.
* **`requirements.txt` / `pyproject.toml`**: Project dependencies (FastAPI, Uvicorn, SQLAlchemy, Pydantic, Passlib, PyJWT, Pytest).
* **`README.md`**: Setup instructions, architecture overview, and API documentation.

### 2. `alembic/` (Database Migrations)
* **`alembic/env.py` & `versions/`**: Manages database schema migrations over time, allowing safe schema updates without data loss.

### 3. `tests/` (Automated Testing Suite)
* **`conftest.py`**: Pytest setup file for test DB fixtures and async HTTP clients.
* **`test_stations.py`**: Tests station creation, querying, and filtering.
* **`test_orders.py`**: Tests order placement, status state transitions, and price calculations.
* **`test_consumption.py`**: Tests gas usage estimation algorithms.

### 4. `app/` (Application Core)

#### Root Application Files
* **`app/__init__.py`**: Python package initializer.
* **`app/main.py`**: FastAPI app instantiation, CORS middleware setup, and master router inclusion.

#### `app/core/` (Configurations & Security)
* **`config.py`**: Type-safe settings management using `pydantic-settings`.
* **`security.py`**: Password hashing (bcrypt) and JWT access token creation/verification.
* **`deps.py`**: Shared FastAPI dependencies like `get_db()` and `get_current_user()`.

#### `app/db/` (Database Session & Infrastructure)
* **`base.py`**: SQLAlchemy `DeclarativeBase` metadata instance for model registration.
* **`session.py`**: Database engine creation and `SessionLocal` maker.

#### `app/models/` (SQLAlchemy ORM Database Models)
* **`gas_station.py`**: Table for gas stations (`name`, `address`, `coordinates`, `price_per_kg`, `available_cylinder_sizes`).
* **`refill_log.py`**: Table for cylinder refill records (`cylinder_size_kg`, `refill_date`, `user_id`, `cost`).
* **`delivery_order.py`**: Table for delivery orders (`user_id`, `station_id`, `driver_id`, `pickup_address`, `status`, `created_at`).
* **`user_profile.py`**: Table for User and Consumption Profile (`household_size`, `cooking_frequency_per_day`, `appliances_used`).

#### `app/schemas/` (Pydantic V2 Schemas)
* **`gas_station.py`**: `StationCreate`, `StationUpdate`, and `StationResponse` models.
* **`refill_log.py`**: `RefillLogCreate` and `RefillLogResponse` models with validators for cylinder sizes.
* **`delivery_order.py`**: `OrderCreate`, `OrderUpdateStatus`, and `OrderResponse` models with status Enums.
* **`user_profile.py`**: `ProfileCreate` and `ProfileResponse` models.

#### `app/services/` (Business Logic & Algorithms)
* **`consumption_engine.py`**: Calculates consumption rate (kg/day) and projects remaining days of gas.
* **`dispatch_service.py`**: Driver assignment and nearest gas station matching logic.
* **`order_service.py`**: Order price calculation and state transition checks (`PENDING` -> `DRIVER_ASSIGNED` -> `COMPLETED`).
* **`station_service.py`**: DB queries for station lookup and inventory management.

#### `app/routers/` (FastAPI Endpoints)
* **`api.py`**: Master router combining all domain sub-routers.
* **`gas_stations.py`**: Routes for `/stations`.
* **`refill_logs.py`**: Routes for `/refills`.
* **`delivery_orders.py`**: Routes for `/orders`.
* **`consumption.py`**: Routes for `/consumption`.

---

## ⚡ Quick Layer Reference Matrix

| Layer | Primary Tool | Main Responsibility | Example Code |
| :--- | :--- | :--- | :--- |
| **Routers** | FastAPI `APIRouter` | Endpoint handlers & HTTP responses | `@router.post("/orders")` |
| **Schemas** | Pydantic V2 | Payload validation & response serialization | `class OrderCreate(BaseModel)` |
| **Services** | Pure Python | Business rules, calculations & workflows | `calculate_depletion_date()` |
| **Models** | SQLAlchemy ORM | Database schema & relationship mapping | `class DeliveryOrder(Base)` |
| **Core/DB** | SQLAlchemy / PyJWT | Database sessions, auth & global settings | `get_db()`, `create_token()` |
