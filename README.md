<div align="center">

# 🏗️ Project Workflow

### Sistema de Desenvolvimento Estruturado com IA

Um framework completo em **6 comandos** que separa planejamento de execução,
garantindo que a IA nunca perca o contexto no meio do caminho.

[![Made for Claude](https://img.shields.io/badge/Made%20for-Claude-blueviolet?style=for-the-badge&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0IiBmaWxsPSJ3aGl0ZSI+PHBhdGggZD0iTTEyIDJDNi40OCAyIDIgNi40OCAyIDEyczQuNDggMTAgMTAgMTAgMTAtNC40OCAxMC0xMFMxNy41MiAyIDEyIDJ6bTAgMThjLTQuNDIgMC04LTMuNTgtOC04czMuNTgtOCA4LTggOCAzLjU4IDggOC0zLjU4IDgtOCA4eiIvPjwvc3ZnPg==)](https://claude.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=for-the-badge)](http://makeapullrequest.com)

</div>

---

## 📖 Sobre o Projeto

**Project Workflow** é um skill/framework projetado para ser utilizado com assistentes de IA (como Claude) durante o desenvolvimento de software. Ele implementa uma metodologia estruturada que resolve os 5 maiores problemas de usar IA para programar:

| # | Problema | Como o Workflow Resolve |
|---|---|---|
| 1 | **Over-engineering** — a IA complica o que poderia ser simples | Spec detalhada limita o escopo antes de codar |
| 2 | **Reinventar a roda** — criar do zero em vez de reutilizar | Pesquisa interna obrigatória antes de cada issue |
| 3 | **Limites de conhecimento** — docs mais recentes que o training cutoff | Pesquisa externa com Context7 MCP + WebSearch |
| 4 | **Código duplicado** — componentes repetidos sem visão do projeto | Cada issue analisa o codebase existente |
| 5 | **IA desobediente** — muda arquivos que não deveria | PLAN.md lista arquivos exatos a criar/modificar |

### Inspiração

Inspirado no workflow **Anti-Vibe Coding** (Deb Folloni / Epic CLI / Traycer), que propõe separar radicalmente o planejamento da execução ao trabalhar com IA.

---

## 🎯 Filosofia

> *"A IA nunca vê o projeto inteiro de uma vez — cada issue é pequena e autocontida."*

O sistema funciona como uma **linha de produção** onde cada etapa tem um objetivo claro:

1. **Pesquisa acontece POR ISSUE**, não só no início
2. **Cada issue roda em branch isolada** — o `main` nunca quebra
3. **Agentes especializados por tipo de arquivo** — cada um segue regras específicas
4. **Testes em cada implementação** — nada sobe quebrado
5. **Code review automatizado** — antes de qualquer merge

---

## 🔄 Os 6 Comandos

```
/brief   → (OPCIONAL) Transforma uma ideia em briefing estruturado
/spec    → Descreve TUDO que a aplicação precisa fazer (4 camadas)
/break   → Quebra a spec em issues pequenas (visual + funcional)
/plan    → Pesquisa interna + externa antes de cada issue
/execute → Implementa com agentes especializados + testes
/review  → Code review automatizado antes do merge
```

### Fluxo Visual

```
┌──────────────────────────────────────────────────┐
│  /brief (OPCIONAL)                [Conversa 0]   │
│  Projeto do zero? Entrevista interativa que gera:│
│  PRD + wireframes + diagramas + modelo de dados  │
│  Output: docs/brief/                             │
└────────────────────┬─────────────────────────────┘
                     ↓ (ou pular direto para /spec)
┌──────────────────────────────────────────────────┐
│  /spec                            [Conversa 1]   │
│  4 camadas: Overview → Páginas →                 │
│  Componentes → Comportamentos                    │
│  Output: docs/SPEC.md                            │
└────────────────────┬─────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│  /break                           [Conversa 2]   │
│  Issues com subtasks:                            │
│  V (visual) + F (funcional)                      │
│  Output: docs/ISSUES.md                          │
└────────────────────┬─────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│  /plan (por issue)                [Conversa 3+]  │
│  Pesquisa interna: código reutilizável           │
│  Pesquisa externa: Context7 → WebSearch → repos  │
│  Output: docs/issues/[id]/PLAN.md                │
└────────────────────┬─────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│  /execute (por issue)             [Conversa N+]  │
│  Branch isolada + agentes especializados         │
│  Design System First + testes                    │
│  Output: código + testes implementados           │
└────────────────────┬─────────────────────────────┘
                     ↓
┌──────────────────────────────────────────────────┐
│  /review                                         │
│  3 camadas: Requisitos → Design → Implementação  │
│  ✅ → merge   ❌ → corrige → re-review → merge   │
└──────────────────────────────────────────────────┘
```

---

## 🤖 Agentes Especializados

O sistema usa **9 agentes especializados**, cada um com regras próprias otimizadas para seu domínio:

### Frontend (Next.js + React)

| Agente | Responsabilidade |
|---|---|
| `component-writer` | Cria elementos visuais (botões, forms, cards, layouts) |
| `action-writer` | Escreve Server Actions (processar dados no servidor) |
| `hook-writer` | Cria hooks que conectam UI com actions |
| `route-writer` | Cria API routes e endpoints Next.js |

### Backend

| Agente | Responsabilidade |
|---|---|
| `backend-node-writer` | Controllers, services, middleware (Express/Fastify/Hono) |
| `backend-python-writer` | Endpoints, services, schemas (FastAPI/Flask/Django) |

### Compartilhados

| Agente | Responsabilidade |
|---|---|
| `model-writer` | Define schema do banco de dados |
| `integration-writer` | Conecta com serviços externos (Stripe, email, etc.) |
| `test-writer` | Escreve testes unitários e E2E |

---

## 📁 Estrutura do Projeto

```
project-workflow/
├── SKILL.md                 ← Skill principal (instruções completas do workflow)
├── README.md                ← Este arquivo
│
├── agents/                  ← Agentes especializados por tipo de arquivo
│   ├── component-writer.md
│   ├── action-writer.md
│   ├── hook-writer.md
│   ├── route-writer.md
│   ├── backend-node-writer.md
│   ├── backend-python-writer.md
│   ├── model-writer.md
│   ├── integration-writer.md
│   └── test-writer.md
│
└── references/              ← Guias detalhados de cada comando
    ├── brief-guide.md
    ├── spec-guide.md
    ├── break-guide.md
    ├── plan-guide.md
    ├── execute-guide.md
    ├── review-guide.md
    └── project-docs-templates.md
```

---

## 🚀 Como Usar

### Pré-requisitos

- **Claude Code (CLI)** ou **Claude Desktop** com capacidade de leitura de arquivos
- Projeto configurado com a estrutura de docs interna (`.claude/docs/`)

### Instalação

1. Clone este repositório como skill do seu projeto:

```bash
# Copiar para a pasta de skills do seu assistente
git clone https://github.com/jraugustos/project-workflow.git
```

2. Configure a documentação interna do seu projeto:

```
seu-projeto/
├── .claude/
│   └── docs/
│       ├── stack.md             ← Stack técnica completa
│       ├── architecture.md      ← Arquitetura e convenções
│       ├── design-system.md     ← Tokens visuais
│       ├── business-rules.md    ← Regras de negócio
│       ├── ux-patterns.md       ← Padrões de UX
│       └── workflow.md          ← Convenções de git/PR/deploy
```

### Uso no Claude Code (CLI)

```bash
# /brief (opcional — projetos do zero)
claude "Tenho uma ideia: [descrição]. Me ajude a transformar em briefing."
/clear

# /spec
claude "Gere a spec em docs/SPEC.md com overview, páginas, componentes e comportamentos."
/clear

# /break
claude "Leia docs/SPEC.md e quebre em issues em docs/ISSUES.md"
/clear

# /plan (repetir por issue)
claude "Leia issue proto-01. Pesquise o codebase e gere docs/issues/proto-01/PLAN.md"
/clear

# /execute + /review (repetir por issue)
claude "Leia docs/issues/proto-01/PLAN.md. Crie a branch e implemente."
claude "Agora faça o /review."
# ✅ → merge → /clear
```

### Quando usar cada comando inicial

| Situação | Comando inicial |
|---|---|
| Projeto do zero, ideia nova | `/brief` → depois `/spec` |
| Feature nova em projeto existente | `/spec` direto |
| Alteração/melhoria em feature existente | `/spec` direto |
| Bug fix complexo | `/spec` direto (spec simplificada) |

---

## 🔧 Stacks Suportadas

O workflow suporta múltiplas stacks de desenvolvimento:

- **Frontend:** Next.js + React (App Router)
- **Backend Node.js:** Express, Fastify, Hono
- **Backend Python:** FastAPI, Flask, Django
- **Banco de dados:** PostgreSQL, MySQL, MongoDB (via Prisma, Drizzle, etc.)
- **Monorepos:** Suporte nativo com docs por app

---

## 🛠️ Ferramentas Externas (Opcionais)

O workflow se integra com ferramentas externas quando disponíveis:

| Ferramenta | Finalidade |
|---|---|
| **Context7 MCP** | Documentação atualizada de libs na versão correta |
| **Figma MCP** | Extrair componentes, tokens e screenshots do design |
| **Skill frontend-design** | Elevar qualidade visual dos componentes |

Nenhuma é obrigatória — o workflow funciona sem elas.

---

## 📚 Documentação dos Guias

Cada comando possui um guia de referência detalhado em `references/`:

| Guia | Conteúdo |
|---|---|
| [brief-guide.md](references/brief-guide.md) | Perguntas da entrevista, templates, critérios de qualidade |
| [spec-guide.md](references/spec-guide.md) | Como escrever cada camada da spec |
| [break-guide.md](references/break-guide.md) | Regras de granularidade, exemplos de issues |
| [plan-guide.md](references/plan-guide.md) | Pesquisa interna/externa, formato do PLAN.md |
| [execute-guide.md](references/execute-guide.md) | Fluxo de execução, branches, testes |
| [review-guide.md](references/review-guide.md) | Checklist das 3 camadas de review |
| [project-docs-templates.md](references/project-docs-templates.md) | Templates para docs internos do projeto |

---

## 🤝 Contribuindo

Contribuições são bem-vindas! Se você tem sugestões para melhorar o workflow:

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/minha-melhoria`)
3. Commit suas mudanças (`git commit -m 'feat: adiciona melhoria X'`)
4. Push para a branch (`git push origin feature/minha-melhoria`)
5. Abra um Pull Request

---

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.

---

<div align="center">

**Feito com ❤️ para desenvolvedores que querem usar IA de forma estruturada.**

*Pare de "vibe codar". Comece a planejar.*

</div>
