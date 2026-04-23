# Parking Enforcement Web System

Web-only parking enforcement workflow for residential communities. Residents and visitors use a mobile-friendly page opened from a property QR code, while agents and admins work from secure web dashboards.

## What is included

- Resident registration with account details and a strict 3-vehicle limit per apartment or unit
- Resident sign-in and self-service vehicle/profile management
- Visitor registration without an account, with a 1-24 hour timer that starts immediately
- Agent dashboard with fast plate lookup and enforcement logging for invalid vehicles only
- Admin panel for property management, QR entry links, residents, visitor passes, agents, and enforcement history
- Admin removal actions for resident vehicles and enforcement agents
- SQLite + Prisma schema for properties, residents, vehicles, visitor passes, users, and enforcement actions
- Cookie-based login for agents and admins
- Production-oriented environment validation, upload limits, session expiration, and security headers

## Tech stack

- Next.js App Router
- TypeScript
- Prisma ORM
- SQLite for local development

## Production baseline

- Environment variables are validated at startup in `src/lib/env.ts`
- Sessions are signed and time-limited
- Staff and resident auth endpoints are throttled in memory to reduce brute-force abuse
- Staff and resident password reset flows are included
- Evidence uploads are validated for type and size before being stored
- Upload handling is abstracted through `src/lib/storage.ts` and supports `local` or `s3`
- Prisma config has been moved into `prisma.config.ts`
- Response headers disable framing and add basic browser hardening
- Docker deployment scaffolding is included

## Local setup

1. Install dependencies:

```bash
npm install
```

2. Copy environment variables:

```bash
copy .env.example .env
```

3. Generate the Prisma client and migrate the database:

```bash
npm run prisma:generate
npm run prisma:migrate -- --name init
```

4. Seed demo data:

```bash
npm run prisma:seed
```

5. Start the app:

```bash
npm run dev
```

## Environment variables

Copy `.env.example` to `.env` and adjust values as needed:

- `DATABASE_URL`: local SQLite by default for development
- `SESSION_SECRET`: required, use a long random string
- `SESSION_MAX_AGE_HOURS`: signed session lifetime
- `APP_URL`: public application base URL
- `STORAGE_DRIVER`: currently `local`
- `LOCAL_UPLOAD_DIR`: where evidence photos are stored in development
- `UPLOAD_PUBLIC_PATH`: public URL prefix for uploaded evidence
- `MAX_UPLOAD_MB`: maximum allowed evidence photo size
- `RESET_TOKEN_TTL_MINUTES`: password reset link lifetime
- `SMTP_*`: optional SMTP configuration for password reset emails
- `S3_*`: optional object storage configuration for production evidence uploads

## Demo credentials

- Admin: `admin@parking.local` / `Admin123!`
- Agent: `agent@parking.local` / `Agent123!`
- Demo property code: `SUNSET-TOWERS`

## Main routes

- `/` landing page
- `/p/SUNSET-TOWERS` QR entry page for a property
- `/register/resident` resident registration form
- `/resident/login` resident sign-in
- `/resident` resident self-service dashboard
- `/register/visitor` visitor registration form
- `/login` staff sign-in
- `/agent` enforcement dashboard
- `/admin` admin panel

## Notes

- QR generation is implemented as an SVG endpoint per property at `/api/properties/[propertyCode]/qr`.
- Photo evidence uploads are validated and saved into `public/uploads` for local development, or S3-compatible storage in production.
- Agents are restricted to lookup and enforcement actions for their assigned property.
- The next production cutover should replace SQLite with Postgres and local uploads with object storage.
- SMS alerts, pre-expiry reminders, payment collection, and advanced plate formatting are intentionally left as future enhancements.

## Deployment

Basic container deployment files are included:

- `Dockerfile`
- `docker-compose.yml`

Before running in production, configure:

- a strong `SESSION_SECRET`
- a real SMTP provider for reset emails
- `STORAGE_DRIVER=s3` plus S3-compatible credentials for evidence storage
