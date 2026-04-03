---
name: environment
description: Environment configuration and secrets management
metadata:
  tags: environment, configuration, env, secrets
---

# Environment Configuration in Node.js

## Purpose

Use this skill for environment configuration, env file loading, validation, configuration object design, and secret handling in Node.js projects.

## Trigger

Use this skill when:

- the task involves `.env` files or environment variable loading;
- the task involves validating configuration from `process.env`;
- the task involves designing or reviewing Node.js configuration patterns;
- the task involves secrets handling for Node.js applications.

## Inputs

Gather the minimum relevant context before acting:

- the current env loading approach in the project;
- whether the project already uses validation libraries such as Zod;
- the Node.js runtime assumptions in the project;
- any existing config module or env utility;
- whether the task is local development, CI, containers, or production deployment.

## Workflow

1. Inspect how the project currently loads environment variables.
2. If env files are being used, prefer Node.js built-in env file support where it fits the project.
3. If environment variables are consumed by application code, validate them before building higher-level config objects.
4. Keep configuration explicit and structured instead of scattering raw `process.env` access throughout the codebase.
5. If secrets are involved, keep them out of version control and prefer runtime injection through the infrastructure already used by the project.

## Loading Environment Files

Use Node.js built-in `--env-file` flag to load environment variables:

```bash
# Load from .env file
node --env-file=.env app.ts

# Load multiple env files (later files override earlier ones)
node --env-file=.env --env-file=.env.local app.ts
```

### Programmatic API

Use this approach as the main one for CLI applications.
Load environment files programmatically with `process.loadEnvFile()`:

```typescript
import { loadEnvFile } from 'node:process';

// Load .env from current directory
loadEnvFile();

// Load specific file
loadEnvFile('.env.local');
```

## Environment Variables Validation

Use [Zod](https://github.com/colinhacks/zod) for validation:

```typescript
import { z } from 'zod';

const EnvSchema = z.object({
  PORT: z.coerce.number().default(3000),
  DATABASE_URL: z.string().url(),
  API_KEY: z.string().min(1),
  LOG_LEVEL: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
});

type Env = z.infer<typeof EnvSchema>;

export function loadEnv(): Env {
  const result = EnvSchema.safeParse(process.env);

  if (!result.success) {
    console.error('Invalid environment variables:');
    console.error(result.error.format());
    process.exit(1);
  }

  return result.data;
}
```

## Avoid NODE_ENV

`NODE_ENV` is an antipattern. It conflates multiple concerns into a single variable:

- **Environment detection** (development vs production vs staging)
- **Behavior toggling** (verbose logging, debug features)
- **Optimization flags** (minification, caching)
- **Security settings** (strict validation, HTTPS)

This leads to problems:

```typescript
// BAD - NODE_ENV conflates concerns
if (process.env.NODE_ENV === 'development') {
  enableDebugLogging();    // logging concern
  disableRateLimiting();   // security concern
  useMockDatabase();       // infrastructure concern
}
```

Instead, use explicit environment variables for each concern:

```typescript
// GOOD - explicit variables for each concern
import { loadEnv } from './env.js';

const env = loadEnv();

const config = {
  logging: {
    level: env.LOG_LEVEL || 'info',
    pretty: env.LOG_PRETTY === 'true',
  },
  security: {
    rateLimitEnabled: env.RATE_LIMIT_ENABLED !== 'false',
    httpsOnly: env.HTTPS_ONLY === 'true',
  },
  database: {
    url: env.DATABASE_URL,
  },
};
```

This approach:

- makes configuration explicit and discoverable;
- allows fine-grained control per environment;
- avoids hidden behavior changes;
- makes testing easier by toggling individual features.

## Configuration Object Pattern

Create a typed configuration object:

```typescript
import { loadEnv } from './env.js';

const env = loadEnv();

interface Config {
  server: {
    port: number;
    host: string;
  };
  database: {
    url: string;
    poolSize: number;
  };
  auth: {
    jwtSecret: string;
    jwtExpiresIn: string;
  };
  features: {
    enableMetrics: boolean;
    enableTracing: boolean;
  };
}

export function createConfig(): Config {
  return {
    server: {
      port: env.PORT,
      host: env.HOST,
    },
    database: {
      url: env.DB_URL,
      poolSize: env.DB_POOL_SIZE,
    },
    auth: {
      jwtSecret: env.JWT_SECRET,
      jwtExpiresIn: env.JWT_EXPIRES_IN,
    },
    features: {
      enableMetrics: env.ENABLE_METRICS,
      enableTracing: env.ENABLE_TRACING,
    },
  };
}
```

## .env File Structure

Organize `.env` files properly:

```bash
# .env.example - committed to git, documents all variables
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/db
API_KEY=your-api-key-here

# .env - local development, NOT committed
PORT=3000
DATABASE_URL=postgresql://dev:dev@localhost:5432/myapp
API_KEY=sk-dev-key-123
```

## Secrets In Production

Never commit secrets to version control. Use a secrets management service appropriate for your infrastructure.

## Output

Apply this skill to produce:

- a clear env loading approach;
- validated environment variables before use;
- an explicit configuration pattern instead of ad hoc `process.env` access;
- a safe secrets handling approach appropriate to the deployment environment.

## Guardrails

- Do not commit secrets or recommend committing secrets.
- Do not introduce new env conventions that conflict with the project unless the task explicitly requires it.
- Prefer explicit configuration variables over overloaded environment modes.
- Keep env handling consistent with the project runtime and deployment model.
