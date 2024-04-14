---
title: Drizzle
description: Usage of Drizzle
layout: ../../../layouts/docs.astro
lang: en
---

Drizzle is an ORM for TypeScript, that allows you to define your database schema and models in TypeScript itself that can be used to interact with your database from your backend. Unlike Prisma, your database client is type safe without the need for type generation task. 

## Drizzle Client

Located at `src/server/db/index.ts`, the Drizzle Client is instantiated as a global variable and exported to be used in your API routes. We include the Drizzle Client in [Context](/en/usage/trpc#-serverapitrpcts) by default and recommend using this instead of importing it separately in each file.

## Schema

You will find the Database schema file at `src/server/db/schema.ts`. This file is where you define your database schema and models.

### With NextAuth.js

When you select NextAuth.js in combination with Drizzle, the schema file is generated and set up for you with the recommended values for the `User`, `Session`, `Account`, and `VerificationToken` models, as per the [NextAuth.js documentation](https://authjs.dev/getting-started/adapters/drizzle).

## Default Database

The default database is an SQLite database, which is great for development and quickly spinning up a proof-of-concept but is not recommended for production. You can change the database with a few steps:

- Uninstall the current database driver and install the one you want to switch to
- Replace the database connection code (and related import) to the new database driver
- Update the drizzle import in `src/server/db/index.ts` to the new database provider
- Update the `src/server/db/schema.ts` file to use relevant database helpers (this could be a bit tedious if you've already established your database schema)
- If you're using Drizzle Kit for migrations, update the `driver` field in the `drizzle.config.ts` to the new driver and then updating the connection string within environment variables to point to your database

## Seeding your Database

Seeding your database is a great way to quickly populate your database with test data to help you get started. In order to setup seeding, you will need to create a `seed.ts` file in the `src/server/db` directory, and then add a `seed` script to your `package.json` file. You'll also need a TypeScript runner that can execute the seed-script. We recommend [tsx](https://github.com/esbuild-kit/tsx), which is a very performant TypeScript runner that uses esbuild and doesn't require any ESM configuration, but `ts-node` or other runners will work as well.

```jsonc:package.json
{
  "scripts": {
    "db-seed": "NODE_ENV=development tsx ./src/server/db/seed.ts"
  }
}
```

```ts:src/server/db/seed.ts
import { conn, db } from "./";
import { posts } from "./schema";

async function main() {
  await db.insert(posts)
  .values({ id: 1, name: 'Example Post' })
  .onConflictDoUpdate({
    target: users.id,
    set: { name: 'Example Post' },
  });

main()
  .then(async () => {
    await conn.close();
  })
  .catch(async (e) => {
    console.error(e);
    await conn.close();
    process.exit(1);
  });
```

Then, just run `pnpm db-seed` (or `npm`/`yarn`) to seed your database.

## Useful Resources

| Resource                     | Link                                                        |
| ---------------------------- | ------------------------------------------------------------|
| Drizzle Docs                 | https://orm.drizzle.team/docs/overview                      |
| Drizzle GitHub               | https://github.com/drizzle-team/drizzle-orm                 |
| Drizzle Kit Docs             | https://orm.drizzle.team/kit-docs/overview                  |
| NextAuth.JS Drizzle Adapter  | https://authjs.dev/getting-started/adapters/drizzle         |
| PlanetScale Connection Guide | https://orm.drizzle.team/docs/get-started-mysql#planetscale |
