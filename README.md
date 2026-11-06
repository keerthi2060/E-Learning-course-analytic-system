# E-Learning Course Analytics System

Next.js App Router, TypeScript, Prisma, MySQL, and Recharts project. The current dashboard reads database-backed reports. It does not contain a sign-in system.

## Local Requirements

- Node.js LTS and npm
- MySQL Community Server 8.x, or Docker Desktop with a local MySQL container
- A local development database only; do not use production credentials or data

The project currently expects `DATABASE_URL` from `.env`. The checked-in `.env.example` documents the local connection shape. Do not commit `.env` or paste its password into chat or logs.

## Windows MySQL Setup

### Install MySQL Server

1. Install MySQL Community Server using the official Windows installer, or install Docker Desktop if you prefer a local container.
2. In the installer, configure MySQL as a Windows service and start that service. Keep the service limited to your local machine for development.
3. Open a new PowerShell window and check that the client is available:

   ```powershell
   mysql --version
   Get-Service *mysql*
   Test-NetConnection localhost -Port 3306
   ```

If using Docker Desktop instead, start Docker Desktop and create a local MySQL container with a database named `e_learning_analytics_dev`. Keep its port bound to localhost. Do not publish it to the network.

### Create a Development Database

Connect to your local MySQL instance. The password prompt is handled directly by MySQL; do not put the password into this command or share it:

```powershell
mysql -u root -p
```

At the MySQL prompt, create a development-only database:

```sql
CREATE DATABASE e_learning_analytics_dev CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

For better isolation, create a separate local development user in MySQL Workbench or your local MySQL administration tool and grant it access only to `e_learning_analytics_dev`. Choose and enter that password locally; do not commit it or share it.

### Configure the Application

1. Copy `.env.example` to `.env` in the project root.
2. Edit `DATABASE_URL` locally so it points to `localhost:3306/e_learning_analytics_dev` and uses the local database user's credentials. URL-encode special characters in the password. Never use a production URL here.
3. Leave `REPORTING_TIME_ZONE=UTC` and `INACTIVITY_THRESHOLD_DAYS=14` unless you intentionally want different local reporting settings.
4. From PowerShell in the project directory, confirm Prisma can read the schema and apply the migration:

   ```powershell
   npx prisma validate
   npx prisma migrate dev --name init
   ```

The initial migration is in `prisma/migrations/20261027000000_init/`. If Prisma reports that the migration was already applied, do not reset a database containing data; inspect the migration status first.

### Seed Fictional Data

After the migration succeeds, run:

```powershell
npm run db:seed
```

The seed creates fictional courses, learners, enrolments, lessons, progress, and activity. It is designed to be repeatable for the same database. Do not point it at production.

## Tests and Checks

Unit tests do not require MySQL:

```powershell
npm run lint
npx tsc --noEmit
npm test
npm run build
```

The database integration test is opt-in. It accepts only MySQL URLs for `localhost`, `127.0.0.1`, or `::1` whose database name ends in `_dev` or `_test`. Set a separate `DATABASE_INTEGRATION_URL` in the PowerShell process to a disposable local test database, apply migrations to that database first, then run:

```powershell
$env:DATABASE_INTEGRATION_URL = "mysql://LOCAL_TEST_USER:LOCAL_TEST_PASSWORD@localhost:3306/e_learning_analytics_test"
npm run test:integration
Remove-Item Env:DATABASE_INTEGRATION_URL
```

Replace the illustrative URL locally. Do not send its password to chat, include it in command history, or use production credentials. Prefer setting this environment variable through a local secret manager or a temporary PowerShell prompt that does not persist the value.

The integration test creates uniquely named records, verifies reads, updates, duplicate constraints, and restrictive foreign-key behavior, then cleans up. It is skipped unless the URL passes the local database safety check.

## Production Write Access

Sign-in was removed. Production API writes are deliberately blocked with HTTP 403 by the shared API wrapper until secure authentication and server-side authorization are implemented. Do not remove that guard or expose this application as a publicly writable service.
"# E-Learning" 
