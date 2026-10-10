# Express.js + TypeScript + Prisma + MongoDB

Express.js, TypeScript, Prisma ORM ar MongoDB diye banano ekta clean, modular REST API starter.
Ei guide ta step by step follow korle zero theke running project pawa jabe.

---

## Table of Contents

1. [Tech Stack](#tech-stack)
2. [Prerequisites](#prerequisites)
3. [Step 1: Project Initialization](#step-1-project-initialization)
4. [Step 2: Dependencies Install](#step-2-dependencies-install)
5. [Step 3: TypeScript Configuration](#step-3-typescript-configuration)
6. [Step 4: package.json Configuration](#step-4-packagejson-configuration)
7. [Step 5: Prisma Init](#step-5-prisma-init)
8. [Step 6: Environment Configuration](#step-6-environment-configuration)
9. [Step 7: Prisma Schema](#step-7-prisma-schema)
10. [Step 8: Prisma Generate and DB Push](#step-8-prisma-generate-and-db-push)
11. [Step 9: Project Structure](#step-9-project-structure)
12. [Step 10: Prisma Configuration](#step-10-prisma-configuration)
13. [Step 11: Env Config File](#step-11-env-config-file)
14. [Step 12: User Module](#step-12-user-module)
15. [Step 13: Express App](#step-13-express-app)
16. [Step 14: Server Entry Point](#step-14-server-entry-point)
17. [Step 15: Project Run](#step-15-project-run)
18. [Step 16: Production Build](#step-16-production-build)
19. [Prisma Commands Cheat Sheet](#prisma-commands-cheat-sheet)
20. [New Module Add Kora](#new-module-add-kora)
21. [Troubleshooting](#troubleshooting)
22. [Security Checklist](#security-checklist)
23. [Production Improvements](#production-improvements)

---

## Tech Stack

| Tool | Kaj |
| --- | --- |
| Node.js | Runtime |
| Express.js | Web framework |
| TypeScript | Type safety |
| Prisma ORM v6 | Database ORM |
| MongoDB | Database |
| CORS | Cross-origin request handle |
| Helmet | Security headers |
| Morgan | HTTP request logger |
| dotenv | Environment variable load |
| tsx | TypeScript direct run (dev) |

---

## Prerequisites

Shuru korar age check kore nao:

- Node.js 18 ba tar upore (`node -v`)
- npm (`npm -v`)
- MongoDB Atlas account (free cluster hole cholbe) ba local MongoDB
- Code editor (VS Code recommended)

---

## Step 1: Project Initialization

Notun folder baniye project initialize koro:

```bash
mkdir express-prisma
cd express-prisma
npm init -y
```

Ei command `package.json` file create korbe.

---

## Step 2: Dependencies Install

### Production Dependencies

```bash
npm install express cors helmet morgan dotenv @prisma/client@^6.4.1
```

### Development Dependencies

```bash
npm install -D typescript tsx prisma@^6.4.1 @types/node @types/express @types/cors @types/morgan
```

---

## Step 3: TypeScript Configuration

Project root-e `tsconfig.json` file create koro:

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

| Option | Kaj |
| --- | --- |
| `target` | Modern JavaScript-e compile kore |
| `module` | Node.js ESM module system use kore |
| `moduleResolution` | Node.js-er rule onujayi module resolve kore |
| `rootDir` | Source code folder |
| `outDir` | Compiled JavaScript folder |
| `strict` | Strict type checking on kore |
| `esModuleInterop` | CommonJS ar ESM compatibility thik rakhe |
| `skipLibCheck` | Declaration file type check skip kore (fast build) |
| `forceConsistentCasingInFileNames` | File name casing issue atkay |

---

## Step 4: package.json Configuration

`npm init -y` er por generate hoya `package.json` open kore ei field gulo update koro. `"type": "module"` ta must, karon amra ESM use korchi.

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

Scripts er kaj:

| Command | Kaj |
| --- | --- |
| `npm run dev` | Development server, file change hole auto reload |
| `npm run build` | TypeScript ke JavaScript-e compile kore `dist/` folder-e rakhe |
| `npm start` | Compiled production build run kore |

Note: `dependencies` ar `devDependencies` section Step 2 te automatic add hoye geche, seta delete korbe na.

---

## Step 5: Prisma Init

MongoDB provider diye Prisma initialize koro:

```bash
npx prisma init --datasource-provider mongodb
```

Ei command duita jinis create korbe:

```text
prisma/
└── schema.prisma

.env
```

Ekhon `.gitignore` file create koro (na thakle) ar ei lines add koro:

```gitignore
node_modules
dist
.env
.env.local
```

Important: `.env` kokhono Git-e commit korbe na. Ete database password thake.

---

## Step 6: Environment Configuration

### 6.1 MongoDB connection string collect koro

MongoDB Atlas-e:

1. Cluster-e giye **Connect** button click koro
2. **Drivers** select koro
3. Connection string copy koro

### 6.2 `.env` file update koro

Prisma init `.env` file already baniyeche. Setake ei bhabe update koro:

```env
PORT=5000
NODE_ENV=development

DATABASE_URL="mongodb+srv://<username>:<password>@cluster0.xxxxx.mongodb.net/<dbname>?retryWrites=true&w=majority"
```

Real example:

```env
PORT=5000
NODE_ENV=development

DATABASE_URL="mongodb+srv://myuser:mypassword@cluster0.xxxxx.mongodb.net/mydatabase?retryWrites=true&w=majority"
```

### 6.3 Checklist

- `<username>` ar `<password>` replace koro (angle bracket `<>` soho)
- Database name (`mydatabase`) must dite hobe, na dile Prisma `test` database use korbe
- Password-e special character (`@`, `#`, `/`, `:`) thakle URL-encode korte hobe (jemon `@` hobe `%40`)
- MongoDB Atlas > **Network Access**-e tomar IP whitelist koro (development-e `0.0.0.0/0` dile cholbe, production-e na)
- Database user create kora ache kina check koro

---

## Step 7: Prisma Schema

`prisma/schema.prisma` file open kore puro content ei dike replace koro:

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

Schema bujhe nao:

| Part | Mane |
| --- | --- |
| `@id @default(auto()) @map("_id") @db.ObjectId` | MongoDB-r `_id` ke Prisma-r `id` hishebe map kore |
| `@unique` | Same email duibar use kora jabe na |
| `@default("user")` | Role na dile default `user` hobe |
| `@updatedAt` | Record update hole automatic time update hoy |
| `@@map("users")` | MongoDB collection-er naam `users` hobe |

Schema validate ar format kore dekho:

```bash
npx prisma validate
npx prisma format
```

---

## Step 8: Prisma Generate and DB Push

### 8.1 Prisma Client generate

```bash
npx prisma generate
```

Ei command schema theke type-safe Prisma Client banay. Jokhon-i `schema.prisma` change korbe, abar run korbe.

### 8.2 Database-e schema sync

MongoDB-te `prisma migrate` kaj kore na. Tai `db push` use korte hoy:

```bash
npx prisma db push
```

Ete collection ar unique index (`email`) database-e create hoy. Ei step skip korle `email @unique` kaj korbe na.

Expected output:

```text
Your database is now in sync with your Prisma schema.
```

---

## Step 9: Project Structure

Ekhon `src` folder ar baki file/folder gulo baniye nao. Final structure ei rokom hobe:

```text
express-prisma/
│
├── prisma/
│   └── schema.prisma
│
├── src/
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

Folder ar file ek command-e baniye nite chaile:

```bash
mkdir -p src/config src/modules/user
touch src/app.ts src/server.ts
touch src/config/env.ts src/config/prisma.ts
touch src/modules/user/user.controller.ts src/modules/user/user.service.ts src/modules/user/user.routes.ts
```

Architecture flow:

```text
Request -> Route -> Controller -> Service -> Prisma Client -> MongoDB
```

| Layer | Dayitto |
| --- | --- |
| Route | API endpoint define kora |
| Controller | Request data read kora, service call kora, response pathano, error pass kora |
| Service | Business logic ar Prisma query |
| Prisma | Database communication |

---

## Step 10: Prisma Configuration

`src/config/prisma.ts`:

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

export default prisma;
```

Ei file ekta reusable Prisma Client instance dey. Puro app-e ekhan theke import korbe, kokhono notun `new PrismaClient()` baniye na.

---

## Step 11: Env Config File

`src/config/env.ts`:

```typescript
import dotenv from "dotenv";

dotenv.config();

export const ENV = {
  PORT: process.env.PORT ? Number(process.env.PORT) : 5000,
  NODE_ENV: process.env.NODE_ENV || "development",
  DATABASE_URL: process.env.DATABASE_URL as string,
};
```

Environment variable gulo ek jaygay rakhle puro app-e consistent vabe access kora jay.

---

## Step 12: User Module

Ei module-e 3 ta file lagbe. Order: Service, Controller, Routes.

### 12.1 Service

`src/modules/user/user.service.ts`:

```typescript
import prisma from "../../config/prisma.js";

export const getUsers = async () => {
  return prisma.user.findMany({
    select: {
      id: true,
      name: true,
      email: true,
      role: true,
      createdAt: true,
      updatedAt: true,
    },
    orderBy: {
      createdAt: "desc",
    },
  });
};
```

Note: `select` diye `password` field bad deya hoyeche, jate API response-e password leak na hoy.

### 12.2 Controller

`src/modules/user/user.controller.ts`:

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

### 12.3 Routes

`src/modules/user/user.routes.ts`:

```typescript
import { Router } from "express";
import { getUsersController } from "./user.controller.js";

const router = Router();

router.get("/", getUsersController);

export const userRoutes = router;
```

Important: ESM mode-e import path-er shesh-e `.js` extension likhte hobe (file `.ts` hole-o). Eta na likhle runtime error pabe.

---

## Step 13: Express App

`src/app.ts`:

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

// Security headers
app.use(helmet());

// CORS
app.use(cors());

// HTTP request logger
app.use(morgan("dev"));

// Body parsers
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Health check
app.get("/health", (req: Request, res: Response) => {
  res.status(200).json({
    status: "OK",
    timestamp: new Date().toISOString(),
  });
});

// API routes
app.use("/api/v1/users", userRoutes);

// 404 handler
app.use((req: Request, res: Response) => {
  res.status(404).json({
    success: false,
    message: `Cannot ${req.method} ${req.originalUrl}`,
  });
});

// Global error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err);

  res.status(500).json({
    success: false,
    message: err.message || "Internal Server Error",
  });
});

export default app;
```

Middleware order important: security, parser, routes, 404, tarpor error handler shobar shesh-e.

---

## Step 14: Server Entry Point

`src/server.ts`:

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

// Graceful shutdown
const shutdown = async () => {
  console.log("Shutting down server...");
  await prisma.$disconnect();
  process.exit(0);
};

process.on("SIGINT", shutdown);
process.on("SIGTERM", shutdown);

bootstrap();
```

Server start hole:

1. Environment variable load hoy
2. Prisma diye MongoDB connect hoy
3. Express server port-e listen kore
4. Database connect na hole app exit kore

---

## Step 15: Project Run

### 15.1 Development server start

```bash
npm run dev
```

Expected output:

```text
Connected to MongoDB via Prisma
Server running on port 5000
```

### 15.2 Test koro

| Endpoint | URL |
| --- | --- |
| Health check | `GET http://localhost:5000/health` |
| Users list | `GET http://localhost:5000/api/v1/users` |

Health check response:

```json
{
  "status": "OK",
  "timestamp": "2026-10-01T05:00:00.000Z"
}
```

Users response (shurute empty array pabe):

```json
{
  "success": true,
  "data": []
}
```

Browser, Postman, ba curl diye test korte paro:

```bash
curl http://localhost:5000/health
```

---

## Step 16: Production Build

```bash
npm run build
npm start
```

Build korle `dist/` folder generate hoy:

```text
dist/
├── app.js
├── server.js
├── config/
└── modules/
```

Production server-e `.env` file na rekhe hosting platform-er environment variable setting use koro.

---

## Prisma Commands Cheat Sheet

| Command | Kaj |
| --- | --- |
| `npx prisma generate` | Prisma Client generate kore |
| `npx prisma db push` | Schema database-e sync kore |
| `npx prisma validate` | Schema valid kina check kore |
| `npx prisma format` | Schema file format kore |
| `npx prisma studio` | Browser-e database GUI open kore |

---

## New Module Add Kora

Dhoro `auth` module add korbe:

```text
src/modules/
├── auth/
│   ├── auth.controller.ts
│   ├── auth.service.ts
│   └── auth.routes.ts
└── user/
```

Tarpor `app.ts`-e route register koro:

```typescript
import { authRoutes } from "./modules/auth/auth.routes.js";

app.use("/api/v1/auth", authRoutes);
```

Prottek notun feature-er jonno ekta module folder baniye same 3 file pattern (controller, service, routes) follow koro.

---

## Troubleshooting

| Problem | Karon | Solution |
| --- | --- | --- |
| `@prisma/client did not initialize yet` | Prisma Client generate hoyni | `npx prisma generate` run koro |
| `Can't reach database server` | IP whitelist nai ba connection string vul | MongoDB Atlas Network Access check koro, `.env` check koro |
| `Authentication failed` | Username/password vul | Password URL-encode koro, user permission check koro |
| `Cannot find module './app'` | Import-e `.js` extension nai | `./app.js` likho |
| `Unique constraint` ba index kaj korche na | `db push` kora hoyni | `npx prisma db push` run koro |
| Port already in use | 5000 port onno process use korche | `.env`-e `PORT` change koro |
| Schema change korar por type error | Client purono | `npx prisma generate` abar run koro |

---

## Security Checklist

Production-e deploy korar age:

- [ ] `.env` kokhono commit kora hoyni
- [ ] Strong database credentials
- [ ] MongoDB Network Access thik kora
- [ ] HTTPS use kora
- [ ] CORS shudhu trusted origin-er jonno configure kora
- [ ] Helmet enabled
- [ ] Shob request input validate kora
- [ ] Password hash kora (Argon2 ba bcrypt)
- [ ] API response-e password expose hoy na
- [ ] Rate limiting add kora
- [ ] Authentication ar authorization add kora
- [ ] Proper error handling
- [ ] Dependency update rakha
- [ ] Production environment variable use kora

---

## Production Improvements

Ei setup ekta MVP foundation. Boro application-er jonno ei gulo add korar kotha bhabo:

- Zod diye request validation
- Custom error class
- JWT/session authentication
- Argon2 ba bcrypt diye password hashing
- Rate limiting
- Structured logging
- Swagger/OpenAPI documentation
- Pagination
- Database index
- Automated test
- CI/CD
- Docker support
- Monitoring ar error tracking

---

## Quick Start (Already Setup Kora Project)

Kono developer jodi ready project clone kore, tahole shudhu:

```bash
npm install
npx prisma generate
npx prisma db push
npm run dev
```

Tarpor `http://localhost:5000/health` open koro.

---

## License

Ei project personal ar commercial use-er jonno available. Public distribute korle tomar pochondoer license ekhane add koro.
