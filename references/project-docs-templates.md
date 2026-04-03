# Templates dos Documentos Internos (.claude/docs/)

Estes templates guiam a criacao dos documentos que a IA le antes de implementar.
Criar na Etapa 0 (Setup) para projetos novos, ou gerar escaneando o codebase para projetos existentes.

---

## stack.md

```markdown
# Stack Tecnica

## Core
- **Runtime:** [ex: Node.js 20 LTS]
- **Framework:** [ex: Next.js 14.2 (App Router)]
- **Linguagem:** [ex: TypeScript 5.4 (strict mode)]

## Frontend
- **Styling:** [ex: Tailwind CSS 3.4]
- **Componentes UI:** [ex: shadcn/ui (Radix primitives)]
- **Icones:** [ex: Lucide React]
- **Forms:** [ex: React Hook Form + Zod]
- **State management:** [ex: Zustand / React Context / nenhum]

## Backend
- **ORM:** [ex: Prisma 5.x]
- **Banco de dados:** [ex: PostgreSQL 16 (Supabase)]
- **Auth:** [ex: NextAuth.js 5 (Auth.js)]
- **Email:** [ex: Resend]
- **Pagamentos:** [ex: Stripe]
- **Storage:** [ex: Supabase Storage / S3]

## Infra
- **Hosting:** [ex: Vercel]
- **CI/CD:** [ex: GitHub Actions]
- **Monitoramento:** [ex: Vercel Analytics + Sentry]
- **DNS:** [ex: Cloudflare]

## Versoes Importantes
[Listar versoes que afetam APIs, ex:]
- Next.js 14.2 usa App Router (nao Pages Router)
- NextAuth v5 usa `auth()` (nao `getServerSession()`)
- Prisma 5 usa `$extends` (nao middleware)
```

---

## architecture.md

```markdown
# Arquitetura

## Estrutura de Pastas
[Arvore de pastas com explicacao de cada diretorio]

## Convencoes de Naming
- Arquivos: [ex: kebab-case para componentes, camelCase para utilitarios]
- Componentes: [ex: PascalCase]
- Hooks: [ex: useCamelCase]
- Actions: [ex: camelCase — createTask, deleteClient]
- Routes: [ex: kebab-case — /api/user-profile]

## Patterns Estabelecidos
- **Server Actions:** [ex: Zod validation → auth check → prisma operation → revalidatePath]
- **Error handling:** [ex: try/catch em toda action, retorna { data } ou { error }]
- **Paginacao:** [ex: cursor-based com limit default 20]
- **Loading states:** [ex: useTransition para pending, Skeleton components para loading]
- **Forms:** [ex: React Hook Form + Zod schema, validacao client + server]

## Separacao de Responsabilidades
- Componentes: so renderizam, nao fazem fetch nem logica de negocio
- Hooks: conectam UI com actions, gerenciam estado local
- Actions: logica de negocio no servidor, validacao, persistencia
- Lib/utils: funcoes puras reutilizaveis (formatDate, cn, etc.)
```

---

## design-system.md

```markdown
# Design System

## Cores
- Primary: #2563EB (blue-600)
- Primary hover: #1D4ED8 (blue-700)
- Secondary: #64748B (slate-500)
- Background: #FFFFFF
- Surface: #F8FAFC (slate-50)
- Border: #E2E8F0 (slate-200)
- Text primary: #0F172A (slate-900)
- Text secondary: #64748B (slate-500)
- Success: #16A34A (green-600)
- Error: #DC2626 (red-600)
- Warning: #D97706 (amber-600)

## Tipografia
- Font family: Inter
- Heading 1: 2.25rem / bold
- Heading 2: 1.5rem / semibold
- Heading 3: 1.25rem / semibold
- Body: 1rem / regular
- Small: 0.875rem / regular
- Caption: 0.75rem / regular

## Espacamento
- Base unit: 4px
- xs: 4px, sm: 8px, md: 16px, lg: 24px, xl: 32px, 2xl: 48px

## Bordas
- Radius sm: 4px, md: 8px, lg: 12px, full: 9999px
- Border: 1px solid border-color

## Sombras
- sm: 0 1px 2px rgba(0,0,0,0.05)
- md: 0 4px 6px rgba(0,0,0,0.1)
- lg: 0 10px 15px rgba(0,0,0,0.1)

## Componentes Base (src/components/ui/)
[Listar todos os componentes base disponiveis:]
- Button: variants (primary, secondary, ghost, destructive), sizes (sm, md, lg)
- Input: text, email, password, com label e error state
- Select: dropdown com opcoes
- Card: container com header, content, footer
- Badge: status indicators com cores
- Modal/Dialog: overlay com conteudo
- Toast: feedback notifications (success, error, info)
- Skeleton: loading placeholder

## Fonte de Interface
- Ferramenta: [Figma / Google Stitch / outra / nenhuma]
- Link: [URL do arquivo de design, se disponivel]
- MCP disponivel: [Sim/Nao — se sim, usar para extrair tokens e componentes]

## Tailwind Config
[Resumo do tailwind.config customizado, se houver]
```

---

## business-rules.md

```markdown
# Regras de Negocio

## Usuarios
- [ex: Usuario free pode criar ate 3 projetos]
- [ex: Usuario Pro pode criar projetos ilimitados]
- [ex: Conta deletada mantem dados por 30 dias antes de apagar]

## [Entidade principal 1, ex: Tarefas]
- [ex: Tarefa atrasada muda badge para vermelho automaticamente]
- [ex: Tarefa concluida nao pode ser editada, so reaberta]
- [ex: Deadline e opcional, mas se definida, nao pode ser no passado]

## [Entidade principal 2, ex: Clientes]
- [ex: Cliente so pode ser deletado se nao tem tarefas ativas]
- [ex: Nome do cliente e unico por usuario]

## Permissoes
- [ex: Dono do workspace pode convidar membros]
- [ex: Membro pode ver e editar tarefas, mas nao deletar clientes]

## Integracoes
- [ex: Webhook do Stripe atualiza status do plano em tempo real]
- [ex: Email de boas-vindas dispara 5 min apos signup (nao instantaneo)]

## Validacoes Globais
- [ex: Todos os campos de texto tem limite de 500 caracteres]
- [ex: Uploads limitados a 10MB]
- [ex: Rate limit de 100 requests/min por usuario]
```

---

## ux-patterns.md

```markdown
# Padroes de UX

## Feedback
- **Toast de sucesso:** aparece no canto superior direito, desaparece em 3s
- **Toast de erro:** aparece no canto superior direito, persiste ate fechar
- **Validacao de form:** inline, aparece ao sair do campo (onBlur) e ao submeter
- **Loading (mutacao):** botao fica disabled com spinner
- **Loading (pagina):** skeleton matching o layout final

## Navegacao
- **Apos criar:** redireciona para lista/dashboard
- **Apos deletar:** redireciona para lista + toast confirmando
- **Apos editar:** permanece na mesma pagina + toast confirmando
- **Erro de auth:** redireciona para /login com query param ?redirect=

## Empty States
- Cada lista tem empty state com ilustracao + CTA para criar primeiro item
- Empty state de busca: "Nenhum resultado para [query]"

## Modais
- Confirmacao de delete: modal com "Tem certeza?" + botao destructive
- Modais fecham com Escape e click fora

## Responsividade
- Mobile-first
- Sidebar collapsa em hamburger no mobile
- Tabelas viram cards empilhados no mobile

## Acessibilidade
- Todo input tem label associado
- Focus visible em todos os interativos
- Navegacao por teclado funciona
- Contraste minimo AA (4.5:1 para texto)
```
