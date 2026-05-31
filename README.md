# SynCraft

A Zapier-style automation platform. You connect a **trigger** (like an incoming webhook) to a chain of **actions**, bundle them into a "Craft", and let it run. When the trigger fires, the actions execute in order.

## Tech Stack

- **Monorepo:** [Turborepo](https://turbo.build/) + npm workspaces
- **Language:** TypeScript
- **Backend:** Node.js + Express
- **Database:** PostgreSQL with [Prisma](https://www.prisma.io/) ORM
- **Messaging:** Apache Kafka (via [KafkaJS](https://kafka.js.org/))
- **Auth:** JWT
- **Validation:** [Zod](https://zod.dev/)

## How It Works

The project is split into a few small services that each do one job:

| Service | What it does |
| --- | --- |
| **primary-backend** | The main REST API — handles sign up / sign in, and lets users create and view their Crafts, triggers, and actions. |
| **hooks** | Receives incoming webhooks (`/hooks/catch/:userId/:craftId`) and records that a Craft should run. |
| **sweeper** | A background worker that picks up those pending runs and pushes them onto Kafka for processing. |

When a webhook comes in, `hooks` writes the run to the database along with an entry in an **outbox** table (in a single transaction). The `sweeper` then continuously reads from that outbox and publishes each run to a Kafka topic, where downstream workers can pick it up and actually execute the actions. This outbox pattern keeps things reliable — a run is never lost just because a message failed to send.

Shared logic lives in `packages/`:

- `database` – Prisma schema + client
- `schemas` – shared Zod validation schemas
- `middleware` – JWT auth middleware

## Getting Started

```sh
# install dependencies
npm install

# set up the database (from packages/database)
npm run db:generate
npm run db:push

# run everything in dev
npm run dev
```

You'll need a running **PostgreSQL** instance (set `DATABASE_URL`) and a **Kafka** broker (defaults to `localhost:9092`).

## Project Structure

```
apps/
  primary-backend/   REST API
  hooks/             webhook receiver
  sweeper/           outbox → Kafka worker
packages/
  database/          Prisma schema & client
  schemas/           Zod schemas
  middleware/        JWT auth
```
