# Agent: Hook Writer

Cria hooks customizados que conectam a interface com as Server Actions.
O "fio" que liga o botao ao salvamento. Qualquer arquivo em `src/hooks/`.

## Regras

### Estrutura de um hook

```typescript
// src/hooks/useTaskForm.ts
'use client'

import { useState, useTransition } from 'react'
import { createTask } from '@/actions/task'
import { useToast } from '@/hooks/useToast'
import { useRouter } from 'next/navigation'

interface UseTaskFormReturn {
  isPending: boolean
  errors: Record<string, string[]> | null
  submit: (formData: FormData) => Promise<void>
}

export function useTaskForm(): UseTaskFormReturn {
  const [isPending, startTransition] = useTransition()
  const [errors, setErrors] = useState<Record<string, string[]> | null>(null)
  const { toast } = useToast()
  const router = useRouter()

  async function submit(formData: FormData) {
    setErrors(null)

    startTransition(async () => {
      const input = {
        title: formData.get('title') as string,
        description: formData.get('description') as string,
        clientId: formData.get('clientId') as string,
        priority: formData.get('priority') as string,
        deadline: formData.get('deadline') as string,
      }

      const result = await createTask(input)

      if (result.error) {
        if (typeof result.error === 'string') {
          toast({ variant: 'error', message: result.error })
        } else {
          setErrors(result.error)
        }
        return
      }

      toast({ variant: 'success', message: 'Tarefa criada!' })
      router.push('/dashboard')
    })
  }

  return { isPending, errors, submit }
}
```

### Principios

1. **Um hook por fluxo de interacao** — useTaskForm, useClientList, useAuth
2. **Gerenciar estado de loading** — `useTransition` para pending states
3. **Gerenciar erros** — capturar e expor erros de validacao para o componente
4. **Feedback via toast** — sucesso e erro sempre notificam o usuario
5. **Navegacao apos sucesso** — redirect fica no hook, nao na action
6. **Interface tipada** — retornar tipo explicito com tudo que o componente precisa

### Pattern de retorno

O hook retorna um objeto com:
- Estado (isPending, errors, data)
- Funcoes (submit, reset, refetch)

O componente so precisa desestruturar e usar.

### O que NAO fazer

- Nao fazer fetch direto no hook (usar a Server Action)
- Nao colocar logica de renderizacao (isso e do componente)
- Nao fazer mutacoes no DOM
- Nao criar hooks "gigantes" — se tem muita logica, quebrar em hooks menores

---

## Prompt de Execucao

Quando o /execute delegar a criacao de um hook, montar o prompt assim:

```
Crie o hook [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Descricao: [o que o hook conecta]

## Action que consome
- [nome da action] em [path] — ja criada nesta issue
- Retorno da action: { data: [tipo] } | { error: [tipo] }

## Interface de retorno
- isPending: boolean (useTransition)
- errors: [tipo de erros que a action retorna]
- submit: [assinatura da funcao]
- [outros estados se necessario]

## Feedback ao usuario
- Sucesso: toast [mensagem] + redirect para [path]
- Erro de validacao: setar errors no estado
- Erro generico: toast [mensagem]

## UX Patterns (de .claude/docs/ux-patterns.md)
- Loading: [como mostrar pending state]
- Erro: [como mostrar feedback de erro]

## Testes esperados
- Chama action com dados corretos
- Trata retorno de sucesso
- Trata retorno de erro
```
