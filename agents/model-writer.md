# Agent: Model Writer

Define como os dados sao organizados no banco de dados.
Qualquer arquivo em `src/models/` ou `prisma/`.

## Regras (Prisma)

### Estrutura do schema

```prisma
// prisma/schema.prisma

model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  image     String?
  tasks     Task[]
  clients   Client[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Client {
  id        String   @id @default(cuid())
  name      String
  email     String?
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  tasks     Task[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([userId])
}

model Task {
  id          String    @id @default(cuid())
  title       String
  description String?
  priority    Priority  @default(MEDIUM)
  deadline    DateTime?
  completed   Boolean   @default(false)
  clientId    String
  client      Client    @relation(fields: [clientId], references: [id], onDelete: Cascade)
  userId      String
  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([userId])
  @@index([clientId])
  @@index([deadline])
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}
```

### Principios

1. **IDs com `cuid()`** — mais seguro que auto-increment, funciona em distributed systems
2. **`createdAt` e `updatedAt` em todo model** — auditoria basica
3. **Indexes nas foreign keys** — performance em queries com filtros
4. **`onDelete: Cascade`** — definir comportamento de delecao explicito
5. **Enums para valores fixos** — Priority, Status, Role
6. **Relations explicitas** — sempre definir os dois lados da relacao

### Migrations

```bash
# Criar migration
npx prisma migrate dev --name [descricao-curta]

# Exemplos de nomes
npx prisma migrate dev --name add-task-model
npx prisma migrate dev --name add-priority-enum
npx prisma migrate dev --name add-deadline-index
```

### Prisma Client

```typescript
// src/lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }

export const prisma = globalForPrisma.prisma || new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### O que NAO fazer

- Nao usar `autoincrement()` para IDs (usar `cuid()` ou `uuid()`)
- Nao esquecer indexes em foreign keys
- Nao criar migrations com nomes genericos ("update", "fix")
- Nao alterar migrations ja aplicadas em producao
- Nao colocar logica de negocio no schema (isso e da action)

---

## Prompt de Execucao

Quando o /execute delegar a criacao/alteracao do modelo, montar o prompt assim:

```
[Crie o model / Adicione ao schema] [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Descricao: [o que o model representa]

## Campos
| Campo | Tipo | Obrigatorio | Default | Descricao |
|---|---|---|---|---|
| [campo] | [tipo] | [sim/nao] | [default] | [descricao] |

## Relacoes
- [Model] → [outro Model]: [tipo de relacao] — [onDelete behavior]

## Indexes
- [campos que precisam de index e por que]

## Enums necessarios
- [EnumName]: [VALOR1, VALOR2, ...]

## Migration
- Nome sugerido: [descricao-curta]

## Modelo de dados do brief (se existe)
- Referencia: docs/brief/data-model.mermaid

## Validacao
- Modelo existente: [se ja existe, o que modificar vs criar]
```
