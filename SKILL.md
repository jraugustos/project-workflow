---
name: project-workflow
description: >
  Sistema completo de desenvolvimento estruturado com 6 comandos: /brief (opcional) → /spec → /break → /plan → /execute → /review.
  Spec com 4 camadas (overview, paginas, componentes, comportamentos), quebra em issues de prototipo
  e issues funcionais, pesquisa interna+externa por issue (com Context7 MCP para docs de libs),
  execucao com agentes especializados por tipo de arquivo, branches isoladas por issue,
  testes em cada implementacao, e code review automatizado pos-execucao.
  Suporta Next.js + React, Node.js (Express/Fastify) e Python (FastAPI/Flask).
  Inspirado no workflow Anti-Vibe Coding (Deb Folloni / Epic CLI / Traycer).
  Acione sempre que o usuario quiser iniciar um projeto, planejar uma feature, estruturar desenvolvimento,
  ou mencionar "novo projeto", "iniciar projeto", "/brief", "/spec", "/break", "/plan", "/execute", "/review", "workflow",
  "framework de projeto", "planejar feature", "pesquisar antes de implementar", "spec", "como comecar",
  "estruturar projeto", "anti-vibe coding", "quero fazer direito", "planejamento tecnico".
  Tambem acione quando o usuario parecer prestes a "vibe codar" algo complexo sem planejamento.
  Nao use para documentacao de projeto existente (use project-knowledge-hub).
---

# Project Workflow — Sistema de Desenvolvimento Estruturado

Sistema completo em **6 comandos** que separa planejamento de execucao,
garantindo que a IA nunca engasgue no meio do caminho.

```
/brief   → (OPCIONAL) Transforma uma ideia em briefing estruturado
/spec    → Descreve TUDO que a aplicacao precisa fazer (4 camadas)
/break   → Quebra a spec em issues pequenas (prototipo + funcional)
/plan    → Pesquisa interna+externa antes de cada issue
/execute → Implementa com agentes especializados + testes
/review  → Code review automatizado antes do merge
```

### Quando usar /brief vs ir direto pro /spec

| Situacao | Comando inicial |
|---|---|
| Projeto do zero, ideia nova | `/brief` → depois `/spec` |
| Feature nova em projeto existente | `/spec` direto |
| Alteracao/melhoria em feature existente | `/spec` direto |
| Bug fix complexo | `/spec` direto (spec simplificada) |

## Por que este sistema funciona

1. **A IA nunca ve o projeto inteiro de uma vez** — cada issue e pequena e autocontida
2. **Pesquisa acontece POR ISSUE, nao so no inicio** — reutiliza codigo existente, evita duplicacao
3. **Cada issue roda em branch isolada** — voce aprova e faz merge, sem risco de quebrar o main
4. **Agentes especializados por tipo de arquivo** — cada um segue regras especificas para seu dominio
5. **Testes em cada implementacao** — nada sobe quebrado

## Os 5 Problemas que Este Sistema Resolve

1. **Over-engineering** — a IA complica o que poderia ser simples
2. **Reinventar a roda** — criar do zero em vez de usar padroes/codigo existentes
3. **Limites de conhecimento do modelo** — docs mais recentes que o training cutoff
4. **Codigo duplicado** — componentes repetidos por falta de visao do projeto
5. **IA desobediente** — muda arquivos que nao deveria (resolvido pelo /plan que lista arquivos exatos)

---

## Setup do Projeto

Antes de comecar, o projeto deve ter esta estrutura:

```
projeto/
├── .claude/
│   └── docs/                    ← Documentacao interna para a IA
│       ├── stack.md             ← Stack tecnica completa (versoes, libs, infra)
│       ├── architecture.md      ← Arquitetura e convencoes de codigo
│       ├── design-system.md     ← Tokens visuais e componentes base
│       ├── business-rules.md    ← Regras de negocio do dominio
│       ├── ux-patterns.md       ← Padroes de UX e interacao
│       └── workflow.md          ← Convencoes de git, PR, testes, deploy
├── docs/
│   ├── brief/                   ← Output do /brief (opcional, so projetos do zero)
│   │   ├── BRIEF.md             ← PRD compacto
│   │   ├── user-flows.mermaid   ← Diagramas de fluxo
│   │   ├── architecture.mermaid ← Diagrama de arquitetura
│   │   ├── wireframes.html      ← Wireframes low-fidelity
│   │   ├── data-model.mermaid   ← Modelo de dados
│   │   └── decisions.md         ← Registro de decisoes
│   ├── SPEC.md                  ← Output do /spec
│   ├── ISSUES.md                ← Output do /break
│   └── issues/                  ← Uma pasta por issue (output do /plan)
│       ├── proto-01/
│       │   └── PLAN.md
│       ├── proto-02/
│       │   └── PLAN.md
│       ├── func-01/
│       │   └── PLAN.md
│       └── ...
├── CLAUDE.md                    ← Contexto persistente do projeto
├── src/
│   ├── app/                     ← Pages e routes (Next.js App Router)
│   ├── components/              ← Componentes React
│   ├── actions/                 ← Server Actions
│   ├── hooks/                   ← Custom hooks
│   ├── lib/                     ← Utilitarios, clients
│   ├── models/                  ← Schema do banco (Prisma, Drizzle, etc.)
│   └── integrations/            ← Conexoes com servicos externos
└── tests/
    ├── unit/
    └── e2e/
```

A arquitetura segue o principio **Paginas > Comportamentos > Componentes**:
organizada para ser facil de entender e manter, mesmo depois de crescer.

### Variacao: Monorepo e Multi-Projeto

Para monorepos ou projetos com frontend e backend separados, adaptar a estrutura:

```
monorepo/
├── .claude/
│   └── docs/                    ← Docs GLOBAIS (stack.md, workflow.md)
├── docs/                        ← Specs e issues do projeto inteiro
├── apps/
│   ├── web/                     ← Frontend Next.js
│   │   ├── .claude/docs/        ← Docs especificos (design-system.md, ux-patterns.md)
│   │   └── src/
│   └── api/                     ← Backend Node.js ou Python
│       ├── .claude/docs/        ← Docs especificos (business-rules.md, architecture.md)
│       └── src/
├── packages/
│   ├── ui/                      ← Design system compartilhado
│   ├── types/                   ← Types compartilhados
│   └── utils/                   ← Utilitarios compartilhados
└── CLAUDE.md
```

**Regras para monorepo:**
1. `.claude/docs/` na raiz tem docs globais (stack.md com overview, workflow.md)
2. Cada app tem `.claude/docs/` proprio para docs especificos
3. No PLAN.md, prefixar paths com o app: `apps/web/src/components/...`
4. Agentes seguem as mesmas regras mas respeitam as fronteiras entre apps
5. `packages/` compartilhados tem suas proprias regras em `packages/[nome]/.claude/docs/`

**Regras para frontend + backend separados:**
1. Issues de prototipo tocam apenas `apps/web/`
2. Issues funcionais podem tocar ambos (ex: action no web + endpoint na api)
3. No PLAN.md de issues cross-app, separar arquivos por app
4. Testes E2E testam a integracao entre apps

---

## Comando 0: /brief — Da Ideia ao Briefing (OPCIONAL)

**Objetivo:** Transformar uma ideia vaga em um briefing estruturado que serve de input para o /spec.
**Output:** `docs/brief/` (BRIEF.md + artefatos visuais)
**Conversa:** 1 conversa dedicada
**Quando usar:** Projetos do zero, quando o usuario ainda nao tem clareza total do que quer.
**Quando NAO usar:** Features em projeto existente, alteracoes, bug fixes — ir direto pro /spec.

O /brief funciona como uma entrevista interativa em 6 fases:

1. **Captura de intent** — o que, por que, pra quem (a IA desafia ambiguidade)
2. **Escopo e funcionalidades** — core do MVP, o que fica de fora, fluxo critico
3. **Referencia visual** — de onde vem o design (Figma, Stitch, screenshots, nada)
4. **Stack e restricoes** — decisoes tecnicas, prazos, integrações
5. **Gerar artefatos visuais** — diagramas Mermaid + wireframes HTML + modelo de dados
6. **Validar e fechar** — revisar tudo com usuario, iterar, gerar pasta `docs/brief/`

O output nao e so um documento — e uma pasta com artefatos que materializam a ideia:

```
docs/brief/
├── BRIEF.md              ← PRD compacto
├── user-flows.mermaid    ← Diagramas de fluxo do usuario
├── architecture.mermaid  ← Componentes do sistema e conexoes
├── wireframes.html       ← Wireframes low-fidelity das telas principais
├── data-model.mermaid    ← Entidades e relacoes do banco
└── decisions.md          ← Registro de decisoes com justificativas
```

Esses artefatos alimentam o /spec:
- BRIEF.md → Overview (problema, solucao, publico)
- wireframes.html → Componentes (extrair elementos de cada tela)
- user-flows.mermaid → Comportamentos (cada passo do fluxo)
- data-model.mermaid → Schema referenciado nos comportamentos
- decisions.md → Contexto para nao revisitar decisoes ja tomadas

Consulte `references/brief-guide.md` para o guia completo com perguntas,
template do BRIEF.md e criterios de qualidade.

### Fluxo do /brief

1. Usuario diz o que quer construir (pode ser vago)
2. IA conduz entrevista interativa por 4 fases (desafia ambiguidade, forca decisoes)
3. Gerar artefatos visuais: diagramas Mermaid + wireframes HTML + modelo de dados
4. Apresentar tudo ao usuario: "Os fluxos e wireframes fazem sentido?"
5. Iterar ate aprovacao
6. Salvar tudo em `docs/brief/`
7. **Limpar contexto** — `/clear` ou nova conversa
8. Iniciar /spec usando `docs/brief/` como input

---

## Comando 1: /spec — Escreva a Spec

**Objetivo:** Descrever TUDO que a aplicacao precisa fazer ANTES de escrever qualquer codigo.
**Input:** `docs/brief/` (se veio do /brief) ou descricao direta do usuario
**Output:** `docs/SPEC.md`
**Conversa:** 1 conversa dedicada

A Spec tem **4 camadas hierarquicas**:

```
Overview (visao geral do projeto)
  └── Paginas (todas as telas)
        └── Componentes (elementos visiveis em cada pagina)
              └── Comportamentos (o que o usuario pode fazer com cada componente)
```

Consulte `references/spec-guide.md` para o guia completo de como escrever cada camada,
com exemplos e criterios de qualidade.

### Fluxo do /spec

1. Se existe `docs/brief/` → ler BRIEF.md + artefatos como input (pula para passo 3)
2. Se nao tem brief → usuario descreve o projeto (pode ser vago, a IA vai refinar)
3. Fazer perguntas de clarificacao para preencher lacunas
4. Gerar a spec completa em `docs/SPEC.md` com as 4 camadas
5. Revisar com o usuario: "A spec cobre tudo?"
6. **Limpar contexto** — `/clear` ou nova conversa

---

## Comando 2: /break — Quebre em Issues

**Objetivo:** Transformar a spec em issues pequenas e bem definidas, separando visual de funcional.
**Input:** `docs/SPEC.md`
**Output:** `docs/ISSUES.md`
**Conversa:** 1 conversa dedicada

Cada issue representa uma **unidade de entrega coesa** (uma pagina, um fluxo, uma integracao)
e pode conter **subtasks**:

### Subtask Visual (V)
A parte de interface — layout, componentes, responsividade, dados mockados.
Sempre executada primeiro quando presente.

### Subtask Funcional (F)
A logica — server actions, banco, validacao, integracoes.
Depende da subtask V da mesma issue (quando existe).

Nem toda issue precisa das duas: landing pages podem ter so V,
cron jobs podem ter so F. A maioria das features core tem V + F.

Consulte `references/break-guide.md` para o guia completo de como fazer a quebra,
com regras de granularidade e exemplos.

### Fluxo do /break

1. Ler `docs/SPEC.md`
2. Identificar unidades de entrega (paginas, fluxos, integracoes)
3. Para cada issue, definir subtasks: V (visual), F (funcional), ou ambas
4. Definir ordem de implementacao e dependencias (entre issues e entre subtasks)
5. Salvar em `docs/ISSUES.md`
6. Revisar com o usuario
7. **Limpar contexto** — `/clear` ou nova conversa

---

## Comando 3: /plan — Pesquise e Planeje Cada Issue

**Objetivo:** Antes de implementar qualquer issue, pesquisar o que ja existe e planejar exatamente o que fazer.
**Input:** `docs/ISSUES.md` (issue especifica) + codebase atual
**Output:** `docs/issues/[issue-id]/PLAN.md`
**Conversa:** 1 conversa por issue (ou grupo pequeno de issues relacionadas)

O /plan faz a IA pesquisar em **dois lugares**:

### 1. Pesquisa Interna (dentro do projeto)
- Componentes que ja existem e podem ser reutilizados
- Patterns ja estabelecidos no codigo
- Utilitarios, hooks, types que ja foram criados
- Estilos e design tokens ja definidos

### 2. Pesquisa Externa (fora do projeto)
- **Context7 MCP (se disponivel)** — usar o MCP do Context7 para buscar documentacao atualizada
  de qualquer lib ou framework. Isso garante docs precisos e na versao correta do projeto.
  Quando disponivel, preferir Context7 antes de WebSearch para documentacao de libs.
- Documentacoes oficiais das libs usadas (via Context7 ou WebSearch)
- Repos de referencia no GitHub
- Stack Overflow para padroes comprovados
- Docs mais recentes (via WebSearch quando Context7 nao tem)

Consulte `references/plan-guide.md` para o guia completo.

### Output do /plan: Issue Enriquecida

O PLAN.md de cada issue deve conter:
- Descricao detalhada da tarefa
- Cenarios: caminho feliz, edge cases, erros
- Tabelas de banco necessarias (se aplicavel)
- Dependencias externas
- **Arquivos exatos** que precisam ser criados ou modificados (e o que mudar em cada um)

Esse ultimo item e o mais poderoso: quando a issue diz exatamente quais arquivos tocar,
a IA nao vai mexer em mais nada. Isso resolve o problema da "IA desobediente".

### Fluxo do /plan

1. Ler a issue do `docs/ISSUES.md`
2. Escanear o codebase para pesquisa interna
3. Pesquisar externamente se necessario
4. Gerar `docs/issues/[issue-id]/PLAN.md`
5. Revisar com o usuario: "O plano faz sentido?"
6. **Limpar contexto** — `/clear` ou nova conversa

---

## Comando 4: /execute — Implemente com Agentes Especializados

**Objetivo:** Implementar a issue planejada usando agentes especializados por tipo de arquivo.
**Input:** `docs/issues/[issue-id]/PLAN.md` + `CLAUDE.md` + `.claude/docs/`
**Output:** Codigo implementado + testes + branch pronta para merge
**Conversa:** 1 conversa por issue

### Fluxo do /execute

```
1. Criar branch: git checkout -b issue/[issue-id]
2. Ler PLAN.md da issue
3. Ler docs internos (.claude/docs/) para contexto de arquitetura
4. Para cada arquivo no plano:
   → Identificar tipo (component, action, hook, model, route, integration, test)
   → Usar o agente especializado correspondente
   → Implementar seguindo as regras do agente
5. Rodar testes (unit + e2e se aplicavel)
6. Se testes passam → commit
7. Se testes falham → corrigir → re-testar → commit
8. Apresentar diff para o usuario aprovar
9. Usuario aprova → merge na main
```

### Agentes Especializados

Cada tipo de arquivo tem seu proprio agente com regras especificas e **prompt de execucao otimizado**.
Os agentes estao em `agents/` e devem ser lidos antes de implementar cada tipo.

Cada agente inclui um template de prompt que o /execute preenche com dados do PLAN.md + .claude/docs/.
Isso garante que a IA receba instrucoes precisas (quais campos validar, quais patterns seguir,
quais testes cobrir) em vez de interpretar livremente a partir do PLAN.md:

#### Frontend (Next.js + React)

| Agente | O que faz | Quando usar |
|---|---|---|
| `component-writer` | Cria elementos visuais (botoes, forms, cards, layouts) | Qualquer arquivo em `src/components/` |
| `action-writer` | Escreve Server Actions (processar dados no servidor) | Qualquer arquivo em `src/actions/` |
| `hook-writer` | Cria hooks que conectam UI com actions | Qualquer arquivo em `src/hooks/` |
| `route-writer` | Cria API routes e endpoints Next.js | Qualquer arquivo em `src/app/api/` |

#### Backend

| Agente | O que faz | Quando usar |
|---|---|---|
| `backend-node-writer` | Controllers, services, middleware Node.js | Backend separado Express/Fastify/Hono |
| `backend-python-writer` | Endpoints, services, schemas Python | Backend FastAPI/Flask/Django |

#### Compartilhados

| Agente | O que faz | Quando usar |
|---|---|---|
| `model-writer` | Define schema do banco de dados | Qualquer arquivo em `src/models/` ou `prisma/` |
| `integration-writer` | Conecta com servicos externos (Stripe, email, etc.) | Qualquer arquivo em `src/integrations/` |
| `test-writer` | Escreve testes unitarios e E2E | Qualquer arquivo em `tests/` |

Consulte `references/execute-guide.md` para o guia completo de execucao e
`agents/[nome].md` para as regras de cada agente.

### Branches Isoladas

Cada issue e implementada em uma branch separada:

```
main
  ├── issue/proto-01-landing-page
  ├── issue/proto-02-dashboard
  ├── issue/func-01-user-auth
  └── issue/func-02-create-task
```

O usuario revisa o diff e aprova antes do merge.
Isso garante que o main nunca quebra.

### Testes Obrigatorios

Toda issue deve incluir testes:
- **Issues de prototipo:** Testes de renderizacao (componente monta sem erro)
- **Issues funcionais:** Testes unitarios das actions/hooks + E2E do fluxo completo

### Regra: Design System First

**OBRIGATORIO antes de criar qualquer componente visual:**

1. Ler `.claude/docs/design-system.md`
2. Verificar se ja existe um componente base em `src/components/ui/` que atende
3. Se existe → usar o existente (customizar via props se necessario)
4. Se nao existe → criar seguindo os tokens do design system (cores, espacamento, tipografia)
5. **Nunca** hardcodar cores, fontes ou espacamentos — sempre usar tokens/variaveis

Isso vale para TODO agente que toca em componentes visuais:
`component-writer`, `hook-writer` (quando renderiza), e subtasks visuais.

**Dica:** Instalar a skill `frontend-design` da Anthropic para complementar o
design system com principios de qualidade visual profissional. Ver secao
"Ferramentas Externas" para detalhes.

---

## Comando 5: /review — Code Review Pos-Execucao

**Objetivo:** Verificar se a implementacao atende todos os requisitos antes do merge.
**Input:** PLAN.md da issue + diff da branch
**Output:** Relatorio de review com aprovacao ou lista de correcoes
**Conversa:** Mesma conversa do /execute (antes do /clear)

O /review roda DEPOIS do /execute e ANTES do merge. Funciona como um code review
automatizado em **3 camadas**, cada uma com foco distinto:

1. **Requisitos** — a spec foi cumprida? Todos os arquivos do PLAN criados/modificados?
   Caminho feliz funciona? Edge cases tratados? Testes existem e passam?
2. **Design** — visual confere com referencia? Design system seguido? Responsivo?
   Acessivel? UX patterns corretos? (so para issues com componentes visuais)
3. **Implementacao** — codigo limpo? Arquitetura correta? Seguranca ok?
   Sem `any`, sem console.log, sem dados sensiveis expostos?

Cada camada recebe um status independente (✅ ⚠️ ❌) e o relatorio mostra um resumo.

### Fluxo

```
/execute → implementacao pronta → /review → relatorio
  Se ✅ → merge
  Se ⚠️ → usuario decide (mergear ou corrigir)
  Se ❌ → corrigir na mesma conversa → re-review → merge
```

Consulte `references/review-guide.md` para o checklist completo e formato do relatorio.

---

## Ferramentas Externas Disponiveis

O workflow pode se integrar com ferramentas externas quando disponiveis.
Nenhuma e obrigatoria — o workflow funciona sem elas — mas cada uma agrega valor.

### Context7 MCP — Documentacao de libs

Se o MCP do Context7 estiver instalado, usar para buscar documentacao atualizada
de qualquer lib ou framework. Especialmente util quando:

- A lib e mais recente que o knowledge cutoff do modelo
- A versao especifica do projeto tem APIs diferentes da que o modelo conhece
- Precisa de exemplos de codigo na versao exata

```
Ordem para pesquisa de docs de libs:
1. Context7 MCP (se disponivel) → docs oficiais na versao correta
2. WebSearch → quando Context7 nao tem a lib ou precisa de artigos/tutoriais
3. Repos GitHub → clonar em /tmp para estudar patterns
```

### Fonte de Interface — Design como Referencia

O workflow precisa saber DE ONDE vem a referencia visual do projeto.
Pode ser Figma, Google Stitch, screenshots, prototipos em HTML, mockups em outra
ferramenta, ou ate descricao textual detalhada. O importante e que exista uma
fonte de verdade visual e que a IA saiba como consulta-la.

**No /spec ou /brief:** Perguntar ao usuario: "De onde vem a referencia visual?"
As respostas possiveis sao:

| Fonte | Como usar |
|---|---|
| **Figma** (com MCP conectado) | `get_design_context`, `get_screenshot`, `search_design_system`, `get_variable_defs` |
| **Figma** (sem MCP) | Usuario cola screenshots ou links; extrair tokens manualmente |
| **Google Stitch / outra ferramenta** | Usuario exporta screenshots ou CSS; usar como referencia |
| **Screenshots / mockups** | Analisar imagens para extrair layout, cores, tipografia |
| **Descricao textual** | Definir tokens no `design-system.md` antes de qualquer prototipo |
| **Nenhuma (projeto sem design)** | Definir design system minimo antes de comecar (tokens basicos) |

**No /plan (issues de prototipo):** Consultar a fonte de interface definida:
1. Se tem MCP (Figma ou outro): buscar componentes, tokens e screenshots via MCP
2. Se tem screenshots/mockups: referenciar no PLAN.md como guia visual
3. Se so tem descricao: garantir que `design-system.md` tem tokens concretos

**No /execute (component-writer):** Implementar comparando com a referencia visual:
1. Consultar a fonte (MCP, screenshot, tokens) antes de criar qualquer componente
2. Adaptar para o projeto (nao copiar cegamente)
3. Validar visualmente ao final

**No /review:** Comparar implementacao com a referencia visual original.

#### Ferramentas de Design via MCP

Quando um MCP de design esta conectado (Figma, ou outro), as funcoes tipicas sao:

| Funcao | O que faz | Quando usar |
|---|---|---|
| Buscar componentes | Encontra componentes existentes na biblioteca de design | /plan — verificar o que ja existe |
| Extrair contexto | Gera codigo de referencia a partir do design | /execute — base para implementacao |
| Capturar screenshot | Visualiza exatamente como o componente deve ficar | /execute e /review — validacao visual |
| Extrair variaveis | Obtem tokens (cores, espacamento, tipografia) do design | /plan — popular design-system.md |

### Skill frontend-design da Anthropic — Qualidade Visual

Se o projeto tiver a skill `frontend-design` da Anthropic instalada, ela e
usada automaticamente pelo `component-writer` para elevar a qualidade visual.

**Instalacao:**
```bash
npx skills add anthropics/skills -- skill frontend-design
```

A skill complementa o design-system.md do projeto com principios de qualidade
estetica: tipografia distintiva, paleta com intencao, motion em momentos de
alto impacto, composicao espacial e texturas com profundidade. O design-system.md
do projeto sempre tem prioridade (tokens especificos), e a skill frontend-design
atua como camada de qualidade visual por cima.

Se nao houver design-system.md nem referencia visual alguma, a skill frontend-design
se torna a principal guia de qualidade — muito melhor que depender dos defaults
genericos do Tailwind.

### Boas Praticas de Frontend — Evitar Design Generico

O problema mais comum com IA gerando frontend e criar interfaces "genericas" que
parecem templates. Para evitar isso:

1. **Sempre comece pela referencia visual** — se tem MCP de design, use-o.
   Se tem screenshots, analise-os. Se nao tem nada, defina tokens ANTES de codar.
2. **Design system como fonte de verdade** — `.claude/docs/design-system.md` deve
   ter tokens concretos, nao genericos. "Azul #2563EB", nao "cor primaria".
3. **Componentes da ferramenta de design > componentes genericos** — se o design
   tem um Button especifico, buscar na biblioteca e replicar fielmente.
4. **Screenshots como validacao** — ao final de cada issue de prototipo, comparar
   visualmente com a referencia original.
5. **Nao usar Tailwind defaults** — configurar `tailwind.config` com os tokens
   do projeto antes de comecar qualquer prototipo.

---

## Resumo Visual do Sistema

```
┌─────────────────────────────────────────────────────┐
│  /brief (OPCIONAL)               [Conversa 0]       │
│  Projeto do zero? Entrevista interativa que gera:   │
│  PRD + wireframes + diagramas + modelo de dados     │
│  Output: docs/brief/ (BRIEF.md + artefatos)         │
│  → /clear                                           │
└──────────────────────┬──────────────────────────────┘
                       ↓ (ou pular direto para /spec)
┌─────────────────────────────────────────────────────┐
│  /spec                           [Conversa 1]       │
│  Descrever o projeto em 4 camadas:                  │
│  Overview → Paginas → Componentes → Comportamentos  │
│  Input: BRIEF.md (se existe) ou descricao direta    │
│  Output: docs/SPEC.md                               │
│  → /clear                                           │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  /break                          [Conversa 2]       │
│  Quebrar spec em issues com subtasks:               │
│  Subtask V: visual (layout, componentes, mockados)  │
│  Subtask F: funcional (logica, banco, APIs)         │
│  Output: docs/ISSUES.md                             │
│  → /clear                                           │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  /plan (por issue)               [Conversa 3..N]    │
│  Pesquisa interna: codigo existente reutilizavel    │
│  Pesquisa externa: Context7 MCP → WebSearch → repos │
│  Output: docs/issues/[id]/PLAN.md                   │
│  (lista arquivos exatos a criar/modificar)          │
│  → /clear                                           │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  /execute (por issue)            [Conversa N+1..]   │
│  Branch isolada: issue/[id]                         │
│  Design System First → verificar antes de criar     │
│  Agentes com prompts otimizados por tipo de arquivo │
│  Divergencia? → atualizar PLAN.md → continuar       │
│  Testes unitarios + E2E                             │
│  Output: codigo + testes                            │
└──────────────────────┬──────────────────────────────┘
                       ↓
┌─────────────────────────────────────────────────────┐
│  /review (mesma conversa) — 3 camadas               │
│  1. Requisitos: PLAN cumprido? Testes passam?       │
│  2. Design: visual confere? Tokens usados?          │
│  3. Implementacao: codigo limpo? Seguro?            │
│  ✅ Aprovado → merge → /clear → proxima issue       │
│  ❌ Correcoes → fix → re-review → merge → /clear    │
└─────────────────────────────────────────────────────┘
```

---

## Documentacao Interna do Projeto (.claude/docs/)

A IA precisa entender o projeto antes de agir. A documentacao interna e o que
garante que os agentes sigam os padroes do SEU projeto, nao padroes genericos.

**Esses documentos sao lidos ANTES de qualquer implementacao.**

```
.claude/
└── docs/
    ├── stack.md               ← Stack tecnica completa
    ├── architecture.md        ← Arquitetura e convencoes de codigo
    ├── design-system.md       ← Tokens visuais e componentes base
    ├── business-rules.md      ← Regras de negocio do dominio
    ├── ux-patterns.md         ← Padroes de UX e interacao
    └── workflow.md            ← Convencoes de git, PR, testes, deploy
```

| Documento | Conteudo | Quem le |
|---|---|---|
| `stack.md` | Linguagens, frameworks, versoes, banco de dados, servicos, infra de deploy. A IA consulta para saber o que esta disponivel ANTES de sugerir qualquer lib ou abordagem | Todos os agentes |
| `architecture.md` | Estrutura de pastas, convencoes de naming, patterns (ex: "Server Actions usam Zod + createSafeAction"), como o projeto trata erros, como faz paginacao | Todos os agentes |
| `design-system.md` | Tokens concretos: cores com hex, tipografia com font-family/sizes, espacamento, bordas, sombras. Lista de componentes base existentes em `src/components/ui/`. Config do Tailwind. Se tem ferramenta de design (Figma, Stitch, etc.), incluir link e instrucoes de acesso | component-writer, hook-writer, review |
| `business-rules.md` | Regras do dominio que a IA precisa respeitar. Ex: "Usuario free so pode ter 3 projetos", "Tarefa atrasada muda de cor automaticamente", "Email de boas-vindas dispara apos signup" | action-writer, backend-*-writer, test-writer |
| `ux-patterns.md` | Padroes de interacao do projeto: como toasts funcionam, como formularios validam, como loading states aparecem, como erros sao exibidos, padroes de navegacao, empty states | component-writer, hook-writer |
| `workflow.md` | Convencoes de git (branch naming, commit messages), processo de PR/review, como rodar testes, como fazer deploy, ambientes (dev/staging/prod) | Todos os agentes |

### Importancia do stack.md

O `stack.md` e o primeiro documento que a IA deve ler em qualquer comando.
Ele evita que a IA sugira libs que nao estao no projeto ou use patterns
incompativeis com a versao do framework.

Exemplo minimo:

```markdown
# Stack Tecnica

## Core
- **Runtime:** Node.js 20 LTS
- **Framework:** Next.js 14.2 (App Router)
- **Linguagem:** TypeScript 5.4 (strict mode)

## Frontend
- **Styling:** Tailwind CSS 3.4
- **Componentes:** shadcn/ui (Radix primitives)
- **Icones:** Lucide React
- **Forms:** React Hook Form + Zod

## Backend
- **ORM:** Prisma 5.x
- **Banco:** PostgreSQL 16 (Supabase)
- **Auth:** NextAuth.js 5 (Auth.js)
- **Email:** Resend

## Infra
- **Hosting:** Vercel
- **CI/CD:** GitHub Actions
- **Monitoramento:** Vercel Analytics + Sentry
```

### Quando criar esses docs

- **Projeto novo:** Criar na Etapa 0 (Setup), antes do /spec
- **Projeto existente:** Gerar automaticamente escaneando o codebase (package.json, estrutura de pastas, patterns no codigo)
- **Manter atualizado:** Sempre que uma decisao tecnica mudar, atualizar o doc correspondente

---

## Integracao com Outras Skills

| Quando | Skill |
|---|---|
| Documentar o projeto completo apos finalizar | `project-knowledge-hub` |
| Conselho estrategico sobre o projeto | `carreira-advisor` |
| Planejar a semana de desenvolvimento | `rotina-planner` |
| Registrar progresso no diario | `hub-diario` |

---

## Guia de Uso por Contexto

### No Claude Code (CLI)

```bash
# /brief (conversa 0 — OPCIONAL, so para projetos do zero)
claude "Tenho uma ideia: [descricao vaga]. Me ajude a transformar em briefing com diagramas e wireframes."
/clear

# /spec (conversa 1)
claude "Leia docs/brief/ (se existe) e gere a spec em docs/SPEC.md com overview, paginas, componentes e comportamentos."
# Ou sem brief: claude "Quero criar [descricao]. Gere a spec em docs/SPEC.md"
/clear

# /break (conversa 2)
claude "Leia docs/SPEC.md e quebre em issues de prototipo e funcionais em docs/ISSUES.md"
/clear

# /plan (conversa 3 — repetir por issue)
claude "Leia issue proto-01 em docs/ISSUES.md. Pesquise o codebase e docs externas (Context7). Gere o plano em docs/issues/proto-01/PLAN.md"
/clear

# /execute + /review (conversa 4 — repetir por issue)
claude "Leia docs/issues/proto-01/PLAN.md. Crie a branch e implemente."
# ... implementa, testa ...
claude "Agora faca o /review: verifique conformidade com PLAN, design system, testes e seguranca."
# ... review acontece ...
# ✅ → merge → /clear
# ❌ → corrige → re-review → merge → /clear
```

### No Cowork (Claude Desktop)

Cada comando e uma **conversa separada**. Ao iniciar:
- Diga qual comando esta executando: "Estou no /plan da issue func-03"
- Aponte para os docs: "Leia docs/ISSUES.md e docs/issues/func-03/PLAN.md"

### Para projetos do zero

Comece pelo /brief para estruturar a ideia:

```
/brief → docs/brief/ (artefatos) → /spec → /break → /plan → /execute → /review
```

### Para features em projeto existente

Va direto pro /spec (sem /brief). O sistema escala para features individuais:

```
docs/
├── SPEC.md                      ← Spec do projeto
├── ISSUES.md                    ← Issues do projeto
├── features/
│   └── payments/
│       ├── SPEC.md              ← Spec da feature
│       ├── ISSUES.md            ← Issues da feature
│       └── issues/
│           ├── proto-01/PLAN.md
│           └── func-01/PLAN.md
```
