# Agent: Action Writer

Escreve Server Actions — codigo que processa acoes do usuario no servidor
(salvar dados, deletar, atualizar). Qualquer arquivo em `src/actions/`.

## Regras

### Estrutura de uma Server Action

```typescript
// src/actions/task.ts
'use server'

import { z } from 'zod'
import { prisma } from '@/lib/prisma'
import { auth } from '@/lib/auth'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'

// 1. Schema de validacao
const CreateTaskSchema = z.object({
  title: z.string().min(1, 'Titulo obrigatorio'),
  description: z.string().optional(),
  clientId: z.string().min(1, 'Selecione um cliente'),
  priority: z.enum(['LOW', 'MEDIUM', 'HIGH']),
  deadline: z.string().optional().transform(val => val ? new Date(val) : undefined),
})

// 2. Type inferido do schema
type CreateTaskInput = z.infer<typeof CreateTaskSchema>

// 3. Action com tratamento de erro padronizado
export async function createTask(input: CreateTaskInput) {
  // Auth check
  const session = await auth()
  if (!session?.user?.id) {
    return { error: 'Nao autenticado' }
  }

  // Validacao
  const parsed = CreateTaskSchema.safeParse(input)
  if (!parsed.success) {
    return { error: parsed.error.flatten().fieldErrors }
  }

  // Validacao de negocio
  if (parsed.data.deadline && parsed.data.deadline < new Date()) {
    return { error: { deadline: ['Deadline deve ser no futuro'] } }
  }

  try {
    // Persistencia
    const task = await prisma.task.create({
      data: {
        ...parsed.data,
        userId: session.user.id,
      },
    })

    // Revalidar cache
    revalidatePath('/dashboard')

    return { data: task }
  } catch (error) {
    console.error('Failed to create task:', error)
    return { error: 'Erro ao criar tarefa' }
  }
}
```

### Principios

1. **Sempre `'use server'`** no topo do arquivo
2. **Validar com Zod** — schema define formato, action valida
3. **Auth check primeiro** — verificar sessao antes de qualquer operacao
4. **Retornar `{ data }` ou `{ error }`** — nunca throw (o hook trata o retorno)
5. **`revalidatePath`** — sempre revalidar paginas afetadas apos mutacao
6. **Try/catch** — nunca deixar erro vazar para o cliente
7. **Log de erro** — `console.error` com contexto para debug

### Pattern de retorno

```typescript
// Sucesso
return { data: resultado }

// Erro de validacao (campo especifico)
return { error: { campo: ['mensagem'] } }

// Erro generico
return { error: 'Mensagem de erro' }
```

### O que NAO fazer

- Nao fazer redirect dentro da action (deixar o hook/componente decidir)
- Nao acessar cookies/headers sem necessidade
- Nao fazer queries pesadas sem paginacao
- Nao retornar dados sensiveis (senhas, tokens internos)
- Nao misturar logica de multiplas entidades na mesma action

---

## Prompt de Execucao

Quando o /execute delegar a criacao de uma Server Action, montar o prompt assim:

```
Crie a Server Action [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Descricao: [o que a action faz, extraido do PLAN.md]

## Schema de validacao
- Campos: [campo: tipo — validacao]
- Validacoes de negocio: [regras extraidas de .claude/docs/business-rules.md]

## Operacoes
- Auth: [sim/nao — qual check]
- Banco: [CREATE/READ/UPDATE/DELETE em qual tabela]
- Revalidar: [quais paths]
- Redirect: [nao — isso e do hook]

## Cenarios (do PLAN.md)
- Sucesso: [o que retornar]
- Erro de validacao: [quais campos podem falhar e mensagens]
- Erro de servidor: [como tratar]

## Patterns existentes no projeto
- [pattern de action ja usado, extraido da pesquisa interna]

## Testes esperados
- [cenario 1: input valido → resultado esperado]
- [cenario 2: input invalido → erro esperado]
- [cenario 3: erro de servidor → tratamento esperado]
```
