# Express.js + TypeScript + Prisma + MongoDB

A clean and modular MVP backend starter built with **Express.js, TypeScript, Prisma ORM, and MongoDB**.

This setup is designed to provide a simple and scalable foundation for building REST APIs with a modular architecture.

---

## Tech Stack

* **Node.js**
* **Express.js**
* **TypeScript**
* **Prisma ORM v6**
* **MongoDB**
* **CORS**
* **Helmet**
* **Morgan**
* **dotenv**
* **tsx**

---

# 1. Project Initialization

Create a new project directory:

```bash
mkdir express-prisma
cd express-prisma
```

Initialize the Node.js project:

```bash
npm init -y
```

---

# 2. Install Dependencies

## Production Dependencies

Install Express, Prisma Client, and other required packages:

```bash
npm install express cors helmet morgan dotenv @prisma/client@^6.4.1
```

## Development Dependencies

Install TypeScript, Prisma CLI, type definitions, and `tsx`:

```bash
npm install -D typescript tsx prisma@^6.4.1 @types/node @types/express @types/cors @types/morgan
```

---

# 3. TypeScript Configuration

Create a `tsconfig.json` file in the root directory:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"]
}
```

### Configuration Explanation

| Option                             | Purpose                                  |
| ---------------------------------- | ---------------------------------------- |
| `target`                           | Compiles TypeScript to modern JavaScript |
| `module`                           | Uses Node.js ESM modules                 |
| `moduleResolution`                 | Resolves modules using Node.js rules     |
| `rootDir`                          | Source code directory                    |
| `outDir`                           | Compiled JavaScript directory            |
| `strict`                           | Enables strict TypeScript checking       |
| `esModuleInterop`                  | Improves CommonJS/ESM interoperability   |
| `skipLibCheck`                     | Skips type checking of declaration files |
| `forceConsistentCasingInFileNames` | Prevents casing-related import issues    |

---

# 4. Configure package.json

Update your `package.json`:

```json
{
  "name": "express-prisma",
  "version": "1.0.0",
  "description": "Express.js + TypeScript + Prisma + MongoDB backend",
  "type": "module",
  "main": "src/server.ts",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  }
}
```

The important scripts are:

```bash
npm run dev
```

Runs the development server with automatic reload.

```bash
npm run build
```

Compiles TypeScript into JavaScript.

```bash
npm start
```

Runs the compiled production application.

---

# 5. Initialize Prisma

Initialize Prisma:

```bash
npx prisma init
```

This will create:

```text
prisma/
└── schema.prisma

.env
```

---

# 6. MongoDB Configuration

Create or update the `.env` file in the project root:

```env
PORT=5000
NODE_ENV=development

DATABASE_URL="mongodb+srv://<username>:<password>@cluster0.xxxx.mongodb.net/<dbname>?retryWrites=true&w=majority"
```

### Example

```env
PORT=5000
NODE_ENV=development

DATABASE_URL="mongodb+srv://myuser:mypassword@cluster0.xxxxx.mongodb.net/mydatabase?retryWrites=true&w=majority"
```

> Never commit your `.env` file to Git.

Add the following to `.gitignore`:

```gitignore
node_modules
dist
.env
.env.local
```

---

# 7. Prisma Schema

Open:

```text
prisma/schema.prisma
```

Configure Prisma for MongoDB:

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mongodb"
  url      = env("DATABASE_URL")
}

model User {
  id        String   @id @default(auto()) @map("_id") @db.ObjectId
  name      String
  email     String   @unique
  password  String
  role      String   @default("user")
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@map("users")
}
```

---

# 8. Generate Prisma Client

After configuring your Prisma schema, generate the Prisma Client:

```bash
npx prisma generate
```

Whenever you make changes to your Prisma schema, run:

```bash
npx prisma generate
```

again.

---

# 9. Project Structure

Recommended MVP folder structure:

```text
express-prisma/
│
├── prisma/
│   └── schema.prisma
│
├── src/
│   │
│   ├── config/
│   │   ├── env.ts
│   │   └── prisma.ts
│   │
│   ├── modules/
│   │   └── user/
│   │       ├── user.controller.ts
│   │       ├── user.service.ts
│   │       └── user.routes.ts
│   │
│   ├── app.ts
│   └── server.ts
│
├── .env
├── .gitignore
├── package.json
├── package-lock.json
├── tsconfig.json
└── README.md
```

---

# 10. Prisma Configuration

Create:

```text
src/config/prisma.ts
```

Add:

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default prisma;
```

This creates a reusable Prisma Client instance that can be imported throughout the application.

---

# 11. Environment Configuration

Create:

```text
src/config/env.ts
```

Add:

```typescript
import dotenv from "dotenv";

dotenv.config();

export const ENV = {
  PORT: process.env.PORT ? Number(process.env.PORT) : 5000,
  NODE_ENV: process.env.NODE_ENV || "development",
  DATABASE_URL: process.env.DATABASE_URL as string,
};
```

This centralizes environment variables so they can be accessed consistently throughout the application.

---

# 12. Express Application

Create:

```text
src/app.ts
```

Add:

```typescript
import express, {
  Application,
  Request,
  Response,
  NextFunction,
} from "express";
import cors from "cors";
import helmet from "helmet";
import morgan from "morgan";

import { userRoutes } from "./modules/user/user.routes.js";

const app: Application = express();

/**
 * Security
 */
app.use(helmet());

/**
 * CORS
 */
app.use(cors());

/**
 * HTTP Request Logger
 */
app.use(morgan("dev"));

/**
 * Body Parsers
 */
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

/**
 * Health Check
 */
app.get("/health", (req: Request, res: Response) => {
  res.status(200).json({
    status: "OK",
    timestamp: new Date().toISOString(),
  });
});

/**
 * API Routes
 */
app.use("/api/v1/users", userRoutes);

/**
 * 404 Handler
 */
app.use((req: Request, res: Response) => {
  res.status(404).json({
    success: false,
    message: `Cannot ${req.method} ${req.originalUrl}`,
  });
});

/**
 * Global Error Handler
 */
app.use(
  (
    err: Error,
    req: Request,
    res: Response,
    next: NextFunction
  ) => {
    console.error(err);

    res.status(500).json({
      success: false,
      message: err.message || "Internal Server Error",
    });
  }
);

export default app;
```

---

# 13. Server Entry Point

Create:

```text
src/server.ts
```

Add:

```typescript
import app from "./app.js";
import prisma from "./config/prisma.js";
import { ENV } from "./config/env.js";

async function bootstrap() {
  try {
    await prisma.$connect();

    console.log("Connected to MongoDB via Prisma");

    app.listen(ENV.PORT, () => {
      console.log(`Server running on port ${ENV.PORT}`);
    });
  } catch (error) {
    console.error("Database connection failed:", error);

    await prisma.$disconnect();

    process.exit(1);
  }
}

bootstrap();
```

The application will:

1. Load environment variables
2. Connect to MongoDB through Prisma
3. Start the Express server
4. Listen on the configured port

If the database connection fails, the application exits.

---

# 14. User Module

Create the following files:

```text
src/modules/user/
├── user.controller.ts
├── user.service.ts
└── user.routes.ts
```

This follows a simple modular architecture:

```text
Route
  ↓
Controller
  ↓
Service
  ↓
Prisma
  ↓
MongoDB
```

---

## User Service

Create:

```text
src/modules/user/user.service.ts
```

```typescript
import prisma from "../../config/prisma.js";

export const getUsers = async () => {
  return prisma.user.findMany({
    orderBy: {
      createdAt: "desc",
    },
  });
};
```

---

## User Controller

Create:

```text
src/modules/user/user.controller.ts
```

```typescript
import { Request, Response, NextFunction } from "express";
import { getUsers } from "./user.service.js";

export const getUsersController = async (
  req: Request,
  res: Response,
  next: NextFunction
) => {
  try {
    const users = await getUsers();

    res.status(200).json({
      success: true,
      data: users,
    });
  } catch (error) {
    next(error);
  }
};
```

---

## User Routes

Create:

```text
src/modules/user/user.routes.ts
```

```typescript
import { Router } from "express";
import { getUsersController } from "./user.controller.js";

const router = Router();

router.get("/", getUsersController);

export const userRoutes = router;
```

---

# 15. API Endpoints

After starting the application, the following endpoints will be available.

### Health Check

```http
GET /health
```

Example response:

```json
{
  "status": "OK",
  "timestamp": "2026-10-01T05:00:00.000Z"
}
```

---

### Get Users

```http
GET /api/v1/users
```

Example response:

```json
{
  "success": true,
  "data": [
    {
      "id": "66f123456789abcdef123456",
      "name": "John Doe",
      "email": "john@example.com",
      "role": "user",
      "createdAt": "2026-10-01T05:00:00.000Z",
      "updatedAt": "2026-10-01T05:00:00.000Z"
    }
  ]
}
```

> Passwords should never be returned from API responses. In a real application, select only the fields that are safe to expose.

---

# 16. Run the Project

## Development

Start the development server:

```bash
npm run dev
```

Expected output:

```text
Connected to MongoDB via Prisma
Server running on port 5000
```

The API will be available at:

```text
http://localhost:5000
```

Health check:

```text
http://localhost:5000/health
```

Users endpoint:

```text
http://localhost:5000/api/v1/users
```

---

# 17. Build for Production

Compile the TypeScript application:

```bash
npm run build
```

This generates:

```text
dist/
├── app.js
├── server.js
├── config/
└── modules/
```

Start the production build:

```bash
npm start
```

---

# 18. Prisma Commands

### Generate Prisma Client

```bash
npx prisma generate
```

### Validate Prisma Schema

```bash
npx prisma validate
```

### Format Prisma Schema

```bash
npx prisma format
```

### Open Prisma Studio

```bash
npx prisma studio
```

Prisma Studio can be used to inspect and manage database records during development.

---

# 19. MongoDB Notes

This project uses MongoDB through Prisma.

The MongoDB connection string should follow this format:

```env
DATABASE_URL="mongodb+srv://<username>:<password>@<cluster>/<database>?retryWrites=true&w=majority"
```

Make sure:

* MongoDB cluster is running
* Database user exists
* Username and password are correct
* Your IP address is allowed in MongoDB Network Access
* The database name is correct
* The connection string is stored in `.env`

---

# 20. Environment Variables

Recommended `.env`:

```env
PORT=5000
NODE_ENV=development
DATABASE_URL="mongodb+srv://<username>:<password>@<cluster>/<database>?retryWrites=true&w=majority"
```

For production, use environment variables provided by your hosting/server environment instead of committing secrets to Git.

---

# 21. API Architecture

The project follows a modular architecture:

```text
Request
   │
   ▼
Route
   │
   ▼
Controller
   │
   ▼
Service
   │
   ▼
Prisma Client
   │
   ▼
MongoDB
```

### Route

Responsible for defining API endpoints.

### Controller

Responsible for:

* Reading request data
* Calling services
* Returning HTTP responses
* Passing errors to the error handler

### Service

Responsible for:

* Business logic
* Database operations
* Prisma queries

### Prisma

Responsible for:

* Database communication
* Type-safe database queries
* MongoDB interaction

---

# 22. Recommended Development Workflow

After cloning or downloading the project:

### Step 1

Install dependencies:

```bash
npm install
```

### Step 2

Configure `.env`:

```env
PORT=5000
NODE_ENV=development
DATABASE_URL="your-mongodb-connection-string"
```

### Step 3

Generate Prisma Client:

```bash
npx prisma generate
```

### Step 4

Start development server:

```bash
npm run dev
```

### Step 5

Test health endpoint:

```http
GET http://localhost:5000/health
```

---

# 23. Adding a New Module

For example, if you want to add an `auth` module:

```text
src/modules/
├── auth/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.routes.ts
│
└── user/
    ├── user.controller.ts
    ├── user.service.ts
    └── user.routes.ts
```

Then register the route inside `app.ts`:

```typescript
import { authRoutes } from "./modules/auth/auth.routes.js";

app.use("/api/v1/auth", authRoutes);
```

This allows the project to grow without putting all business logic into a single file.

---

# 24. Recommended Production Improvements

This setup is intended as an MVP foundation. For a larger production application, consider adding:

* Request validation with Zod
* Centralized custom error classes
* Authentication with JWT/session
* Password hashing with Argon2 or bcrypt
* Rate limiting
* Request ID / correlation ID
* Structured logging
* API documentation with OpenAPI/Swagger
* Pagination
* Database indexes
* Input sanitization
* Secure CORS configuration
* Graceful shutdown
* Automated tests
* CI/CD
* Environment-specific configuration
* Docker support
* Monitoring and error tracking

---

# 25. Graceful Shutdown

For production applications, Prisma should be disconnected when the Node.js process receives termination signals.

Example:

```typescript
const shutdown = async () => {
  console.log("Shutting down server...");

  await prisma.$disconnect();

  process.exit(0);
};

process.on("SIGINT", shutdown);
process.on("SIGTERM", shutdown);
```

This can be added to `server.ts`.

---

# 26. Security Checklist

Before deploying to production:

* [ ] Never commit `.env`
* [ ] Use strong database credentials
* [ ] Configure MongoDB IP/network access correctly
* [ ] Use HTTPS
* [ ] Configure CORS for trusted origins
* [ ] Enable Helmet
* [ ] Validate all request inputs
* [ ] Hash passwords
* [ ] Never expose passwords in API responses
* [ ] Add rate limiting
* [ ] Add authentication/authorization
* [ ] Add proper error handling
* [ ] Keep dependencies updated
* [ ] Use production environment variables

---

# 27. Quick Start

If everything is already configured, the complete startup process is:

```bash
npm install
```

```bash
npx prisma generate
```

```bash
npm run dev
```

Then open:

```text
http://localhost:5000/health
```

You should receive:

```json
{
  "status": "OK",
  "timestamp": "..."
}
```

---

# License

This project is available for personal and commercial use. Add your preferred license here if the project will be distributed publicly.
