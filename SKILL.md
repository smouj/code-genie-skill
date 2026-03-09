---
name: code-genie
version: 1.2.3
description: AI-powered code generation from natural language specifications with context-aware completions
author: OpenClaw Team
tags: [code-generation, AI-assistant, natural-language, programming, automation]
dependencies:
  - node >= 18.0.0
  - python >= 3.10 (optional, for Python generation)
  - git
  - ripgrep (rg)
  -active: true
last_updated: 2026-03-09
---

# Code Genie

AI-powered code generation assistant that transforms natural language specifications into production-ready code across multiple languages and frameworks.

## Purpose

Real-world use cases:
- Generate TypeScript/React components from UI descriptions
- Create REST API endpoints from specification documents
- Write database migration scripts from schema changes
- Build Python data processing pipelines from transformation requirements
- Generate test suites from function signatures and edge cases
- Create Docker configurations and deployment scripts
- Refactor legacy code with modern patterns
- Generate CLI tools with argument parsing
- Build GraphQL resolvers and schema definitions
- Create configuration files for CI/CD pipelines

## Scope

Supported languages: TypeScript, JavaScript, Python, Go, Rust, Java, C#, SQL, YAML, JSON, Dockerfile, Terraform HCL.

Core commands:

```bash
# Generate code from description
code-genie generate "create a React component with a table that supports pagination, sorting, and filtering"

# Refactor existing code
code-genie refactor ./src/components/OldComponent.tsx --target-pattern "functional component with hooks"

# Generate tests
code-genie test ./src/utils/math.ts --coverage 80 --framework jest

# Generate complete API
code-genie api ./specs/user-api.yaml --output ./src/api --language typescript

# Generate configuration
code-genie config "Next.js app with TypeScript, ESLint, TailwindCSS, and Jest" --output .

# Interactive mode
code-genie chat --context ./src --model claude-3.5-sonnet

# Batch generation
code-genie batch ./todos/ --output ./generated --dry-run

# Validate generated code
code-genie validate ./generated --format prettier --lint eslint --test jest
```

## Work Process

### 1. Context Discovery
```bash
code-genie analyze ./src --depth 2 --include "**/*.ts" --exclude "node_modules,dist"
```
- Scans project structure
- Detects language/framework
- Identifies patterns and conventions
- Extracts existing type definitions
- Maps imports and dependencies

### 2. Specification Parsing
Accepts inputs:
- Natural language descriptions
- YAML/JSON specifications
- Existing code files (for refactoring)
- API schemas (OpenAPI, GraphQL SDL)
- Database schemas

```bash
code-genie spec ./specs/payment.yaml --format openapi --validate
```

### 3. Code Generation Pipeline

Step 1: Plan generation strategy
```bash
code-genie plan "authentication middleware with JWT refresh" --components AuthService,TokenValidator,RefreshStrategy
```

Step 2: Generate with constraints
```bash
code-genie generate ./prompts/feature.txt \
  --constraints "use async/await, validate inputs with zod, error handling middleware" \
  --max-tokens 4000 \
  --temperature 0.3 \
  --output ./src/features/auth
```

Step 3: Apply project conventions
```bash
code-genie format ./generated --config . prettier.config.js --eslint .eslintrc.js
```

### 4. Integration
```bash
code-genie inject ./generated/auth-service.ts --target ./src/index.ts --strategy merge
```
- Analyzes target file AST
- Identifies optimal insertion point
- Preserves existing code structure
- Updates imports automatically
- Creates backup before modification

### 5. Validation
```bash
code-genie test ./generated --framework vitest --coverage --watch
code-genie lint ./generated --fix
code-genie typecheck ./generated --strict
code-genie build ./generated --skip-dependencies
```

## Golden Rules

1. **Never generate secrets or credentials** - All sensitive values must come from environment variables
2. **Always validate type safety** - Generated code must pass strict TypeScript compilation
3. **Preserve project conventions** - Match existing code style, naming, and patterns
4. **Generate tests alongside code** - 100% of generated functionality must have test coverage
5. **Document public APIs** - Every exported function/class must have JSDoc/TSDoc
6. **Handle errors explicitly** - No bare `any` types, no unhandled promise rejections
7. **Respect existing architecture** - Don't introduce circular dependencies or tight coupling
8. **Generate incremental changes** - Create new files by default, modify existing only with explicit flag
9. **Sanitize inputs** - Escape all user-provided strings to prevent injection
10. **Verify before commit** - All generated code must be manually reviewed in PR workflow

## Examples

### Example 1: Generate React component
**Input prompt:**
```bash
code-genie generate "DataTable component with columns configurable via props, supports sorting, pagination with pageSize selector, row selection checkboxes, and sticky header. Use Material-UI v5 and TypeScript"
```

**Generated output (abbreviated):**
```typescript
// src/components/DataTable.tsx
import React, { useState, useMemo } from 'react';
import {
  Table, TableBody, TableCell, TableContainer, TableHead,
  TablePagination, TableRow, TableSortLabel, Checkbox,
  Paper, Box, TextField, Select, MenuItem
} from '@mui/material';

export interface Column<T> {
  id: keyof T;
  label: string;
  numeric?: boolean;
  render?: (value: any, row: T) => React.ReactNode;
}

interface DataTableProps<T> {
  data: T[];
  columns: Column<T>[];
  pageSize?: number;
  selectable?: boolean;
  onSelectionChange?: (selected: T[]) => void;
}

export function DataTable<T extends Record<string, any>>({
  data,
  columns,
  pageSize = 10,
  selectable = false,
  onSelectionChange
}: DataTableProps<T>) {
  const [page, setPage] = useState(0);
  const [order, setOrder] = useState<'asc'|'desc'>('asc');
  const [orderBy, setOrderBy] = useState<keyof T>(columns[0].id);
  const [selected, setSelected] = useState<Set<number>>(new Set());

  // Full implementation with sorting, pagination, selection
  // ...
}

export default DataTable;
```

### Example 2: API endpoint generation
**Input spec (specs/users.yaml):**
```yaml
endpoint: GET /api/users
description: List users with pagination and filtering
params:
  - name: page
    type: integer
    default: 1
  - name: limit
    type: integer
    default: 20
  - name: role
    type: string
    enum: [admin, user, moderator]
response:
  type: array
  items:
    $ref: '#/components/schemas/User'
```

**Command:**
```bash
code-genie api ./specs/users.yaml --output ./src/api --language typescript --framework express
```

**Generated output:**
```typescript
// src/api/routes/users.ts
import { Request, Response, Router } from 'express';
import { z } from 'zod';
import { User, UserRole } from '../models/User';

const querySchema = z.object({
  page: z.number().int().positive().default(1),
  limit: z.number().int().positive().max(100).default(20),
  role: z.nativeEnum(UserRole).optional()
});

export const router = Router();

router.get('/', async (req: Request, res: Response) => {
  try {
    const { page, limit, role } = querySchema.parse(req.query);
    
    const offset = (page - 1) * limit;
    const where = role ? { role } : undefined;
    
    const users = await User.findMany({
      where,
      skip: offset,
      take: limit,
      orderBy: { createdAt: 'desc' }
    });
    
    const total = await User.count({ where });
    
    res.json({
      data: users,
      pagination: {
        page,
        limit,
        total,
        pages: Math.ceil(total / limit)
      }
    });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return res.status(400).json({ errors: error.errors });
    }
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

### Example 3: Refactor to modern patterns
**Command:**
```bash
code-genie refactor ./src/legacy/processor.js \
  --target-pattern "ES6 modules, async/await, error boundaries" \
  --output ./src/modern \
  --preserve-tests
```

**Before:**
```javascript
function processData(data, callback) {
  var results = [];
  for (var i = 0; i < data.length; i++) {
    results.push(transform(data[i]));
  }
  callback(null, results);
}
```

**After:**
```typescript
// src/modern/dataProcessor.ts
export async function processData(data: InputType[]): Promise<OutputType[]> {
  return Promise.all(
    data.map(item => transform(item))
  );
}

// With streaming for large datasets
export async function* processDataStream(
  data: AsyncIterable<InputType>
): AsyncIterable<OutputType> {
  for await (const item of data) {
    yield await transform(item);
  }
}
```

### Example 4: Generate test suite
**Command:**
```bash
code-genie test ./src/utils/crypto.ts \
  --framework vitest \
  --coverage 90 \
  --edge-cases "empty input, invalid format, overflow, concurrent calls" \
  --mocks "crypto.randomBytes"
```

**Generated test:**
```typescript
// src/utils/crypto.test.ts
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { hashPassword, verifyPassword, generateToken } from './crypto';

describe('crypto utilities', () => {
  describe('hashPassword', () => {
    it('hashes password with bcrypt', async () => {
      const password = 'SecurePass123!';
      const hash = await hashPassword(password);
      expect(hash).toBeDefined();
      expect(hash).not.toBe(password);
      expect(hash.startsWith('$2')).toBe(true);
    });

    it('rejects empty password', async () => {
      await expect(hashPassword('')).rejects.toThrow('password required');
    });

    it('generates different hashes for same password', async () => {
      const hash1 = await hashPassword('test');
      const hash2 = await hashPassword('test');
      expect(hash1).not.toBe(hash2);
    });
  });

  describe('verifyPassword', () => {
    it('verifies correct password', async () => {
      const password = 'MyPass123';
      const hash = await hashPassword(password);
      expect(await verifyPassword(password, hash)).toBe(true);
    });

    it('rejects incorrect password', async () => {
      const hash = await hashPassword('correct');
      expect(await verifyPassword('wrong', hash)).toBe(false);
    });
  });
});
```

### Example 5: Docker configuration
**Command:**
```bash
code-genie config "Node.js 18 Express app with TypeScript, using PostgreSQL, Redis for sessions, behind Nginx, with health checks on /health" \
  --output . \
  --env production,staging \
  --expose 3000 \
  --volume ./data:/app/data
```

**Generated docker-compose.yml:**
```yaml
version: '3.8'
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    environment:
      - NODE_ENV=${NODE_ENV:-production}
      - DATABASE_URL=${DATABASE_URL}
      - REDIS_URL=${REDIS_URL}
      - SESSION_SECRET=${SESSION_SECRET}
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - ./logs:/app/logs
      - ./uploads:/app/uploads
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s

  postgres:
    image: postgres:15-alpine
    environment:
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB:-app}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s

  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./ssl:/etc/nginx/ssl:ro
    depends_on:
      - app

volumes:
  postgres_data:
  redis_data:
```

## Dependencies

**Required:**
- Node.js 18+ runtime
- `@openclaw/code-genie` npm package
- Git CLI (for repository operations)
- ripgrep (`rg`) for code search

**Optional language-specific:**
- Python 3.10+ with `black`, `mypy`, `pytest`
- Go 1.21+ with `gofmt`, `golangci-lint`
- Rust 1.70+ with `rustfmt`, `clippy`
- Java 17+ with Maven/Gradle
- .NET 6+ SDK

**Installation:**
```bash
npm install -g @openclaw/code-genie
# or
code-genie install --standalone
```

## Environment Variables

```bash
# API key for AI provider (required)
OPENCLAW_API_KEY=sk-...

# AI provider selection (openai, anthropic, local)
OPENCLAW_MODEL_PROVIDER=anthropic

# Model specification
OPENCLAW_MODEL=claude-3.5-sonnet
OPENCLAW_MAX_TOKENS=4000
OPENCLAW_TEMPERATURE=0.3

# Project context
OPENCLAW_PROJECT_ROOT=.
OPENCLAW_IGNORE_PATTERNS=node_modules,dist,build,.git

# Validation
OPENCLAW_STRICT_MODE=true
OPENCLAW_RUN_LINT_ON_GENERATE=true
OPENCLAW_RUN_TESTS_ON_GENERATE=false
OPENCLAW_MAX_FILE_SIZE=500KB

# Git integration
OPENCLAW_GIT_AUTO_COMMIT=false
OPENCLAW_GIT_BRANCH_PREFIX=code-genie/
OPENCLAW_COMMIT_MESSAGE_TEMPLATE="feat: generated {type} for {description}"

# Logging
OPENCLAW_LOG_LEVEL=info
OPENCLAW_LOG_FILE=./code-genie.log
```

## Verification Steps

After generation, run:

```bash
# 1. Type checking
npx tsc --noEmit --strict ./generated

# 2. Linting
npx eslint ./generated --max-warnings 0

# 3. Testing (if tests exist)
npm test -- ./generated

# 4. Build verification
npm run build --if-present

# 5. Import validation (for TypeScript)
npx dpdm ./generated --warning false --missing false --circular true

# 6. Dependency check (verify no external packages added without approval)
code-genie deps ./generated --whitelist "react,@mui/material,express,zod"

# 7. Security scan (if configured)
npx audit-ci --moderate
```

**Automated verification:**
```bash
code-genie generate "..." --verify all
# Runs: typecheck + lint + test + build + deps + security
```

## Troubleshooting

### Issue: "Context window exceeded"
**Cause:** Project has too many files for analysis.
**Fix:**
```bash
code-genie analyze ./src --max-files 100 --patterns "**/*.ts,**/*.tsx" --exclude "test*,spec*"
```

### Issue: "Unable to detect framework"
**Cause:** Ambiguous project structure.
**Fix:** Explicitly specify:
```bash
code-genie generate "..." --framework react --language typescript --config ./tsconfig.json
```

### Issue: "Generation failed: API rate limit"
**Cause:** Too many requests to AI provider.
**Fix:** Add retry logic or batch work:
```bash
code-genie batch ./prompts/ --parallel 2 --rate-limit 5/min
```

### Issue: "Generated code violates style guide"
**Fix:** Ensure formatter config exists:
```bash
# Create .prettierrc if missing
code-genie init --prettier --eslint
# Rerun with auto-format
code-genie generate "..." --format --fix
```

### Issue: "Circular dependencies detected"
**Fix:** Run dependency analysis and refactor:
```bash
npx dpdm ./generated
# Or let code-genie auto-fix
code-genie generate "..." --fix-circular
```

### Issue: "Tests failing for generated code"
**Debug:**
```bash
# Run tests with coverage to see what's missing
npm test -- --coverage ./generated
# Regenerate with edge cases specified
code-genie test ./generated/component.tsx --edge-cases "null props, async errors, race conditions"
```

### Issue: "Generated code uses deprecated APIs"
**Fix:** Specify version constraints:
```bash
code-genie generate "..." --target "node:18,react:18,mui:v5" --no-deprecated
```

### Performance Tips:
1. Cache context: `code-genie cache --enable` (speeds up repeated runs)
2. Use local models: `code-genie serve --model codellama:34b` for offline work
3. Incremental generation: `code-genie generate --diff` to only update changed files
4. Parallel generation: `code-genie batch ./prompts --workers 4`

## Rollback

### Automatic backups
Every operation creates a backup in `.code-genie/backups/`:
```bash
code-genie generate "..." --backup
# Backup location: .code-genie/backups/20260309_143022/
```

### Manual rollback
```bash
# List available rollback points
code-genie rollback --list

# Revert last operation
code-genie rollback --last

# Revert to specific timestamp
code-genie rollback --to 20260309_143022

# Revert specific file
code-genie rollback ./src/generated/Component.tsx --to original
```

### Git-based rollback
```bash
# If --git-auto-commit was used
git log --oneline -- code-genie/
git checkout <commit-hash> -- ./src/generated/

# Or create reversal PR
code-genie rollback --create-pr --title "Revert: generated auth middleware"
```

### Complete cleanup
```bash
# Remove all generated files (from manifest)
code-genie clean --generated-only

# Full reset (removes cache and backups too)
code-genie reset --all --confirm
```

## Interactive Mode

```bash
code-genie chat
```

**Session example:**
```
> create a React hook for debounced search
Generating useDebouncedSearch.ts...

> add TypeScript generics for the debounced value
Updated with generic type parameter T...

> also add cancellation on unmount
Added cleanup function with AbortController...

> write tests for edge cases
Generated useDebouncedSearch.test.ts with race condition tests...

> looks good, save to src/hooks/
Files saved. Run 'code-genie validate src/hooks/useDebouncedSearch.ts' to verify.
```

## Advanced Features

### Custom templates
```bash
code-genie template install ./templates/my-company/
code-genie generate "API client" --template my-company/http-client
```

### Project-specific rules
Create `.code-genierc.json`:
```json
{
  "rules": {
    "typescript": {
      "strict": true,
      "noAny": true,
      "preferOptional": false
    },
    "testing": {
      "required": true,
      "framework": "vitest",
      "coverage": 85
    },
    " imports": {
      "grouping": ["react", "third-party", "internal", "relative"]
    }
  }
}
```

### CI/CD Integration
```yaml
# .github/workflows/code-genie.yml
name: Validate Generated Code
on:
  pull_request:
    paths:
      - 'generated/**'

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Validate generated code
        run: |
          code-genie validate ./generated \
            --typecheck \
            --lint \
            --test \
            --build
```

## Support

- Docs: https://docs.openclaw.dev/code-genie
- Issues: https://github.com/openclaw/code-genie/issues
- Discord: #code-genie channel in OpenClaw community
```