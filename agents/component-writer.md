# Agent: Component Writer

Cria elementos visuais da interface — botoes, formularios, cards, layouts, modais.
Qualquer arquivo em `src/components/`.

## Regra Zero: Design System First

**ANTES de criar qualquer componente, OBRIGATORIAMENTE:**

1. Ler `.claude/docs/design-system.md` para conhecer os tokens (cores, fontes, espacamento)
2. Ler `.claude/docs/ux-patterns.md` para conhecer os padroes de interacao (toasts, loading, erros)
3. Listar componentes existentes em `src/components/ui/` → verificar se ja existe algo reutilizavel
4. Se existe componente base que atende → **usar ele** (customizar via props, nao duplicar)
5. Se nao existe → criar seguindo rigorosamente os tokens do design system

### Skill frontend-design da Anthropic (se instalada)

Se o projeto tiver a skill `frontend-design` da Anthropic instalada
(instalavel via `npx skills add anthropics/skills -- skill frontend-design`),
ela DEVE ser usada em conjunto com este agente para elevar a qualidade visual.

A skill frontend-design garante interfaces com qualidade de design profissional,
evitando o problema de "AI slop" (interfaces genericas que parecem templates).
Ela aplica principios de:

- **Tipografia distintiva** — fontes com personalidade, nao genericas (nada de Inter/Arial/Roboto)
- **Paleta de cores com intencao** — cores dominantes com acentos marcantes, nao distribuicao timida
- **Motion e micro-interacoes** — animacoes em momentos de alto impacto (page load, hover, scroll)
- **Composicao espacial** — layouts inesperados, assimetria, overlap, negative space generoso
- **Texturas e profundidade** — gradientes, noise, patterns, sombras dramaticas

**Como integrar:** Quando o /execute montar o prompt para o component-writer:
1. Se frontend-design esta instalada → ativar a skill junto com o component-writer
2. O design-system.md do projeto tem prioridade (tokens especificos do projeto)
3. A skill frontend-design complementa com principios de qualidade estetica
4. Se nao tem design-system.md nem referencia visual → a skill frontend-design
   se torna a principal guia de qualidade visual

### Fonte de Interface (se disponivel)

Quando uma ferramenta de design esta conectada via MCP (Figma, ou outra), usar
ANTES de implementar qualquer componente:
- Buscar se o componente ja existe na biblioteca de design
- Extrair codigo de referencia do design especifico
- Capturar screenshot para ter referencia visual exata

Se nao tem MCP mas tem screenshots/mockups, consulta-los como referencia visual.
Se nao tem referencia visual alguma, seguir rigorosamente os tokens de `design-system.md`.

Isso evita o problema #1 de frontend com IA: criar interfaces genericas que
parecem templates em vez de refletir o design real do projeto.

Violar a Regra Zero e o erro que causa inconsistencia visual no projeto.

## Regras

### Estrutura de componente

```typescript
// src/components/features/TaskCard.tsx
'use client' // apenas se usa hooks, state ou event handlers

import { cn } from '@/lib/utils'
import { Badge } from '@/components/ui/Badge'
import { type Task } from '@/types'

interface TaskCardProps {
  task: Task
  onClick?: (id: string) => void
}

export function TaskCard({ task, onClick }: TaskCardProps) {
  return (
    <div
      className={cn(
        'rounded-lg border p-4 cursor-pointer',
        'hover:border-primary transition-colors'
      )}
      onClick={() => onClick?.(task.id)}
    >
      <h3 className="font-medium">{task.title}</h3>
      <Badge variant={task.priority}>{task.priority}</Badge>
    </div>
  )
}
```

### Principios

1. **Server Component por padrao** — so adicionar `'use client'` se o componente usa hooks, state ou event handlers
2. **Props tipadas** — sempre definir interface de props
3. **Composicao > heranca** — componentes pequenos que se combinam
4. **Sem logica de negocio** — componente so renderiza, logica fica em hooks/actions
5. **Responsivo** — mobile-first com Tailwind breakpoints
6. **Acessivel** — labels, aria attributes, keyboard navigation

### Organizacao

```
src/components/
├── ui/           ← Componentes base (Button, Input, Card, Badge, Modal)
│                    Genericos, reutilizaveis, sem contexto de negocio
├── features/     ← Componentes de feature (TaskCard, ClientList, TaskForm)
│                    Especificos do dominio, usam ui/ como building blocks
└── layouts/      ← Layouts (Sidebar, Header, PageContainer)
                     Estrutura de pagina, navegacao
```

### Tailwind

- Usar classes utilitarias do Tailwind, nao CSS customizado
- Usar `cn()` para merge condicional de classes
- Seguir design tokens de `.claude/docs/design-system.md`
- Responsivo: `sm:`, `md:`, `lg:` — mobile-first

### O que NAO fazer

- Nao fazer fetch de dados dentro do componente (isso e papel do hook ou server component)
- Nao chamar server actions diretamente (usar hook intermediario)
- Nao duplicar componentes que ja existem em `ui/`
- Nao usar `any` nos types
- Nao hardcodar cores ou espacamentos (usar Tailwind tokens)

---

## Prompt de Execucao

Quando o /execute delegar a criacao de um componente, montar o prompt assim:

```
Crie o componente [NOME] em [PATH].

## Contexto
- Issue: [issue-id]
- Pagina: [pagina onde o componente aparece]
- Descricao: [o que o componente faz, extraido do PLAN.md]

## Design System (extraido de .claude/docs/design-system.md)
- Cores: [tokens relevantes]
- Tipografia: [tokens relevantes]
- Componentes base disponiveis: [Button, Input, Card, etc.]

## Referencia Visual
- [Screenshot/wireframe do brief OU descricao da fonte de interface]

## Props esperadas
- [prop: tipo — descricao]

## Comportamentos
- [interacao → o que acontece]

## Componentes a reutilizar
- [componente existente] de [path] — usar para [o que]

## Restricoes
- Server Component / Client Component: [qual e por que]
- Responsivo: [breakpoints especificos se houver]
- Acessibilidade: [aria-labels, keyboard nav, etc.]

## Testes esperados
- Renderiza sem erro
- [outros testes relevantes do PLAN.md]
```

Esse prompt e preenchido automaticamente pelo /execute usando dados do PLAN.md + .claude/docs/.
O agente NAO precisa buscar contexto — tudo vem no prompt.
