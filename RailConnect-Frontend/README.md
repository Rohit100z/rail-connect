# RailConnect

RailConnect is a full-stack railway reservation system. It provides a React passenger interface, an Express REST API, and a MySQL data layer for train availability, reservations, cancellations, waitlists, and admin operations.

## Highlights

- Register, log in, log out, and manage a passenger profile
- Browse trains and current seat availability
- Book confirmed tickets or join a train waitlist
- View booking history, summaries, and waitlist position
- Cancel bookings and promote the earliest eligible waitlisted booking
- Admin-only train creation, editing, deletion, occupancy, and booking statistics
- JWT authentication stored in an HTTP-only cookie
- MySQL transactions and stored procedures for booking operations

## Architecture

```text
RailConnect-Frontend/  React 19 + Vite + React Router
          |
          | JSON over HTTP, credentials: include
          v
RailConnect-Backend/   Express + cookie-parser + JWT + mysql2
          |
          v
MySQL                   users, passenger, train, availability, booking
```

The backend mounts these API groups:

- `/api/auth` for authentication
- `/api/train` for public train and waitlist queries
- `/api/user` for authenticated passenger operations
- `/api/admin` for admin-only operations
- `/api/booking` for a legacy placeholder route

## Requirements

- Node.js 18 or newer
- npm
- MySQL 8 or a compatible MySQL server with InnoDB, foreign keys, transactions, and stored-procedure support

## Getting Started

### 1. Clone and prepare the database

From the repository root, run the application-aligned schema script against MySQL:

```bash
mysql -u root -p < RailConnect-Backend/a.sql
```

The script creates the `RAILWAY` database, tables, triggers, stored procedures, and sample train/passenger data. It also contains demonstration queries near the end of the file; review or remove those statements before using the script as a production migration.

### 2. Configure and start the backend

```bash
cd RailConnect-Backend
npm install
```

Copy `.env.example` to `.env` and set values for your local MySQL instance:

```env
PORT=3000
JWT_SECRET=replace-with-a-long-random-secret
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_NAME=RAILWAY
```

Start the API:

```bash
npm run dev
```

Use `npm start` for a regular Node.js start without Nodemon. The API defaults to `http://localhost:3000`.

### 3. Configure and start the frontend

In a second terminal:

```bash
cd RailConnect-Frontend
npm install
```

Copy `.env.example` to `.env` and set the API URL:

```env
VITE_BACKEND_URL=http://localhost:3000
```

Start the Vite development server:

```bash
npm run dev
```

Open the URL printed by Vite, normally `http://localhost:5173`.

## Available Scripts

### Backend

| Command | Description |
| --- | --- |
| `npm run dev` | Start the API with Nodemon |
| `npm start` | Start the API with Node.js |

### Frontend

| Command | Description |
| --- | --- |
| `npm run dev` | Start the Vite development server |
| `npm run build` | Build the production frontend |
| `npm run preview` | Preview the production build |
| `npm run lint` | Run ESLint |

## API Overview

All request bodies use JSON. Authenticated browser requests rely on the `jwt` HTTP-only cookie.

### Authentication

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/auth/register` | Create a user and passenger profile |
| `POST` | `/api/auth/login` | Authenticate and set the JWT cookie |
| `POST` | `/api/auth/logout` | Clear the JWT cookie |

Example registration request:

```json
{
  "name": "Asha Kumar",
  "email": "asha@example.com",
  "password": "change-this-password",
  "phone": "9876543210"
}
```

### Trains

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/train` | List trains and available seats |
| `GET` | `/api/train/:id` | Get one train and its availability |
| `GET` | `/api/train/:id/waitlist` | Get the ordered waitlist for a train |

### Authenticated user operations

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/user/profile` | Get the current user and passenger profile |
| `POST` | `/api/user/passenger` | Create a passenger record |
| `PUT` | `/api/user/profile` | Update profile fields |
| `PUT` | `/api/user/change-password` | Change the account password |
| `POST` | `/api/user/book` | Book a train or join its waitlist |
| `POST` | `/api/user/cancel` | Cancel an owned booking |
| `GET` | `/api/user/bookings` | List bookings; optionally filter with `?status=CONFIRMED` |
| `GET` | `/api/user/summary` | Get confirmed, waitlisted, and cancelled counts |
| `GET` | `/api/user/waitlist/:pnr` | Get the current waitlist position |

Booking request body:

```json
{
  "train_no": 101,
  "phone": "9876543210"
}
```

Cancellation request body:

```json
{
  "pnr": 1
}
```

### Admin operations

Admin endpoints require an authenticated user whose database role is `admin`.

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/admin/train` | Add a train |
| `PUT` | `/api/admin/train/:id` | Update train details |
| `DELETE` | `/api/admin/train/:id` | Delete a train |
| `GET` | `/api/admin/train/:id` | View train details |
| `GET` | `/api/admin/statistics` | View system-wide train statistics |
| `GET` | `/api/admin/statistics?train_no=101` | View detailed train statistics and passengers |

## Frontend Routes

| Route | Access |
| --- | --- |
| `/` | Public introduction page |
| `/home` | Public train browsing and booking entry point |
| `/signup` | Public registration |
| `/login` | Public login |
| `/profile` | Authenticated users |
| `/change-password` | Authenticated users |
| `/update-profile` | Authenticated users |
| `/my-bookings` | Authenticated users |
| `/admin` | Admin users |

## Database Notes

The application expects the schema in `RailConnect-Backend/a.sql`, including the `users.user_id` relationship on `passenger` and procedures such as `register_user`, `create_passenger_and_book`, `book_ticket`, and `cancel_and_promote`.

`railway_system_full.sql` is an older schema variant and does not match the current controllers. `advanced_sql.sql` contains additional queries rather than a complete application bootstrap. Use `a.sql` as the starting point for the current codebase.

Availability is tracked per train, not per travel date. The system currently has no payment flow, ticket PDF, email notification, travel-date selection, or seat-class selection.

## Development Caveats

- Backend CORS currently allows only `http://localhost:5173`; update `RailConnect-Backend/app.js` for another frontend origin.
- The JWT cookie is configured with `secure: false`, which is suitable for local HTTP development but should be changed for HTTPS deployment.
- Use a strong, private `JWT_SECRET` outside local development.
- Do not use the sample database password value `CHANGE_ME` for a real account. Register through the application so passwords are stored as bcrypt hashes.
- The repository has separate `package.json` files; dependencies must be installed independently in the backend and frontend directories.

## Project Structure

```text
RailConnect-Backend/
  controllers/       Request handlers
  middleware/        Authentication, authorization, and errors
  routes/            Express route modules
  utils/             Error and JWT helpers
  app.js             Express application and middleware
  server.js          HTTP server entry point
  db.js              MySQL connection pool
  a.sql              Current schema, procedures, triggers, and sample data

RailConnect-Frontend/
  src/pages/         Application screens
  src/components/    Shared UI and route guards
  src/GlobalContext.jsx  Authentication and global state
  src/App.jsx        Shared application layout
  src/main.jsx       Router and React entry point
```

## API Testing

The repository includes `RailConnect-Backend/Railway Reservation System.postman_collection.json`, which targets the default backend URL `http://localhost:3000` and includes authentication, user, booking, and admin requests.
