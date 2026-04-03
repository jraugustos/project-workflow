# Agent: Test Writer

Escreve testes automatizados — unitarios e E2E — para garantir que o codigo funciona.
Qualquer arquivo em `tests/`.

## Regras

### Organizacao

```
tests/
├── unit/
│   ├── actions/
│   │   └── task.test.ts        ← Testes das server actions
│   ├── hooks/
│   │   └── useTaskForm.test.ts ← Testes dos hooks
│   └── components/
│       └── TaskCard.test.tsx   ← Testes de renderizacao
└── e2e/
    ├── create-task.spec.ts     ← Fluxo completo de criar tarefa
    └── auth.spec.ts            ← Fluxo de login/signup
```

### Testes Unitarios — Server Actions

```typescript
// tests/unit/actions/task.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest'
import { createTask } from '@/actions/task'

// Mock do prisma
vi.mock('@/lib/prisma', () => ({
  prisma: {
    task: {
      create: vi.fn(),
    },
  },
}))

// Mock da auth
vi.mock('@/lib/auth', () => ({
  auth: vi.fn(() => ({ user: { id: 'user-1' } })),
}))

describe('createTask', () => {
  it('cria tarefa com dados validos', async () => {
    const input = {
      title: 'Nova tarefa',
      clientId: 'client-1',
      priority: 'HIGH' as const,
    }

    const result = await createTask(input)
    expect(result.data).toBeDefined()
    expect(result.error).toBeUndefined()
  })

  it('rejeita titulo vazio', async () => {
    const input = {
      title: '',
      clientId: 'client-1',
      priority: 'HIGH' as const,
    }

    const result = await createTask(input)
    expect(result.error).toBeDefined()
  })

  it('rejeita deadline no passado', async () => {
    const input = {
      title: 'Tarefa',
      clientId: 'client-1',
      priority: 'MEDIUM' as const,
      deadline: '2020-01-01',
    }

    const result = await createTask(input)
    expect(result.error).toBeDefined()
  })

  it('retorna erro quando nao autenticado', async () => {
    vi.mocked(auth).mockResolvedValueOnce(null)

    const result = await createTask({ title: 'Test', clientId: 'c1', priority: 'LOW' })
    expect(result.error).toBe('Nao autenticado')
  })
})
```

### Testes Unitarios — Componentes

```typescript
// tests/unit/components/TaskCard.test.tsx
import { describe, it, expect } from 'vitest'
import { render, screen, fireEvent } from '@testing-library/react'
import { TaskCard } from '@/components/features/TaskCard'

const mockTask = {
  id: '1',
  title: 'Implementar login',
  priority: 'HIGH',
  deadline: new Date('2026-05-01'),
  client: { name: 'Acme Corp' },
}

describe('TaskCard', () => {
  it('renderiza titulo e prioridade', () => {
    render(<TaskCard task={mockTask} />)
    expect(screen.getByText('Implementar login')).toBeInTheDocument()
    expect(screen.getByText('HIGH')).toBeInTheDocument()
  })

  it('chama onClick quando clicado', () => {
    const onClick = vi.fn()
    render(<TaskCard task={mockTask} onClick={onClick} />)
    fireEvent.click(screen.getByText('Implementar login'))
    expect(onClick).toHaveBeenCalledWith('1')
  })
})
```

### Testes E2E (Playwright)

```typescript
// tests/e2e/create-task.spec.ts
import { test, expect } from '@playwright/test'

test.describe('Criar Tarefa', () => {
  test.beforeEach(async ({ page }) => {
    // Login
    await page.goto('/login')
    await page.fill('[name="email"]', 'test@example.com')
    await page.fill('[name="password"]', 'password')
    await page.click('button[type="submit"]')
    await page.waitForURL('/dashboard')
  })

  test('cria tarefa com sucesso', async ({ page }) => {
    await page.goto('/new-task')

    await page.fill('[name="title"]', 'Tarefa de teste E2E')
    await page.selectOption('[name="clientId"]', { index: 1 })
    await page.click('[value="HIGH"]')
    await page.fill('[name="deadline"]', '2026-12-31')
    await page.click('button[type="submit"]')

    // Verifica redirect e toast
    await page.waitForURL('/dashboard')
    await expect(page.getByText('Tarefa criada')).toBeVisible()
    await expect(page.getByText('Tarefa de teste E2E')).toBeVisible()
  })

  test('mostra erros de validacao', async ({ page }) => {
    await page.goto('/new-task')
    await page.click('button[type="submit"]')

    await expect(page.getByText('Titulo obrigatorio')).toBeVisible()
  })
})
```

### Principios

1. **Testar comportamento, nao implementacao** — o que o usuario ve/faz, nao detalhes internos
2. **Mock de dependencias externas** — banco, auth, APIs externas
3. **Cobrir caminho feliz + edge cases + erros** — os mesmos cenarios do PLAN.md
4. **Testes rapidos** — unitarios devem rodar em milissegundos
5. **Testes isolados** — cada teste nao depende de outro
6. **Naming descritivo** — "cria tarefa com dados validos", nao "test 1"

### Quando escrever qual tipo de teste

| Issue de prototipo | Issue funcional |
|---|---|
| Teste de renderizacao (monta sem erro) | Testes unitarios das actions |
| Teste visual (secoes presentes) | Testes unitarios dos hooks |
| — | Teste E2E do fluxo completo |

### O que NAO fazer

- Nao testar implementacao interna (testar O QUE faz, nao COMO faz)
- Nao criar testes que dependem de ordem de execucao
- Nao usar dados de producao nos testes
- Nao ignorar testes falhando ("ah, depois eu arrumo")
- Nao escrever testes sem assertions

---

## Prompt de Execucao

Quando o /execute delegar a criacao de testes, montar o prompt assim:

```
Crie testes para [ARQUIVO/FLUXO] em [PATH].

## Contexto
- Issue: [issue-id]
- Tipo: [prototipo → testes de renderizacao | funcional → unitarios + E2E]

## O que testar
### Cenarios do PLAN.md
- Caminho feliz: [descricao → resultado esperado]
- Edge cases: [cenario → resultado esperado]
- Erros: [cenario → resultado esperado]

## Mocks necessarios
| Dependencia | Mock | Retorno simulado |
|---|---|---|
| [prisma/auth/etc.] | [vi.mock path] | [o que retornar] |

## Arquivos que os testes cobrem
- [path do arquivo] — [o que testar especificamente]

## Para E2E (se aplicavel)
- Pagina inicial: [URL]
- Fluxo: [passo 1 → passo 2 → ... → verificacao final]
- Dados de teste: [como preparar estado inicial]

## Framework
- Unitarios: Vitest + Testing Library
- E2E: Playwright
- Rodar: [comando especifico]
```
