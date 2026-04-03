# Guia do /review — Code Review Pos-Execucao

O /review e o quinto comando do workflow. Roda DEPOIS do /execute, ANTES do merge.
Funciona como um code review automatizado que verifica se a implementacao
atende todos os requisitos da issue.

## Quando rodar

```
/execute → implementacao pronta → /review → aprovacao → merge
```

O /review roda na MESMA conversa do /execute (antes do /clear),
porque precisa do contexto do que acabou de ser implementado.

## Validacao em 3 Camadas

O /review verifica em 3 camadas distintas, cada uma com foco diferente.
Isso evita misturar "a feature funciona?" com "o codigo esta limpo?".

### Camada 1: Requisitos (a spec foi cumprida?)

Verificar se o que foi planejado no PLAN.md foi implementado.

- [ ] Todos os arquivos listados em "Criar" foram criados?
- [ ] Todos os arquivos listados em "Modificar" foram modificados?
- [ ] Nenhum arquivo listado em "NAO tocar" foi alterado?
- [ ] Funcionalidade descrita no "Caminho feliz" funciona?
- [ ] Edge cases listados estao tratados?
- [ ] Cenarios de erro estao tratados?
- [ ] Testes unitarios existem e passam
- [ ] Testes E2E existem e passam (para issues funcionais)
- [ ] Cenarios do PLAN.md estao cobertos nos testes

### Camada 2: Design (visual confere com referencia?)

Verificar se a implementacao visual esta alinhada com o design system e referencia.
**Aplicar apenas em issues com componentes visuais (prototipos e funcionais com UI).**

- [ ] Componentes visuais usam tokens do design system (nao cores/espacamentos hardcoded)
- [ ] Componentes base reutilizados (Button, Input, Card de `src/components/ui/`)
- [ ] Responsividade implementada (mobile-first)
- [ ] Acessibilidade basica (labels, aria attributes)
- [ ] Visual confere com referencia (wireframe, Figma, screenshot — se disponivel)
- [ ] UX patterns seguidos (toasts, loading, erros — conforme .claude/docs/ux-patterns.md)

### Camada 3: Implementacao (codigo segue padroes?)

Verificar qualidade de codigo, arquitetura e seguranca.

**Qualidade:**
- [ ] Sem `any` nos types TypeScript
- [ ] Sem `console.log` de debug (apenas `console.error` em catches)
- [ ] Sem codigo comentado ou dead code
- [ ] Sem dependencias nao utilizadas
- [ ] Naming consistente com o projeto (verificar .claude/docs/architecture.md)
- [ ] Imports organizados

**Arquitetura:**
- [ ] Arquivos no lugar certo (actions/ para server actions, hooks/ para hooks, etc.)
- [ ] Separacao de responsabilidades (componente nao faz fetch, action nao renderiza)
- [ ] Patterns do projeto seguidos (verificar .claude/docs/architecture.md)
- [ ] Server vs Client components correto ('use client' so onde necessario)

**Seguranca:**
- [ ] Auth check em todas as actions/routes que precisam
- [ ] Validacao de input com Zod (nao confiar no client)
- [ ] Sem dados sensiveis expostos no client (tokens, secrets)
- [ ] Sem SQL injection / query unsafe (usando ORM corretamente)

---

## Fluxo do /review

```
1. Ler o PLAN.md da issue (para saber o que era esperado)
2. Ler o diff completo (git diff main...issue/[id])
3. Verificar cada item do checklist acima
4. Gerar relatorio de review
5. Se tudo OK → "Pronto para merge"
6. Se tem problemas → listar problemas com sugestao de fix
7. Se tem problemas criticos → corrigir antes de aprovar
```

### Prompt para iniciar o /review

```
Faca o review da implementacao da issue [id].
Leia docs/issues/[id]/PLAN.md para saber o que era esperado.
Rode git diff main...issue/[id] para ver o que foi implementado.
Verifique conformidade com PLAN, design system, arquitetura, testes e seguranca.
```

---

## Output: Relatorio de Review

```markdown
# Review — [issue-id]

## Status: ✅ Aprovado / ⚠️ Aprovado com ressalvas / ❌ Requer correcoes

---

## Camada 1: Requisitos
- ✅ Todos os arquivos criados conforme plano
- ✅ Caminho feliz implementado
- ⚠️ Edge case "titulo vazio" tratado mas sem mensagem descritiva
- ✅ 4 testes unitarios passando
- ✅ 1 teste E2E passando
- ⚠️ Edge case "deadline no passado" nao tem teste
**Resultado: ⚠️**

## Camada 2: Design
- ✅ Usando Button e Input de components/ui/
- ✅ Cores do design system
- ❌ Espacamento hardcoded `p-3` deveria ser `p-4` conforme design tokens
- ✅ Responsivo em mobile e desktop
- ⚠️ Falta aria-label no botao de submit
**Resultado: ❌** (tem bloqueante)

## Camada 3: Implementacao
- ✅ Types corretos, sem `any`
- ✅ Sem console.log de debug
- ⚠️ Import nao utilizado em TaskForm.tsx (line 5)
- ✅ Separacao de responsabilidades correta
- ✅ Auth check presente
- ✅ Validacao Zod no server
**Resultado: ⚠️**

---

## Resumo
| Camada | Status |
|---|---|
| Requisitos | ⚠️ Aprovado com ressalvas |
| Design | ❌ Requer correcao |
| Implementacao | ⚠️ Aprovado com ressalvas |

## Acoes necessarias
1. ❌ Corrigir espacamento no TaskForm (p-3 → p-4) [Design]
2. ⚠️ Adicionar teste para edge case "deadline no passado" [Requisitos]
3. ⚠️ Remover import nao utilizado [Implementacao]
4. ⚠️ Adicionar aria-label ao botao de submit [Design]
```

---

## Severidade dos problemas

| Icone | Severidade | Acao |
|---|---|---|
| ✅ | OK | Nada a fazer |
| ⚠️ | Ressalva | Pode mergear, mas idealmente corrigir |
| ❌ | Bloqueante | Corrigir ANTES de mergear |

Problemas ❌ devem ser corrigidos na mesma conversa antes de aprovar o merge.
Problemas ⚠️ podem ser aceitos pelo usuario ou corrigidos — decisao dele.
