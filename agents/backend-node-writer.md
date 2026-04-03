# Agent: Backend Node Writer

Escreve codigo backend em Node.js/TypeScript — controllers, services, middlewares, rotas Express/Fastify.
Usar quando o projeto tem backend separado (nao Next.js API Routes).
Qualquer arquivo em `src/server/`, `src/api/`, `src/services/`, `src/middleware/`.

## Quando usar este agente (vs route-writer)

| Usar backend-node-writer | Usar route-writer |
|---|---|
| Backend Node.js separado (Express, Fastify, Hono) | Next.js API Routes |
| Microservicos | Monolito Next.js |
| API consumida por multiplos clients (web, mobile, CLI) | API so para o frontend Next.js |

## Arquitetura padrao

```
src/
├── server/
│   ├── index.ts               ← Entry point (Express app)
│   ├── routes/
│   │   ├── index.ts           ← Router principal
│   │   ├── auth.routes.ts     ← Rotas de auth
│   │   └── task.routes.ts     ← Rotas de tasks
│   ├── controllers/
│   │   ├── auth.controller.ts ← Logica de request/response
│   │   └── task.controller.ts
│   ├── services/
│   │   ├── auth.service.ts    ← Logica de negocio
│   │   └── task.service.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts  ← Verificacao de JWT
│   │   ├── validate.middleware.ts ← Validacao Zod
│   │   └── error.middleware.ts ← Error handler global
│   └── utils/
│       ├── errors.ts          ← Classes de erro customizadas
│       └── response.ts        ← Helpers de resposta padronizada
```

## Separacao de responsabilidades

### Route — define o endpoint

```typescript
// src/server/routes/task.routes.ts
import { Router } from 'express'
import { TaskController } from '../controllers/task.controller'
import { authMiddleware } from '../middleware/auth.middleware'
import { validate } from '../middleware/validate.middleware'
import { CreateTaskSchema } from '../schemas/task.schema'

const router = Router()

router.use(authMiddleware) // todas as rotas precisam de auth

router.get('/', TaskController.list)
router.post('/', validate(CreateTaskSchema), TaskController.create)
router.get('/:id', TaskController.getById)
router.patch('/:id', validate(UpdateTaskSchema), TaskController.update)
router.delete('/:id', TaskController.delete)

export { router as taskRoutes }
```

### Controller — orquestra request/response

```typescript
// src/server/controllers/task.controller.ts
import { Request, Response, NextFunction } from 'express'
import { TaskService } from '../services/task.service'

export class TaskController {
  static async create(req: Request, res: Response, next: NextFunction) {
    try {
      const task = await TaskService.create({
        ...req.body,
        userId: req.user.id, // injetado pelo auth middleware
      })
      return res.status(201).json({ data: task })
    } catch (error) {
      next(error) // error middleware trata
    }
  }

  static async list(req: Request, res: Response, next: NextFunction) {
    try {
      const { page = 1, limit = 20, clientId } = req.query
      const result = await TaskService.list({
        userId: req.user.id,
        page: Number(page),
        limit: Number(limit),
        clientId: clientId as string | undefined,
      })
      return res.json({ data: result.items, total: result.total })
    } catch (error) {
      next(error)
    }
  }
}
```

### Service — logica de negocio

```typescript
// src/server/services/task.service.ts
import { prisma } from '../lib/prisma'
import { AppError } from '../utils/errors'

export class TaskService {
  static async create(data: CreateTaskInput) {
    if (data.deadline && data.deadline < new Date()) {
      throw new AppError('Deadline deve ser no futuro', 400)
    }

    return prisma.task.create({ data })
  }

  static async list({ userId, page, limit, clientId }: ListTasksInput) {
    const where = { userId, ...(clientId && { clientId }) }
    const [items, total] = await Promise.all([
      prisma.task.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { createdAt: 'desc' },
        include: { client: { select: { name: true } } },
      }),
      prisma.task.count({ where }),
    ])
    return { items, total }
  }
}
```

### Middleware — validacao com Zod

```typescript
// src/server/middleware/validate.middleware.ts
import { Request, Response, NextFunction } from 'express'
import { ZodSchema } from 'zod'

export function validate(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    const result = schema.safeParse(req.body)
    if (!result.success) {
      return res.status(400).json({
        error: 'Validation failed',
        details: result.error.flatten().fieldErrors,
      })
    }
    req.body = result.data
    next()
  }
}
```

### Error handler global

```typescript
// src/server/middleware/error.middleware.ts
import { Request, Response, NextFunction } from 'express'
import { AppError } from '../utils/errors'

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  console.error(`[${req.method}] ${req.path}:`, err.message)

  if (err instanceof AppError) {
    return res.status(err.statusCode).json({ error: err.message })
  }

  return res.status(500).json({ error: 'Internal server error' })
}
```

## Principios

1. **Route → Controller → Service → Database** — cada camada tem responsabilidade unica
2. **Controller nao tem logica de negocio** — so orquestra request/response
3. **Service nao conhece HTTP** — nao recebe req/res, recebe dados tipados
4. **Validacao no middleware** — request ja chega validado no controller
5. **Error handler global** — nunca retornar stack trace para o client
6. **Tipos em tudo** — Request, Response tipados, DTOs para inputs/outputs

## O que NAO fazer

- Nao colocar logica de negocio no controller
- Nao acessar `req`/`res` no service
- Nao retornar erro generico sem log (sempre logar o erro real)
- Nao esquecer tratamento de erro async (try/catch ou wrapper)
- Nao misturar responsabilidades (um service por entidade)

---

## Prompt de Execucao

Quando o /execute delegar a criacao de backend Node.js, montar o prompt assim:

```
Crie [route/controller/service/middleware] [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Framework: [Express / Fastify / Hono — de stack.md]
- Descricao: [o que esse modulo faz]

## Camada e responsabilidade
- Route: [path, metodos HTTP]
- Controller: [o que orquestra]
- Service: [logica de negocio]
- Middleware: [validacao/auth]

## Endpoint(s)
| Metodo | Path | Descricao | Auth |
|---|---|---|---|
| [GET/POST/etc.] | [/api/...] | [descricao] | [sim/nao] |

## Validacao (Zod schema)
- [campo: tipo — regras]

## Banco de dados
- [operacao em qual tabela]
- ORM: [Prisma / Drizzle / Knex — de stack.md]

## Regras de negocio (de .claude/docs/business-rules.md)
- [regra relevante]

## Testes esperados
- Service: [cenarios unitarios]
- Route: [cenarios de integracao]
```
