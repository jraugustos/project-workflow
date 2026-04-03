# Guia do /plan — Pesquisa e Planejamento por Issue

O /plan e o que diferencia este workflow de "vibe coding".
Antes de implementar QUALQUER issue, a IA pesquisa o que ja existe e planeja exatamente o que fazer.

## Por que planejar por issue (e nao so no inicio)?

1. O codebase muda a cada issue implementada — a pesquisa interna precisa ser atualizada
2. Evita duplicacao de componentes, hooks, utilitarios
3. Garante que a IA use padroes ja estabelecidos no projeto
4. Lista arquivos exatos — a IA so toca no que foi planejado

---

## Pesquisa Interna (dentro do projeto)

Antes de criar qualquer coisa nova, verificar se ja existe:

### Regra: Design System First (obrigatorio)

Antes de planejar qualquer componente visual, o /plan DEVE:
1. Ler `.claude/docs/design-system.md`
2. Listar componentes existentes em `src/components/ui/`
3. No PLAN.md, indicar explicitamente quais componentes base serao reutilizados
4. Se um componente base nao existe e sera necessario, criar como parte da issue

Isso evita o problema #1 de inconsistencia: componentes visuais que nao seguem o design system.

### O que procurar

| Buscar | Onde | Por que |
|---|---|---|
| Componentes reutilizaveis | `src/components/` | Nao criar Button se ja tem Button |
| Hooks existentes | `src/hooks/` | Nao criar useAuth se ja tem useAuth |
| Server Actions | `src/actions/` | Reutilizar logica de servidor |
| Types/Interfaces | `src/types/` | Usar types existentes, nao recriar |
| Utilitarios | `src/lib/` | formatDate, cn, etc. ja existem? |
| Patterns no codigo | Todo o `src/` | Como o projeto faz paginacao? Como trata erros? |
| Design tokens | `.claude/docs/design-system.md` | Cores, fontes, espacamento padrao |
| Convencoes | `.claude/docs/architecture.md` | Naming, estrutura de pastas, patterns |

### Como pesquisar

```bash
# Buscar componentes existentes
find src/components -name "*.tsx" | head -20

# Buscar hooks
find src/hooks -name "*.ts"

# Buscar patterns de um tipo especifico
grep -r "createServerAction\|useFormState\|Server Action" src/ --include="*.ts" --include="*.tsx"

# Verificar se um componente similar ja existe
grep -r "TaskCard\|Card\|ListItem" src/components/ --include="*.tsx"
```

---

## Pesquisa Externa (fora do projeto)

Quando a pesquisa interna nao resolve, buscar fora.

### Ferramentas disponiveis

```
1. Context7 MCP (se disponivel) → docs oficiais da lib na versao correta
2. MCP de design (Figma, Stitch, etc. — se disponivel) → design context, componentes, tokens visuais
3. WebSearch → quando as ferramentas acima nao tem o que precisa
4. Repos GitHub → clonar em /tmp para estudar patterns complexos
```

### Quando pesquisar externamente

| Situacao | Ferramenta | O que buscar |
|---|---|---|
| Docs de lib do projeto | Context7 MCP (se disponivel) → WebSearch | Docs oficiais na versao exata |
| Design de componente visual | MCP de design (se disponivel) → design-system.md | Tokens, componentes, screenshots |
| Lib nova que o modelo nao conhece | Context7 → WebSearch | Docs atualizados |
| Integracao com API externa | Context7 → WebSearch | Docs da API + exemplos |
| Pattern complexo (drag-n-drop, real-time) | GitHub repos | Clonar e estudar |
| Bug conhecido ou limitacao | WebSearch | Issues GitHub, Stack Overflow |

### Tecnicas

1. **Context7 MCP:** Se disponivel, buscar docs oficiais de qualquer lib na versao correta
2. **MCP de design (Figma ou outro):** Se disponivel, buscar componentes existentes na biblioteca, extrair codigo de referencia, capturar screenshots para referencia visual
3. **Clonar repo de exemplo:** `git clone [repo] /tmp/ref-[nome]` → ler o pattern relevante → apagar depois
4. **WebSearch:** Quando as ferramentas acima nao tem o que precisa, ou para artigos e tutoriais
5. **Ler CHANGELOG:** Verificar se a versao que usamos suporta a feature

Nenhuma ferramenta externa e obrigatoria — o workflow funciona com WebSearch como fallback.

---

## Output: PLAN.md

Cada issue planejada gera um arquivo em `docs/issues/[issue-id]/PLAN.md`:

```markdown
# Plan — [issue-id]: [Nome da Issue]

> Issue: [referencia ao ISSUES.md]
> Data: YYYY-MM-DD

## Resumo
[1-2 frases do que sera implementado]

## Pesquisa Interna

### Design System (verificacao obrigatoria)
- Tokens: [cores, tipografia, espacamento encontrados em .claude/docs/design-system.md]
- Componentes ui/ existentes: Button, Input, Card, Badge, Modal
- Componentes a reutilizar nesta issue: Button, Input, Select

### Referencia Visual (se disponivel)
- Fonte: [Figma / Google Stitch / Screenshots / outra]
- Screenshot/mockup: [capturado via MCP ou fornecido pelo usuario]
- Codigo de referencia: [extraido da ferramenta de design, se disponivel]
- Componentes da biblioteca de design: [encontrados na ferramenta]

### Componentes reutilizaveis encontrados
- `src/components/ui/Button.tsx` — usar para botoes do form
- `src/components/ui/Input.tsx` — usar para campos de texto
- `src/components/ui/Select.tsx` — usar para dropdown de clientes
- `src/hooks/useToast.ts` — usar para feedback de sucesso/erro

### Patterns do projeto
- Server Actions usam o pattern `createSafeAction` com validacao Zod
- Forms usam `useFormState` + `useFormStatus` (pattern estabelecido em func-01)
- Toasts usam o hook `useToast` com variantes success/error/info

### Nada encontrado (precisa criar)
- `src/actions/task.ts` — nao existe, criar
- `src/hooks/useTaskForm.ts` — nao existe, criar

## Pesquisa Externa

### Documentacao consultada
- [Next.js Server Actions docs](url) — pattern de revalidacao apos mutacao
- [Prisma Client API](url) — syntax para create com relacoes

### Padroes adotados
- Usar `revalidatePath` apos criar tarefa (padrao Next.js)
- Validacao com Zod schema (padrao ja usado no projeto)

## Cenarios

### Caminho feliz
1. Usuario preenche todos os campos
2. Clica "Criar Tarefa"
3. Validacao client-side passa (Zod)
4. Server Action executa
5. Prisma cria registro
6. `revalidatePath('/dashboard')` atualiza a lista
7. Redirect para /dashboard
8. Toast "Tarefa criada"

### Edge cases
- Titulo vazio → erro de validacao Zod → campo fica vermelho
- Deadline no passado → erro de validacao customizado
- Cliente nao selecionado → erro de validacao

### Erros
- Prisma error (unique constraint, etc.) → catch → toast de erro
- Network error → catch generico → toast "Tente novamente"

## Banco de Dados

### Tabelas envolvidas
- `tasks` — CREATE (nova tarefa)
- `clients` — READ (popular select do form)

### Schema necessario (se nao existe)
```prisma
model Task {
  id          String   @id @default(cuid())
  title       String
  description String?
  priority    Priority @default(MEDIUM)
  deadline    DateTime?
  clientId    String
  client      Client   @relation(fields: [clientId], references: [id])
  userId      String
  user        User     @relation(fields: [userId], references: [id])
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}

enum Priority {
  LOW
  MEDIUM
  HIGH
}
```

## Dependencias Externas
- Nenhuma nova (Prisma e Zod ja estao no projeto)

## Arquivos — O Que Criar e Modificar

### Criar
| Arquivo | O que contem |
|---|---|
| `src/actions/task.ts` | Server Action `createTask` com validacao Zod |
| `src/hooks/useTaskForm.ts` | Hook que gerencia estado do form e chama a action |
| `tests/unit/actions/task.test.ts` | Testes unitarios da action createTask |
| `tests/e2e/create-task.spec.ts` | Teste E2E do fluxo completo |

### Modificar
| Arquivo | O que mudar |
|---|---|
| `src/components/features/TaskForm.tsx` | Conectar com useTaskForm (substituir dados mockados) |
| `prisma/schema.prisma` | Adicionar model Task e enum Priority (se nao existe) |
| `src/app/(auth)/new-task/page.tsx` | Importar TaskForm funcional (substituir prototipo) |

### NAO tocar
Tudo que nao esta listado acima. Especialmente:
- `src/components/ui/*` — componentes base ja prontos
- `src/app/(auth)/dashboard/*` — vai ser tocado em outra issue
- `src/lib/*` — utilitarios ja prontos

## Decisoes desta Issue

### D01 — [Titulo da decisao]
**Contexto:** [por que essa decisao surgiu durante o planejamento]
**Decisao:** [o que foi escolhido]
**Alternativas descartadas:** [o que foi considerado e por que nao]
```

---

## Criterios de Qualidade do /plan

- [ ] Pesquisa interna foi feita (listou componentes/hooks/patterns existentes)
- [ ] Nao esta recriando nada que ja existe
- [ ] Cenarios cobrem caminho feliz, edge cases e erros
- [ ] Schema do banco esta definido (se aplicavel)
- [ ] Lista de arquivos e EXATA (criar, modificar, NAO tocar)
- [ ] Cada arquivo tem descricao do que contem/muda
- [ ] Testes estao incluidos na lista de arquivos
