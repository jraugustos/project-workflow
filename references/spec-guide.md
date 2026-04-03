# Guia do /spec — Como Escrever a Spec em 4 Camadas

A Spec descreve TUDO que a aplicacao precisa fazer antes de uma unica linha de codigo.
Ela da clareza sobre o que esta sendo construido — tanto para voce quanto para a IA.

**Input:** Se existe `docs/brief/` (gerado pelo /brief), usar como base:
- `BRIEF.md` → Overview vem do Problema + Solucao + Publico
- `wireframes.html` → Componentes extraidos de cada tela
- `user-flows.mermaid` → Comportamentos derivados dos fluxos
- `data-model.mermaid` → Schema referenciado nos comportamentos
- `decisions.md` → Decisoes ja tomadas (nao revisitar)
Se nao tem brief, partir da descricao direta do usuario.

## As 4 Camadas

### Camada 1: Overview

Explicacao geral do projeto em linguagem simples.

```markdown
## Overview

**Nome:** TaskFlow
**Descricao:** SaaS de gestao de tarefas para freelancers que trabalham com multiplos clientes.
**Problema:** Freelancers usam planilhas, notas e apps diferentes para cada cliente, perdendo
contexto e prazos.
**Solucao:** Um unico lugar para organizar tarefas por cliente, com deadlines, prioridades
e visao de carga de trabalho.
**Publico:** Freelancers de tecnologia, design e marketing que gerenciam 3+ clientes simultaneos.
**Stack:** Next.js 14 (App Router), Tailwind CSS, Prisma, PostgreSQL, NextAuth.js, Vercel.
```

### Camada 2: Paginas

Todas as telas da aplicacao, descritas pela perspectiva do usuario.

```markdown
## Paginas

### P01 — Landing Page
Pagina publica que apresenta o produto e converte visitantes em usuarios.
Secoes: Hero, Features, Pricing, CTA, Footer.

### P02 — Login / Signup
Tela de autenticacao com email/senha e OAuth (Google, GitHub).
Redireciona para Dashboard apos login.

### P03 — Dashboard
Visao geral de todas as tarefas do usuario, agrupadas por cliente.
Mostra: tarefas atrasadas, tarefas de hoje, tarefas da semana.

### P04 — Pagina do Cliente
Detalhes de um cliente especifico com suas tarefas, notas e historico.

### P05 — Nova Tarefa
Form para criar tarefa com: titulo, descricao, cliente, prioridade, deadline.

### P06 — Settings
Configuracoes do usuario: perfil, notificacoes, integrações, plano.
```

Cada pagina deve ter:
- **ID unico** (P01, P02...) para referencia nas issues
- **Descricao** do que o usuario ve e faz nessa pagina
- **Secoes principais** se for uma pagina complexa

### Camada 3: Componentes

Todos os elementos visiveis em cada pagina.

```markdown
## Componentes

### P03 — Dashboard

#### C01 — TaskCard
Card que mostra uma tarefa com: titulo, cliente, prioridade (badge colorido), deadline.
Clicavel — abre o detalhe da tarefa.

#### C02 — ClientFilter
Dropdown com filtro por cliente. Opcao "Todos" selecionada por padrao.

#### C03 — PriorityFilter
Toggle de prioridade: Alta, Media, Baixa, Todas.

#### C04 — TaskStats
Cards com metricas: total de tarefas, atrasadas, concluidas hoje.

#### C05 — WeeklyTimeline
Timeline visual mostrando tarefas distribuidas na semana.

### P05 — Nova Tarefa

#### C06 — TaskForm
Formulario com campos: titulo (text), descricao (textarea), cliente (select),
prioridade (radio: alta/media/baixa), deadline (date picker).
Botao "Criar Tarefa" no final.
```

Cada componente deve ter:
- **ID unico** (C01, C02...) para referencia nos comportamentos
- **Pagina pai** (em qual pagina aparece)
- **Descricao visual** do que o usuario ve
- **Campos/elementos** se for um form ou componente interativo

### Camada 4: Comportamentos

O que o usuario pode fazer ao interagir com cada componente.
Cada comportamento e uma ACAO com consequencias.

```markdown
## Comportamentos

### P03 — Dashboard

#### B01 — Clicar em TaskCard (C01)
- **Acao:** Abre modal/pagina com detalhes da tarefa
- **Dados:** Carrega tarefa completa do banco (titulo, descricao, comentarios, historico)
- **Sucesso:** Modal abre com dados da tarefa
- **Erro:** Toast de erro se falhar ao carregar

#### B02 — Filtrar por cliente (C02)
- **Acao:** Filtra lista de tarefas pelo cliente selecionado
- **Dados:** Re-fetch das tarefas com filtro de clientId
- **Sucesso:** Lista atualiza mostrando so tarefas do cliente
- **Edge case:** Se nao tem tarefas, mostra empty state

#### B03 — Filtrar por prioridade (C03)
- **Acao:** Filtra lista de tarefas pela prioridade
- **Dados:** Filtro client-side (sem re-fetch)
- **Sucesso:** Lista atualiza

### P05 — Nova Tarefa

#### B04 — Submeter TaskForm (C06)
- **Acao:** Cria nova tarefa no banco
- **Dados:** POST com { titulo, descricao, clienteId, prioridade, deadline }
- **Validacao:** Titulo obrigatorio, deadline no futuro, cliente obrigatorio
- **Sucesso:** Redireciona para Dashboard, toast "Tarefa criada"
- **Erro de validacao:** Highlight nos campos invalidos com mensagem
- **Erro de servidor:** Toast "Erro ao criar tarefa, tente novamente"
```

Cada comportamento deve ter:
- **ID unico** (B01, B02...) para referencia nas issues
- **Componente pai** (qual componente dispara o comportamento)
- **Acao** do usuario (clicar, submeter, arrastar, etc.)
- **Dados** envolvidos (o que e lido/escrito no banco ou API)
- **Cenario de sucesso** (o que acontece quando da certo)
- **Cenario de erro** (o que acontece quando da errado)
- **Edge cases** (situacoes incomuns mas possiveis)

---

## Criterios de Qualidade da Spec

Uma boa spec:

- [ ] Tem overview claro que qualquer pessoa entende
- [ ] Lista TODAS as paginas da aplicacao
- [ ] Cada pagina tem TODOS os componentes visiveis
- [ ] Cada componente interativo tem TODOS os comportamentos mapeados
- [ ] Cada comportamento tem cenarios de sucesso, erro e edge case
- [ ] Usa IDs unicos (P01, C01, B01) para cross-referencia
- [ ] Nao tem codigo — so descreve O QUE, nao COMO
- [ ] E compreensivel para alguem nao-tecnico (exceto a secao de stack)

Uma spec incompleta vai gerar issues incompletas que vao gerar codigo incompleto.
Gaste tempo aqui — cada minuto investido na spec economiza horas de retrabalho.

---

## Registro de Decisoes na Spec

Ao final da SPEC.md, incluir uma secao de decisoes tomadas durante a especificacao.
Isso evita que decisoes sejam revisitadas ou esquecidas nas etapas seguintes.

```markdown
## Decisoes

### D01 — Dashboard como pagina inicial (nao landing page)
**Contexto:** Usuarios logados acessam direto o dashboard.
**Decisao:** Apos login, redirecionar para /dashboard, nao para /.
**Alternativas:** Landing page interna com resumo → descartada (complexidade sem valor).

### D02 — Filtro de tarefas client-side
**Contexto:** Quantidade de tarefas esperada e pequena (<200 por usuario).
**Decisao:** Filtro por prioridade sera client-side (sem re-fetch).
**Alternativas:** Server-side com query params → descartada (overengineering para o volume).

### D03 — Sem notificacoes no MVP
**Contexto:** Push/email notifications aumentam escopo significativamente.
**Decisao:** MVP sem notificacoes. Apenas toast in-app. Avaliar para v2.
```

Cada decisao deve ter: contexto (por que surgiu), decisao (o que foi escolhido),
e alternativas descartadas (o que foi considerado e por que foi rejeitado).
