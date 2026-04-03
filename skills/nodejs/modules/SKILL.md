---
name: modules
description: ES Modules and CommonJS patterns
metadata:
  tags: modules, esm, commonjs, imports, exports
---

# Node.js Modules

## Purpose

Use this skill for module system decisions, import and export patterns, dynamic loading, and modern Node.js module conventions.

## Trigger

Use this skill when:

- the task involves choosing between ES Modules and CommonJS;
- the task involves structuring imports and exports in a Node.js project;
- the task involves reviewing or improving module organization;
- the task involves dynamic imports or ESM runtime patterns.

## Inputs

Gather the minimum relevant context before acting:

- whether the project uses ES Modules or CommonJS today;
- the `package.json` module configuration;
- the file extension and import style already used in the codebase;
- whether the project targets a specific Node.js version;
- whether the task is about library code, app code, or plugin-style loading.

## Workflow

1. Inspect the existing module system before recommending changes.
2. Prefer ES Modules for new Node.js projects unless the project constraints require CommonJS.
3. Keep export style consistent within the project, with a bias toward named exports.
4. Use barrel exports only when they simplify module boundaries instead of obscuring them.
5. Use dynamic imports for conditional or delayed loading where they improve the design.
6. When ESM-specific runtime helpers are needed, align them with the Node.js version used by the project.

## Prefer ES Modules

Use ES Modules (ESM) for new projects:

```json
// package.json
{
  "type": "module"
}
```

```typescript
// Named exports (preferred)
export function processData(data: Data): Result {
  // ...
}

export const CONFIG = {
  timeout: 5000,
};

// Named imports
import { processData, CONFIG } from './utils';
```

## Barrel Exports

Use index files to simplify imports:

```typescript
// src/utils/index.ts
export { formatDate, parseDate } from './date';
export { formatCurrency } from './currency';
export { validateEmail } from './validation';

// Consumer
import { formatDate, formatCurrency } from './utils/index';
```

## Default vs Named Exports

Prefer named exports for better refactoring and tree-shaking:

```typescript
// GOOD - named exports
export function createServer(config: Config): Server {
  // ...
}

export function createClient(config: Config): Client {
  // ...
}

// AVOID - default exports
export default function createServer(config: Config): Server {
  // ...
}
```

## Dynamic Imports

Use dynamic imports for code splitting and conditional loading:

```typescript
async function loadPlugin(name: string): Promise<Plugin> {
  const module = await import(`./plugins/${name}`);
  return module.default;
}

// Conditional loading
const { default: heavy } = await import('./heavy-module');
```

## __dirname and __filename in ESM

Use `import.meta.dirname` and `import.meta.filename` (Node.js 20.11+):

```typescript
import { join } from 'node:path';

const configPath = join(import.meta.dirname, 'config.json');
const currentFile = import.meta.filename;
```

## Output

Apply this skill to produce:

- a clear module system choice for the project;
- consistent import and export patterns;
- simpler module boundaries where barrel exports help;
- correct use of dynamic imports and modern ESM runtime helpers.

## Guardrails

- Do not switch module systems without checking the existing project setup.
- Do not mix incompatible import patterns without a clear reason.
- Prefer consistency with the codebase over stylistic rewrites.
- Keep Node.js version requirements in mind when using newer ESM features.
