# bon-go

Cloudflare Workers + TypeScript + D1 implementation of the MINYO LAND stamp-rally API.

This repository is the independent TypeScript/Cloudflare deployment of the stamp-rally service. The Go implementation remains in [bonsai/minyo-go](https://github.com/bonsai/minyo-go).

## Stack

- Cloudflare Workers for the HTTP API
- Cloudflare D1 for SQLite-compatible persistent storage
- Workers Static Assets for the map UI
- Wrangler for local development, migrations, seeding, and deployment
- Vitest for tests

## API

- `GET /healthz`
- `GET /api/spots`
- `GET /api/spots/:id/quiz`
- `POST /api/attempts`
- `GET /api/users/:id/stamps`
- `GET /api/users/:id/progress`

A stamp is awarded only when all three active questions are answered correctly. Field check-in requires coordinates within 30 km of a verified spot. Test mode is disabled by default.

## Local development

```sh
npm install
npm run db:local
npm run seed:local
npm run dev
```

## Cloudflare deployment

1. Create a D1 database: `npm run db:create`.
2. Put the returned `database_id` into `wrangler.jsonc`.
3. Apply migrations: `npm run db:remote`.
4. Import verified spot and quiz data: `npm run seed:remote`.
5. Deploy: `npm run deploy`.

For GitHub Actions deployment, configure repository secret `CLOUDFLARE_API_TOKEN` and repository variable `CLOUDFLARE_ACCOUNT_ID`. The deployment workflow is manual (`workflow_dispatch`) and refuses to deploy while the D1 ID is the placeholder.

## Data

- `spots-verified.json`: verified stamp-rally locations
- `quizzes.json`: three-question quiz sets for locations

Only verified spots are seeded. Candidate research data is intentionally not imported.

## Security and limitations

The current `user_id` is a client-provided identifier, not an authenticated identity. Do not treat it as secure account authentication. Authentication and abuse prevention remain follow-up work.

Coordinates that are not field-verified remain null and cannot be used for normal GPS check-in.

## Relationship to minyo-go

This repository owns the TypeScript/Workers implementation. Go source and its local SQLite workflow remain in `bonsai/minyo-go`. Keep data/API contracts aligned between the two implementations.
