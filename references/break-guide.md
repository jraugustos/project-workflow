# Guia do /break — Como Quebrar a Spec em Issues

A quebra transforma a spec (um documento grande) em issues pequenas e autocontidas.
Issues pequenas = janela de contexto livre = IA implementa melhor.

## Estrutura: Issues com Subtasks

Cada issue representa uma **unidade de entrega coesa** (uma pagina, um fluxo, uma integracao).
Dentro de cada issue, o trabalho e dividido em **subtasks**:

- **Subtask visual (V):** a parte de interface — layout, componentes, responsividade, dados mockados
- **Subtask funcional (F):** a logica — server actions, banco, validacao, integracoes

Isso mantem o contexto junto (visual + funcional no mesmo PR), facilita o review
(a feature aparece completa no diff), e preserva a vantagem do "visual first"
(a subtask V e feita antes da F, entao o layout e validado antes da logica).

### Quando uma issue tem so 1 subtask

Nem toda issue precisa de ambas:

| Caso | Subtasks |
|---|---|
| Pagina com UI + logica | V + F (padrao) |
| Pagina puramente visual (landing, marketing) | Apenas V |
| Logica pura sem UI (cron job, webhook, migration) | Apenas F |
| Integracao externa (Stripe, email) | Apenas F (ou V + F se tem UI dedicada) |

---

## Como Criar as Issues

### Passo 1: Identificar unidades de entrega

Cada **pagina** da spec que tem comportamentos associados vira uma issue.
Fluxos que cruzam multiplas paginas (ex: auth) viram uma issue de fluxo.

### Passo 2: Definir subtasks

Para cada issue, separar o que e visual do que e funcional.

### Como nomear

```
issue-01 — Landing Page (P01)
issue-02 — Login/Signup (P02)
issue-03 — Dashboard (P03)
issue-04 — Pagina do Cliente (P04)
issue-05 — Criar Tarefa (P05)
issue-06 — Settings (P06)
issue-07 — Integracao Stripe
issue-08 — Cron de Notificacoes
```

### Template de issue com subtasks

```markdown
### issue-05 — Criar Tarefa

**Pagina:** P05
**Componentes:** C06, C07, C08
**Comportamentos:** B04

---

#### Subtask V: Interface do formulario de tarefa

**Tipo:** visual
**Descricao:** Criar a pagina e componentes do form de criacao de tarefa.
Dados mockados, sem funcionalidade.

**Secoes:**
- TaskForm (C06): campos titulo, descricao, cliente (select mockado), prioridade, deadline
- Botao "Criar Tarefa" (desabilitado sem titulo)
- Preview opcional do card de tarefa

**Responsividade:** mobile-first, breakpoints em sm, md, lg
**Design tokens:** usar variaveis de .claude/docs/design-system.md
**Skill frontend-design:** se instalada, usar para garantir qualidade visual

**Criterio de aceitacao (V):**
- [ ] Pagina renderiza sem erros
- [ ] Layout correto em mobile, tablet e desktop
- [ ] Todos os componentes visuais presentes
- [ ] Dados sao mockados (nenhuma chamada real)
- [ ] Design system tokens respeitados

---

#### Subtask F: Logica de criacao de tarefa

**Tipo:** funcional
**Depende de:** Subtask V desta issue
**Descricao:** Implementar a criacao de tarefas com validacao, persistencia
no banco e feedback visual.

**Caminho feliz:**
1. Usuario preenche o TaskForm (C06)
2. Clica em "Criar Tarefa"
3. Validacao client-side passa
4. Server Action cria registro no banco
5. Redireciona para Dashboard
6. Toast "Tarefa criada com sucesso"

**Edge cases:**
- Titulo vazio → mensagem "Titulo obrigatorio"
- Deadline no passado → mensagem "Deadline deve ser no futuro"
- Cliente nao selecionado → mensagem "Selecione um cliente"

**Cenario de erro:**
- Falha no servidor → Toast "Erro ao criar tarefa, tente novamente"
- Timeout → Toast generico de erro

**Dados:**
- Tabela: `tasks`
- Campos: id, title, description, clientId, priority, deadline, userId, createdAt
- Relacoes: Task belongsTo Client, Task belongsTo User

**Criterio de aceitacao (F):**
- [ ] Tarefa persiste no banco com todos os campos
- [ ] Validacao funciona para todos os edge cases
- [ ] Toast de sucesso aparece
- [ ] Toast de erro aparece em caso de falha
- [ ] Testes unitarios da action passam
- [ ] Teste E2E do fluxo completo passa

---

**Dependencias entre issues:**
- issue-02 (auth — precisa do userId)
- issue-04 (CRUD de clientes — precisa dos clientes pra popular o select)
```

### Template de issue so visual (sem subtask F)

```markdown
### issue-01 — Landing Page

**Pagina:** P01
**Componentes:** C01, C02, C03

---

#### Subtask V: Interface da landing page

**Tipo:** visual
**Descricao:** Criar a pagina de landing com todas as secoes visuais.
Dados mockados, sem funcionalidade.

**Secoes:**
- Hero: titulo, subtitulo, CTA button, imagem/ilustracao
- Features: grid 3 colunas com icone + titulo + descricao
- Pricing: 2-3 cards de plano com features e preco
- CTA: banner de conversao com email input
- Footer: links, social, copyright

**Responsividade:** mobile-first, breakpoints em sm, md, lg
**Design tokens:** usar variaveis de .claude/docs/design-system.md
**Skill frontend-design:** se instalada, usar para garantir qualidade visual

**Criterio de aceitacao:**
- [ ] Pagina renderiza sem erros
- [ ] Layout correto em mobile, tablet e desktop
- [ ] Todos os componentes visuais presentes
- [ ] Dados sao mockados (nenhuma chamada real)
```

### Template de issue so funcional (sem subtask V)

```markdown
### issue-08 — Cron de Notificacoes

**Comportamentos:** B40, B41

---

#### Subtask F: Job de notificacoes por email

**Tipo:** funcional
**Descricao:** Implementar cron job que envia notificacoes de tarefas
proximas do deadline.

**Logica:**
1. Query tarefas com deadline em 24h
2. Agrupar por usuario
3. Enviar email consolidado via Resend
4. Registrar log de envio

**Criterio de aceitacao:**
- [ ] Cron roda no horario configurado
- [ ] Emails sao enviados corretamente
- [ ] Testes unitarios passam
```

---

## Ordenacao das Issues

### Ordem padrao

```
1. Issues so visuais (landing, marketing)
   issue-01, issue-09...
   (podem ser paralelas entre si)

2. Fundacao
   issue-02: Auth (quase tudo depende disso)
   issue-03: Dashboard (base visual + dados)

3. Features core (cada uma com subtask V → F)
   issue-04, issue-05, issue-06...

4. Integracoes
   issue-07: Stripe, issue-10: Email, etc.

5. Polish
   issue-20: Temas, issue-21: Acessibilidade, etc.
```

### Dependencias e Paralelizacao

Existem dois niveis de dependencia:

1. **Dentro da issue:** Subtask V sempre vem antes da Subtask F (a logica depende da UI existir)
2. **Entre issues:** Uma issue pode depender de outra (ex: "Criar Tarefa" depende de "Auth")

No ISSUES.md, marcar dependencias e paralelismo:

```markdown
### issue-01 — Landing Page
**Paralelo:** sim (independente, so subtask V)

### issue-02 — Login/Signup
**Paralelo:** sim (independente)

### issue-03 — Dashboard
**Paralelo:** sim (independente da issue-01, mas subtask F depende de issue-02)

### issue-04 — CRUD de Clientes
**Paralelo com:** issue-05 (ambas dependem de issue-02, mas nao entre si)

### issue-05 — Criar Tarefa
**Paralelo com:** issue-04 (ambas dependem de issue-02, mas nao entre si)
```

**Regras de paralelizacao:**
1. Issues sem dependencias entre si podem rodar em paralelo (em branches separadas)
2. Dentro de cada issue, subtask V sempre antes de subtask F
3. Issues que modificam os mesmos arquivos NAO podem ser paralelas
4. Na duvida, marcar como sequencial — e mais seguro

---

## Output: docs/ISSUES.md

```markdown
# Issues — [Nome do Projeto]

> Baseado em: docs/SPEC.md
> Total: XX issues | Concluidas: 0/XX

## Visuais (so subtask V)

### issue-01 — Landing Page
[subtask V]

## Fundacao

### issue-02 — Login/Signup
[subtask V + subtask F]

## Features Core

### issue-04 — CRUD de Clientes
[subtask V + subtask F]

### issue-05 — Criar Tarefa
[subtask V + subtask F]

...

## Integracoes

### issue-07 — Pagamentos com Stripe
[subtask F]

...

## Polish

### issue-20 — Acessibilidade
[subtask F]

...
```

---

## Criterios de Qualidade do /break

- [ ] Toda pagina da spec tem uma issue correspondente
- [ ] Todo comportamento da spec esta coberto por uma subtask funcional
- [ ] Nenhuma issue e grande demais (se leva mais de 1 conversa, quebre mais)
- [ ] Cada issue com UI tem subtask V separada da subtask F
- [ ] Dependencias entre issues estao explicitas
- [ ] Ordem de implementacao faz sentido (fundacao → core → integracoes → polish)
- [ ] Subtasks funcionais referenciam os B-IDs da spec
- [ ] Criterios de aceitacao incluem testes
- [ ] Issues so-visuais (landing, marketing) nao tem subtask F desnecessaria
