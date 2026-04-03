# Agent: Route Writer

Cria API Routes — endpoints para comunicacao entre frontend e backend,
ou para webhooks de servicos externos. Qualquer arquivo em `src/app/api/`.

## Regras

### Estrutura de uma API Route

```typescript
// src/app/api/tasks/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { prisma } from '@/lib/prisma'
import { auth } from '@/lib/auth'
import { z } from 'zod'

// GET /api/tasks?page=1&limit=20&clientId=xxx
export async function GET(req: NextRequest) {
  const session = await auth()
  if (!session?.user?.id) {
    return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  }

  const { searchParams } = new URL(req.url)
  const page = parseInt(searchParams.get('page') || '1')
  const limit = parseInt(searchParams.get('limit') || '20')
  const clientId = searchParams.get('clientId')

  const where = {
    userId: session.user.id,
    ...(clientId && { clientId }),
  }

  const [tasks, total] = await Promise.all([
    prisma.task.findMany({
      where,
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' },
      include: { client: { select: { name: true } } },
    }),
    prisma.task.count({ where }),
  ])

  return NextResponse.json({ data: tasks, total, page, limit })
}
```

### Route com parametro dinamico

```typescript
// src/app/api/tasks/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  // ...
}

export async function PATCH(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  // ...
}

export async function DELETE(
  req: NextRequest,
  { params }: { params: { id: string } }
) {
  // ...
}
```

### Principios

1. **Preferir Server Actions para mutacoes** — API Routes so quando necessario (webhooks, integracao externa, consumo por app mobile)
2. **Auth check em toda rota** — verificar sessao antes de qualquer operacao
3. **Validacao com Zod** — validar body e query params
4. **Paginacao** — nunca retornar listas sem limit
5. **Status codes corretos** — 200, 201, 400, 401, 404, 500
6. **Respostas padronizadas** — `{ data }` para sucesso, `{ error }` para erro

### Quando usar API Route vs Server Action

| Usar API Route | Usar Server Action |
|---|---|
| Webhooks (Stripe, etc.) | Forms e mutacoes de UI |
| Consumo por app mobile/externo | Operacoes disparadas por interacao do usuario |
| Upload de arquivos complexos | CRUD simples |
| Streaming / SSE | Qualquer mutacao com revalidation |

### O que NAO fazer

- Nao criar API Routes para coisas que Server Actions resolvem
- Nao esquecer auth check
- Nao retornar dados sem paginacao
- Nao usar `any` nos types de request/response
- Nao logar dados sensiveis

---

## Prompt de Execucao

Quando o /execute delegar a criacao de uma API Route, montar o prompt assim:

```
Crie a API Route [METODO] [PATH].

## Contexto
- Issue: [issue-id]
- Descricao: [o que o endpoint faz]
- Motivo de ser Route e nao Action: [webhook / consumo externo / mobile / etc.]

## Request
- Metodo: [GET/POST/PATCH/DELETE]
- Query params: [param: tipo — descricao]
- Body: [campo: tipo — descricao]
- Headers especiais: [se houver]

## Response
- Sucesso (200/201): { data: [estrutura] }
- Erro validacao (400): { error: [mensagem] }
- Nao autenticado (401): { error: 'Unauthorized' }
- Nao encontrado (404): { error: 'Not found' }

## Auth
- Requer autenticacao: [sim/nao]
- Tipo: [session check / API key / webhook signature]

## Banco
- [operacao em qual tabela]
- Paginacao: [sim/nao — se sim, params page/limit]

## Testes esperados
- Request valido → response esperado
- Request invalido → erro esperado
- Sem auth → 401
```
