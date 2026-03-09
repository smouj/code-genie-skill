---
name: code-genie
version: 1.2.3
description: Generación de código impulsada por IA a partir de especificaciones en lenguaje natural con finalizaciones conscientes del contexto
author: OpenClaw Team
tags: [code-generation, AI-assistant, natural-language, programming, automation]
dependencies:
  - node >= 18.0.0
  - python >= 3.10 (optional, for Python generation)
  - git
  - ripgrep (rg)
  - active: true
last_updated: 2026-03-09
---

# Code Genie

Asistente de generación de código impulsado por IA que transforma especificaciones en lenguaje natural en código listo para producción en múltiples lenguajes y frameworks.

## Propósito

Casos de uso reales:
- Generar componentes TypeScript/React a partir de descripciones de UI
- Crear endpoints API REST a partir de documentos de especificación
- Escribir scripts de migración de base de datos a partir de cambios de esquema
- Construir pipelines de procesamiento de datos Python a partir de requisitos de transformación
- Generar suites de tests a partir de firmas de funciones y casos límite
- Crear configuraciones Docker y scripts de deployment
- Refactorizar código legacy con patrones modernos
- Generar herramientas CLI con parsing de argumentos
- Construir resolvers GraphQL y definiciones de schema
- Crear archivos de configuración para pipelines CI/CD

## Alcance

Lenguajes soportados: TypeScript, JavaScript, Python, Go, Rust, Java, C#, SQL, YAML, JSON, Dockerfile, Terraform HCL.

Comandos principales:

```bash
# Generar código desde descripción
code-genie generate "create a React component with a table that supports pagination, sorting, and filtering"

# Refactorizar código existente
code-genie refactor ./src/components/OldComponent.tsx --target-pattern "functional component with hooks"

# Generar tests
code-genie test ./src/utils/math.ts --coverage 80 --framework jest

# Generar API completa
code-genie api ./specs/user-api.yaml --output ./src/api --language typescript

# Generar configuración
code-genie config "Next.js app with TypeScript, ESLint, TailwindCSS, and Jest" --output .

# Modo interactivo
code-genie chat --context ./src --model claude-3.5-sonnet

# Generación por lotes
code-genie batch ./todos/ --output ./generated --dry-run

# Validar código generado
code-genie validate ./generated --format prettier --lint eslint --test jest
```

## Proceso de Trabajo

### 1. Descubrimiento de Contexto
```bash
code-genie analyze ./src --depth 2 --include "**/*.ts" --exclude "node_modules,dist"
```
- Escanea estructura del proyecto
- Detecta lenguaje/framework
- Identifica patrones y convenciones
- Extrae definiciones de tipos existentes
- Mapea imports y dependencias

### 2. Parseo de Especificación
Acepta inputs:
- Descripciones en lenguaje natural
- Especificaciones YAML/JSON
- Archivos de código existentes (para refactoring)
- Schemas API (OpenAPI, GraphQL SDL)
- Schemas de base de datos

```bash
code-genie spec ./specs/payment.yaml --format openapi --validate
```

### 3. Pipeline de Generación de Código

Paso 1: Planificar estrategia de generación
```bash
code-genie plan "authentication middleware with JWT refresh" --components AuthService,TokenValidator,RefreshStrategy
```

Paso 2: Generar con restricciones
```bash
code-genie generate ./prompts/feature.txt \
  --constraints "use async/await, validate inputs with zod, error handling middleware" \
  --max-tokens 4000 \
  --temperature 0.3 \
  --output ./src/features/auth
```

Paso 3: Aplicar convenciones del proyecto
```bash
code-genie format ./generated --config . prettier.config.js --eslint .eslintrc.js
```

### 4. Integración
```bash
code-genie inject ./generated/auth-service.ts --target ./src/index.ts --strategy merge
```
- Analiza AST del archivo objetivo
- Identifica punto de inserción óptimo
- Preserva estructura de código existente
- Actualiza imports automáticamente
- Crea backup antes de modificación

### 5. Validación
```bash
code-genie test ./generated --framework vitest --coverage --watch
code-genie lint ./generated --fix
code-genie typecheck ./generated --strict
code-genie build ./generated --skip-dependencies
```

## Reglas de Oro

1. **Nunca generar secretos o credenciales** - Todos los valores sensibles deben venir de variables de entorno
2. **Siempre validar seguridad de tipos** - El código generado debe pasar compilación TypeScript estricta
3. **Preservar convenciones del proyecto** - Coincidir con estilo, nomenclatura y patrones de código existentes
4. **Generar tests junto al código** - 100% de la funcionalidad generada debe tener cobertura de tests
5. **Documentar APIs públicas** - Cada función/clase exportada debe tener JSDoc/TSDoc
6. **Manejar errores explícitamente** - No hay tipos `any` desnudos, no hay promesas no manejadas
7. **Respetar arquitectura existente** - No introducir dependencias circulares o acoplamiento fuerte
8. **Generar cambios incrementales** - Crear nuevos archivos por defecto, modificar existentes solo con flag explícito
9. **Sanitizar inputs** - Escapar todas las cadenas proporcionadas por usuario para prevenir inyección
10. **Verificar antes de commit** - Todo código generado debe ser revisado manualmente en workflow de PR

## Ejemplos

### Ejemplo 1: Generar componente React
**Prompt de entrada:**
```bash
code-genie generate "DataTable component with columns configurable via props, supports sorting, pagination with pageSize selector, row selection checkboxes, and sticky header. Use Material-UI v5 and TypeScript"
```

**Salida generada (abreviada):**
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

  // Implementación completa con sorting, pagination, selection
  // ...
}

export default DataTable;
```

### Ejemplo 2: Generación de endpoint API
**Spec de entrada (specs/users.yaml):**
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

**Comando:**
```bash
code-genie api ./specs/users.yaml --output ./src/api --language typescript --framework express
```

**Salida generada:**
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

### Ejemplo 3: Refactorizar a patrones modernos
**Comando:**
```bash
code-genie refactor ./src/legacy/processor.js \
  --target-pattern "ES6 modules, async/await, error boundaries" \
  --output ./src/modern \
  --preserve-tests
```

**Antes:**
```javascript
function processData(data, callback) {
  var results = [];
  for (var i = 0; i < data.length; i++) {
    results.push(transform(data[i]));
  }
  callback(null, results);
}
```

**Después:**
```typescript
// src/modern/dataProcessor.ts
export async function processData(data: InputType[]): Promise<OutputType[]> {
  return Promise.all(
    data.map(item => transform(item))
  );
}

// Con streaming para datasets grandes
export async function* processDataStream(
  data: AsyncIterable<InputType>
): AsyncIterable<OutputType> {
  for await (const item of data) {
    yield await transform(item);
  }
}
```

### Ejemplo 4: Generar suite de tests
**Comando:**
```bash
code-genie test ./src/utils/crypto.ts \
  --framework vitest \
  --coverage 90 \
  --edge-cases "empty input, invalid format, overflow, concurrent calls" \
  --mocks "crypto.randomBytes"
```

**Test generado:**
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

### Ejemplo 5: Configuración Docker
**Comando:**
```bash
code-genie config "Node.js 18 Express app with TypeScript, using PostgreSQL, Redis for sessions, behind Nginx, with health checks on /health" \
  --output . \
  --env production,staging \
  --expose 3000 \
  --volume ./data:/app/data
```

**docker-compose.yml generado:**
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

## Dependencias

**Requeridas:**
- Node.js 18+ runtime
- Paquete npm `@openclaw/code-genie`
- Git CLI (para operaciones de repositorio)
- ripgrep (`rg`) para búsqueda de código

**Opcionales específicas por lenguaje:**
- Python 3.10+ con `black`, `mypy`, `pytest`
- Go 1.21+ con `gofmt`, `golangci-lint`
- Rust 1.70+ con `rustfmt`, `clippy`
- Java 17+ con Maven/Gradle
- .NET 6+ SDK

**Instalación:**
```bash
npm install -g @openclaw/code-genie
# o
code-genie install --standalone
```

## Variables de Entorno

```bash
# API key para proveedor de IA (requerida)
OPENCLAW_API_KEY=sk-...

# Selección de proveedor de IA (openai, anthropic, local)
OPENCLAW_MODEL_PROVIDER=anthropic

# Especificación del modelo
OPENCLAW_MODEL=claude-3.5-sonnet
OPENCLAW_MAX_TOKENS=4000
OPENCLAW_TEMPERATURE=0.3

# Contexto del proyecto
OPENCLAW_PROJECT_ROOT=.
OPENCLAW_IGNORE_PATTERNS=node_modules,dist,build,.git

# Validación
OPENCLAW_STRICT_MODE=true
OPENCLAW_RUN_LINT_ON_GENERATE=true
OPENCLAW_RUN_TESTS_ON_GENERATE=false
OPENCLAW_MAX_FILE_SIZE=500KB

# Integración Git
OPENCLAW_GIT_AUTO_COMMIT=false
OPENCLAW_GIT_BRANCH_PREFIX=code-genie/
OPENCLAW_COMMIT_MESSAGE_TEMPLATE="feat: generated {type} for {description}"

# Logging
OPENCLAW_LOG_LEVEL=info
OPENCLAW_LOG_FILE=./code-genie.log
```

## Pasos de Verificación

Después de la generación, ejecutar:

```bash
# 1. Type checking
npx tsc --noEmit --strict ./generated

# 2. Linting
npx eslint ./generated --max-warnings 0

# 3. Testing (si existen tests)
npm test -- ./generated

# 4. Verificación de build
npm run build --if-present

# 5. Validación de imports (para TypeScript)
npx dpdm ./generated --warning false --missing false --circular true

# 6. Check de dependencias (verificar que no se añaden paquetes externos sin aprobación)
code-genie deps ./generated --whitelist "react,@mui/material,express,zod"

# 7. Scan de seguridad (si está configurado)
npx audit-ci --moderate
```

**Verificación automatizada:**
```bash
code-genie generate "..." --verify all
# Ejecuta: typecheck + lint + test + build + deps + security
```

## Solución de Problemas

### Problema: "Context window exceeded"
**Causa:** El proyecto tiene demasiados archivos para análisis.
**Fix:**
```bash
code-genie analyze ./src --max-files 100 --patterns "**/*.ts,**/*.tsx" --exclude "test*,spec*"
```

### Problema: "Unable to detect framework"
**Causa:** Estructura de proyecto ambigua.
**Fix:** Especificar explícitamente:
```bash
code-genie generate "..." --framework react --language typescript --config ./tsconfig.json
```

### Problema: "Generation failed: API rate limit"
**Causa:** Demasiadas peticiones al proveedor de IA.
**Fix:** Añadir lógica de retry o trabajar por lotes:
```bash
code-genie batch ./prompts/ --parallel 2 --rate-limit 5/min
```

### Problema: "Generated code violates style guide"
**Fix:** Asegurar que existe config de formatter:
```bash
# Crear .prettierrc si falta
code-genie init --prettier --eslint
# Reejecutar con auto-formateo
code-genie generate "..." --format --fix
```

### Problema: "Circular dependencies detected"
**Fix:** Ejecutar análisis de dependencias y refactorizar:
```bash
npx dpdm ./generated
# O dejar que code-genie auto-fix
code-genie generate "..." --fix-circular
```

### Problema: "Tests failing for generated code"
**Debug:**
```bash
# Ejecutar tests con coverage para ver qué falta
npm test -- --coverage ./generated
# Regenerar con casos límite especificados
code-genie test ./generated/component.tsx --edge-cases "null props, async errors, race conditions"
```

### Problema: "Generated code uses deprecated APIs"
**Fix:** Especificar restricciones de versión:
```bash
code-genie generate "..." --target "node:18,react:18,mui:v5" --no-deprecated
```

## Consejos de Rendimiento:
1. Cachear contexto: `code-genie cache --enable` (acelera ejecuciones repetidas)
2. Usar modelos locales: `code-genie serve --model codellama:34b` para trabajo offline
3. Generación incremental: `code-genie generate --diff` para solo actualizar archivos cambiados
4. Generación paralela: `code-genie batch ./prompts --workers 4`

## Rollback

### Backups automáticos
Cada operación crea un backup en `.code-genie/backups/`:
```bash
code-genie generate "..." --backup
# Ubicación backup: .code-genie/backups/20260309_143022/
```

### Rollback manual
```bash
# Listar puntos de rollback disponibles
code-genie rollback --list

# Revertir última operación
code-genie rollback --last

# Revertir a timestamp específico
code-genie rollback --to 20260309_143022

# Revertir archivo específico
code-genie rollback ./src/generated/Component.tsx --to original
```

### Rollback basado en Git
```bash
# Si se usó --git-auto-commit
git log --oneline -- code-genie/
git checkout <commit-hash> -- ./src/generated/

# O crear PR de reversión
code-genie rollback --create-pr --title "Revert: generated auth middleware"
```

### Limpieza completa
```bash
# Eliminar todos los archivos generados (del manifiesto)
code-genie clean --generated-only

# Reset completo (elimina cache y backups también)
code-genie reset --all --confirm
```

## Modo Interactivo

```bash
code-genie chat
```

**Ejemplo de sesión:**
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

## Características Avanzadas

### Templates personalizados
```bash
code-genie template install ./templates/my-company/
code-genie generate "API client" --template my-company/http-client
```

### Reglas específicas del proyecto
Crear `.code-genierc.json`:
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
    "imports": {
      "grouping": ["react", "third-party", "internal", "relative"]
    }
  }
}
```

### Integración CI/CD
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

## Soporte

- Docs: https://docs.openclaw.dev/code-genie
- Issues: https://github.com/openclaw/code-genie/issues
- Discord: canal #code-genie en la comunidad OpenClaw
```