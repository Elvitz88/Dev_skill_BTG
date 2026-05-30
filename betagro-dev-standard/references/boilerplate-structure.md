# Betagro Node.js Backend Boilerplate — Structure Reference

<!-- อ้างอิงจาก boilerplate_NodeJS/Backend-Template (ณ วันที่ 2026-05-30) -->
<!-- ใช้เป็น ground-truth เมื่อ generate หรือ review โครงสร้างโปรเจกต์ -->

## Runtime & Toolchain

| Item | Value |
|------|-------|
| Node.js | 20.13.1 (pin via `.nvmrc`) — pipeline ใช้ 22.x |
| Language | TypeScript 5.x (`"target": "ES2020"`, `"module": "commonjs"`) |
| Module system | CommonJS (`"type": "commonjs"` in package.json) |
| Package manager | npm (lock-file committed) |
| ORM | Prisma 5.x + PostgreSQL |
| Web framework | Express 4.x |
| Process manager | Graceful-shutdown built into `src/index.ts` (SIGTERM/SIGINT) |

---

## Folder Structure

```
Backend-Template/
├── .devcontainer.json          # Dev container config
├── .dockerignore
├── .env.example                # Template — ห้าม commit .env จริง
├── .eslintignore
├── .eslintrc.json
├── .gitignore
├── .husky/
│   ├── pre-commit              # runs: npm run lint:fix
│   └── pre-push                # runs: npm run test
├── .nvmrc                      # "20.13.1"
├── .nycrc                      # nyc coverage config (70/70/70/50)
├── .prettierignore
├── .prettierrc.json
├── .vscode/
│   ├── launch.json
│   ├── setting.json
│   └── tasks.json
├── Dockerfile                  # multi-stage build (node:24-alpine)
├── docker-compose.yml          # (template, currently commented out)
├── jest.config.js
├── knip.json                   # dead-code detector
├── openapi.yaml                # API spec (copied to dist/ on build)
├── package.json
├── package-lock.json
├── catalog-info.yaml           # Backstage catalog
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── pipelines/
│   ├── azure-pipeline-dev.yaml
│   ├── azure-pipeline-scanning.yaml
│   ├── azure-pipelines-dev-azure.yml
│   ├── azure-pipelines-prod.yml
│   ├── azure-pipelines-qa.yml
│   └── azure-pipelines-uat.yml
├── scripts/
│   └── loadTest.js             # k6 load test script
├── src/
│   ├── index.ts                # entry point
│   ├── api-types.ts            # generated from openapi.yaml (openapi-typescript)
│   ├── swagger.json
│   ├── controllers/
│   │   └── <feature>Controller.ts
│   ├── middlewares/
│   │   ├── middlewares.ts
│   │   ├── transactionId.ts    # injects req.id = uuidv4()
│   │   └── validationMiddleware.ts
│   ├── routes/
│   │   ├── index.ts            # setupRoutes(app) — mount all routers under /api
│   │   └── <feature>Routes.ts
│   ├── services/
│   │   └── <feature>Service.ts
│   └── utils/
│       ├── datadog.ts          # dd-trace init (import FIRST in index.ts)
│       ├── logger.ts           # Winston → stdout JSON
│       └── swagger.ts
├── tests/
│   ├── controller/
│   │   └── <feature>Controller.spec.ts
│   ├── services/
│   │   └── <feature>Service.spec.ts
│   └── utils/
│       └── *.spec.ts
├── tsconfig.json
└── updateK8smanifest.sh        # GitOps helper for Azure pipeline
```

---

## Naming Conventions

| Layer | Pattern | Example |
|-------|---------|---------|
| Controller file | `<feature>Controller.ts` or `<feature>-controller.ts` | `exampleController.ts` |
| Service file | `<feature>Service.ts` | `exampleService.ts` |
| Route file | `<feature>Routes.ts` or `<feature>-routes.ts` | `exampleRoutes.ts` |
| Test file | `<feature>.spec.ts` (mirrors src path under `tests/`) | `exampleController.spec.ts` |
| Prisma model | PascalCase | `Example` |
| Env vars | SCREAMING_SNAKE_CASE | `DATABASE_URL`, `APP_PORT` |

<!-- kebab-case ก็ใช้ได้ (ดู public-variable-controller.ts) — ให้สอดคล้องกันภายในโปรเจกต์ -->

---

## Key Source Patterns

### `src/index.ts` — Entry Point
- **Import `./utils/datadog` first** (dd-trace hoisting requirement)
- Middleware order: `transactionIdMiddleware` → `X-commit-ID` header → `express.json()` → `helmet()`
- CORS enabled **only in development** with explicit `allowedOrigins` whitelist
- Health check: `GET /api/health` → 200 `OK` (excluded from DataDog traces)
- Graceful shutdown: `SIGTERM` / `SIGINT` → close HTTP server (10 s timeout) → `prisma.$disconnect()`
- Handles: `uncaughtException`, `unhandledRejection`

```typescript
import './utils/datadog'   // MUST be first import
// ...
app.use(transactionIdMiddleware)
app.use(helmet())
setupRoutes(app)
setupSwagger(app)
app.get('/api/health', ...)
```

### `src/utils/logger.ts` — Winston Logger
```typescript
// Format: JSON + splat
// Transport: Console only (stdout → DataDog agent picks up)
// exitOnError: false
export const logger = winston.createLogger({ ... })
```
**Never use `console.log`** — ESLint rule `"no-console": "error"` enforces this.

### `src/utils/datadog.ts` — APM Tracing
```typescript
import tracer from 'dd-trace'
tracer.init()
tracer.use('http', { blocklist: ['/api/health'] })
```

### `src/middlewares/transactionId.ts`
- Injects `req.id = uuidv4()` on every request
- Extend Express `Request` type locally (not globally)

### Controller Pattern
```typescript
export async function getExamples(_req: Request, res: Response) {
  try {
    const data = await service.getExamplesFromDatabase()
    res.json(data)
  } catch (err) {
    res.status(500).json({ error: '...' })
  }
}
```

### Service Pattern
```typescript
const prisma = new PrismaClient()   // singleton per service file

export async function createExample(payload: { ... }): Promise<object> {
  try {
    logger.info('...')
    return await prisma.example.create({ data: payload })
  } catch (error) {
    logger.error('...', error)
    throw error   // re-throw; controller handles HTTP response
  }
}
```

### Route Registration (`src/routes/index.ts`)
```typescript
export function setupRoutes(app: Express) {
  app.use('/api', exampleRoutes)
  // add feature routes here
}
```

---

## package.json Scripts

| Script | Purpose |
|--------|---------|
| `dev` | nodemon watch + ts-node (development) |
| `build` | `tsc` + copy openapi.yaml to dist/ |
| `serve` | run compiled `dist/index.js` |
| `test` | jest (also runs on pre-push via husky) |
| `test:coverage` | jest with coverage report |
| `lint` | ESLint on src/ |
| `lint:fix` | ESLint fix (runs on pre-commit via husky) |
| `prettier` | format src/**/*.ts |
| `knip` | dead-code / unused deps analysis |
| `migrate:up` | `prisma migrate dev` |
| `prisma:generate` | generate Prisma client |
| `generate-api-types` | openapi-typescript → `src/api-types.ts` |

---

## tsconfig.json Key Settings

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "outDir": "./dist",
    "rootDir": ".",
    "strict": true,
    "esModuleInterop": true,
    "sourceMap": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true
  },
  "include": ["src/**/*.ts", "tests/**/*.ts", "scripts"]
}
```

---

## ESLint Rules (highlights)

```json
{
  "no-console": "error",
  "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
  "@typescript-eslint/no-floating-promises": "error",
  "@typescript-eslint/no-misused-promises": "error",
  "typesafe/no-await-without-trycatch": "warn"
}
```
Extends: `eslint:recommended`, `@typescript-eslint/recommended`, `prettier`

---

## Prettier Config

```json
{ "singleQuote": true, "printWidth": 80, "tabWidth": 2, "trailingComma": "es5", "semi": false }
```

---

## Test Configuration (Jest)

- Preset: `ts-jest`, env: `node`
- Pattern: `tests/**/*.spec.ts`
- **Coverage threshold: 70%** (branches / functions / lines / statements)
- Module alias: `src/` → `<rootDir>/src/`

---

## Dockerfile Pattern (multi-stage)

```dockerfile
# Stage 1: build
FROM node:24-alpine AS build
RUN npm ci && npx prisma generate && npm run build

# Stage 2: production
FROM node:24-alpine AS production
ENV NODE_ENV=production
RUN npm ci --only=production
# non-root user
RUN adduser -D -u 1001 nodeuser && chown -R nodeuser /usr/src/app
USER nodeuser
EXPOSE 3000
HEALTHCHECK CMD curl --fail http://localhost:3000/api/health || exit 1
CMD ["node", "dist/src/index.js"]
```

Key points: non-root user (uid 1001), healthcheck on `/api/health`, curl installed via `apk`.

---

## Azure Pipelines (dev flow)

Stages in `azure-pipeline-dev.yaml`:
1. **CodeQuality** — knip dead-code analysis, publishes `knip-results` artifact
2. **Build** — Docker build + push to GCP Artifact Registry (tags: `dev-latest`, `dev-<commit>`)
3. **Update** — runs `updateK8smanifest.sh` on self-hosted pool to update GitOps manifest repo

Variables pattern:
```yaml
containerRegistryConection: 'app-nonprd-registry'
GCP_PROJECT_ID: 'app-nonprod-project'
GCP_REPO:        'app-nonprod-ar'
pool:            'GCP-NONPRD-POOL'
```

Separate pipeline files per environment: `dev`, `qa`, `uat`, `prod`.

---

## Load Test (k6)

File: `scripts/loadTest.js`

```javascript
export let options = {
  stages: [
    { duration: '30s', target: 20 },
    { duration: '1m',  target: 20 },
    { duration: '30s', target: 0  },
  ],
  thresholds: {
    http_req_duration: ['p(95)<5000'],   // p95 < 5 s
    http_req_failed:   ['rate<0.05'],    // error rate < 5%
  },
}
```

---

## Environment Variables (`.env.example`)

```
DATABASE_URL="postgresql://user:password@postgres-service:5432/dbname?schema=public"
DATADOG_RUM_ENV=local
COMMIT_ID=
NODE_ENV=development
APP_PORT=3000
```

---

## Compliance Checklist Implications from Boilerplate

| Check | How boilerplate satisfies it |
|-------|------------------------------|
| Helmet security headers | `app.use(helmet())` in index.ts |
| CORS whitelist | Explicit `allowedOrigins` array; dev-only |
| Structured JSON logging | Winston → stdout JSON |
| DataDog APM | dd-trace init as first import |
| Transaction ID | `transactionIdMiddleware` → `req.id` |
| Graceful shutdown | SIGTERM/SIGINT handlers |
| Non-root Docker user | `adduser -u 1001 nodeuser` |
| Health check endpoint | `GET /api/health` |
| Coverage gate | Jest 70% threshold |
| Dead code detection | knip in CI (CodeQuality stage) |
| GitOps deployment | `updateK8smanifest.sh` + Azure pipeline |
| Load test script | k6 in `scripts/loadTest.js` |
