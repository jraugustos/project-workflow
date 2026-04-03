# Guia do /execute — Implementacao com Agentes Especializados

O /execute transforma o PLAN.md em codigo real, usando agentes especializados por tipo de arquivo.

## Fluxo Completo

```
1. git checkout -b issue/[issue-id]
2. Ler PLAN.md da issue
3. Ler .claude/docs/ na seguinte ordem:
   a. stack.md (entender o que esta disponivel)
   b. architecture.md (convencoes de codigo)
   c. design-system.md + ux-patterns.md (se issue visual)
   d. business-rules.md (se issue funcional)
   e. workflow.md (convencoes de git/testes)
4. Para cada arquivo no plano:
   a. Identificar tipo do arquivo
   b. Ler o agente correspondente em agents/[tipo].md
   c. Implementar seguindo as regras do agente
5. Rodar testes
6. Se passam → git add + commit
7. Se falham → corrigir → re-testar
8. Mostrar diff para aprovacao
9. git checkout main && git merge issue/[issue-id]
10. git branch -d issue/[issue-id]
```

---

## Mapeamento Arquivo → Agente

| Path do arquivo | Agente | Descricao |
|---|---|---|
| `src/components/**/*.tsx` | `component-writer` | Elementos visuais da UI |
| `src/actions/**/*.ts` | `action-writer` | Server Actions (logica de servidor) |
| `src/hooks/**/*.ts` | `hook-writer` | Hooks que conectam UI com actions |
| `src/models/**` ou `prisma/**` | `model-writer` | Schema do banco de dados |
| `src/app/api/**/*.ts` | `route-writer` | API Routes / endpoints |
| `src/integrations/**/*.ts` | `integration-writer` | Servicos externos (Stripe, email...) |
| `tests/**/*.{test,spec}.ts` | `test-writer` | Testes unitarios e E2E |

### Quando um arquivo nao se encaixa em nenhum agente

Arquivos em `src/lib/`, `src/types/`, `src/utils/` etc. nao tem agente especifico.
Nesses casos, seguir as convencoes de `.claude/docs/architecture.md`.

---

## Ordem de Implementacao dentro de uma Issue

Seguir de baixo para cima (camada de dados → logica → UI):

```
1. model-writer    → Schema/migrations (se a issue precisa)
2. action-writer   → Server Actions (logica de negocio)
3. route-writer    → API Routes (se necessario)
4. integration-writer → Integracoes externas (se necessario)
5. hook-writer     → Hooks que conectam action com UI
6. component-writer → Componentes visuais (conectar com hooks)
7. test-writer     → Testes unitarios + E2E
```

Essa ordem garante que cada camada ja tem suas dependencias prontas.

---

## Prompts Otimizados por Agente

Cada agente tem um template de "Prompt de Execucao" no final do seu arquivo.
O /execute DEVE montar esse prompt antes de implementar cada arquivo:

### Como funciona

```
1. Ler PLAN.md da issue
2. Para cada arquivo no plano:
   a. Identificar o agente (pelo path)
   b. Ler agents/[agente].md (regras + prompt template)
   c. Ler .claude/docs/ relevantes (design-system, business-rules, etc.)
   d. PREENCHER o prompt template com dados do PLAN.md e dos docs
   e. Executar a implementacao com o prompt montado
```

### Por que isso importa

Sem prompt otimizado, a IA recebe "crie src/actions/task.ts" e interpreta livremente.
Com prompt otimizado, a IA recebe instrucoes precisas:
- Quais campos validar e com que regras
- Quais patterns do projeto seguir
- Quais cenarios de teste cobrir
- Quais componentes reutilizar

Isso e o equivalente a dar um briefing detalhado para um dev junior em vez de
dizer "faz la o CRUD de tarefas".

---

## Divergencia do PLAN — Specs Atualizaveis

Se durante a implementacao a IA descobrir que o PLAN.md estava incompleto,
errado, ou impossivel de seguir, NAO ignorar e seguir em frente.

### Processo de atualizacao

```
1. PARAR a implementacao
2. Descrever a divergencia para o usuario:
   "O PLAN dizia [X] mas na pratica [Y] porque [motivo]"
3. Propor alternativa
4. Se usuario aprova → atualizar PLAN.md com a mudanca:
   - Adicionar secao "## Divergencias" no final do PLAN.md
   - Registrar: o que mudou, por que, quando
5. Continuar implementacao com o plano atualizado
```

### Template da secao de divergencias

```markdown
## Divergencias (atualizacoes durante /execute)

### DIV-01 — [Titulo]
**Data:** YYYY-MM-DD
**PLAN original:** [o que estava planejado]
**Realidade encontrada:** [o que se descobriu durante implementacao]
**Mudanca feita:** [o que foi alterado]
**Aprovado pelo usuario:** sim
```

### Quando acionar

- Schema do banco nao suporta o que foi planejado
- Lib tem limitacao nao prevista
- Componente base que deveria existir nao existe
- Conflito entre issues que nao foi previsto no /break
- Performance inviavel com a abordagem planejada

### Quando NAO acionar (resolver sem atualizar PLAN)

- Ajustes de nomes de variaveis
- Pequenas diferencas de implementacao que nao mudam o resultado
- Bugs que podem ser corrigidos sem alterar o escopo

---

## Branches

### Nomenclatura

```
issue/proto-01-landing-page
issue/proto-02-dashboard
issue/func-01-auth
issue/func-03-create-task
```

### Fluxo de branch

```bash
# Antes de comecar
git checkout main
git pull
git checkout -b issue/[issue-id]

# Implementar...

# Ao terminar
git add [arquivos especificos]
git commit -m "feat(issue-id): descricao curta"

# Apos aprovacao do usuario
git checkout main
git merge issue/[issue-id]
git branch -d issue/[issue-id]
```

### Conflitos

Se houver conflito no merge:
1. Mostrar os conflitos para o usuario
2. Resolver juntos
3. Testar apos resolucao
4. Commitar resolucao

---

## Commit Messages

Seguir conventional commits:

```
feat(proto-01): implement landing page layout
feat(func-03): add create task server action and form
fix(func-03): handle edge case for past deadline validation
test(func-03): add unit tests for createTask action
```

---

## Testes Obrigatorios

### Issues de prototipo

```typescript
// Teste minimo: componente renderiza sem erro
import { render } from '@testing-library/react'
import { LandingPage } from './LandingPage'

describe('LandingPage', () => {
  it('renders without crashing', () => {
    render(<LandingPage />)
  })

  it('displays all sections', () => {
    const { getByTestId } = render(<LandingPage />)
    expect(getByTestId('hero')).toBeInTheDocument()
    expect(getByTestId('features')).toBeInTheDocument()
    expect(getByTestId('pricing')).toBeInTheDocument()
  })
})
```

### Issues funcionais

```typescript
// Testes unitarios da action
describe('createTask', () => {
  it('creates a task with valid data', async () => { ... })
  it('rejects empty title', async () => { ... })
  it('rejects past deadline', async () => { ... })
  it('handles database error gracefully', async () => { ... })
})

// Teste E2E do fluxo
describe('Create Task Flow', () => {
  it('creates a task end-to-end', async () => {
    // Navigate to /new-task
    // Fill form
    // Submit
    // Verify redirect to dashboard
    // Verify task appears in list
  })
})
```

### Rodar testes

```bash
# Unitarios
npm run test -- --filter=[arquivo-de-teste]

# E2E
npm run test:e2e -- --filter=[arquivo-de-teste]

# Tudo
npm run test
```

---

## Checklist de Qualidade do /execute

Antes de pedir aprovacao do usuario:

- [ ] Branch criada a partir do main atualizado
- [ ] Todos os arquivos do PLAN.md foram implementados
- [ ] Nenhum arquivo fora do PLAN.md foi modificado
- [ ] Codigo segue as convencoes de .claude/docs/architecture.md
- [ ] Componentes seguem .claude/docs/design-system.md
- [ ] Testes unitarios passam
- [ ] Testes E2E passam (se aplicavel)
- [ ] Nenhum console.log ou codigo de debug
- [ ] Commit message segue conventional commits
- [ ] Diff esta limpo e revisavel
