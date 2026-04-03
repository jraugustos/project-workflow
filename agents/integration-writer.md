# Agent: Integration Writer

Cria conexoes com servicos externos — Stripe, SendGrid, S3, APIs terceiras.
Qualquer arquivo em `src/integrations/`.

## Regras

### Estrutura de uma integracao

```typescript
// src/integrations/stripe.ts
import Stripe from 'stripe'

// 1. Client singleton
const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2024-06-20',
})

// 2. Funcoes tipadas e atomicas
export async function createCheckoutSession({
  priceId,
  userId,
  successUrl,
  cancelUrl,
}: {
  priceId: string
  userId: string
  successUrl: string
  cancelUrl: string
}) {
  return stripe.checkout.sessions.create({
    mode: 'subscription',
    payment_method_types: ['card'],
    line_items: [{ price: priceId, quantity: 1 }],
    metadata: { userId },
    success_url: successUrl,
    cancel_url: cancelUrl,
  })
}

export async function getSubscription(subscriptionId: string) {
  return stripe.subscriptions.retrieve(subscriptionId)
}

export async function cancelSubscription(subscriptionId: string) {
  return stripe.subscriptions.cancel(subscriptionId)
}
```

### Webhook handler

```typescript
// src/app/api/webhooks/stripe/route.ts
import { NextRequest, NextResponse } from 'next/server'
import Stripe from 'stripe'
import { prisma } from '@/lib/prisma'

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!)

export async function POST(req: NextRequest) {
  const body = await req.text()
  const signature = req.headers.get('stripe-signature')!

  let event: Stripe.Event

  try {
    event = stripe.webhooks.constructEvent(
      body,
      signature,
      process.env.STRIPE_WEBHOOK_SECRET!
    )
  } catch (err) {
    console.error('Webhook signature verification failed:', err)
    return NextResponse.json({ error: 'Invalid signature' }, { status: 400 })
  }

  switch (event.type) {
    case 'checkout.session.completed':
      // Handle successful checkout
      break
    case 'customer.subscription.deleted':
      // Handle cancellation
      break
    default:
      console.log(`Unhandled event type: ${event.type}`)
  }

  return NextResponse.json({ received: true })
}
```

### Principios

1. **Client como singleton** — instanciar uma vez, reutilizar
2. **Funcoes atomicas** — uma funcao por operacao (createCheckout, getSubscription)
3. **Env vars para secrets** — nunca hardcodar chaves
4. **Tipagem forte** — usar types da lib oficial
5. **Webhook com verificacao de assinatura** — sempre validar
6. **Logging** — logar eventos para debug

### Organizacao

```
src/integrations/
├── stripe.ts          ← Funcoes do Stripe
├── sendgrid.ts        ← Funcoes de email
├── s3.ts              ← Upload de arquivos
└── openai.ts          ← Chamadas de IA
```

### O que NAO fazer

- Nao hardcodar API keys
- Nao criar integracao sem verificacao de webhook
- Nao ignorar erros de servico externo (sempre catch + log)
- Nao misturar logica de negocio na integracao (ela so faz a chamada, action processa)

---

## Prompt de Execucao

Quando o /execute delegar a criacao de uma integracao, montar o prompt assim:

```
Crie a integracao com [SERVICO] em [PATH].

## Contexto
- Issue: [issue-id]
- Descricao: [o que a integracao faz]

## Servico externo
- Nome: [Stripe / Resend / S3 / etc.]
- Lib: [nome do pacote npm]
- Versao: [versao no projeto, de stack.md]
- Docs: [link ou usar Context7 MCP se disponivel]

## Funcoes necessarias
| Funcao | O que faz | Params | Retorno |
|---|---|---|---|
| [nome] | [descricao] | [params tipados] | [retorno tipado] |

## Env vars necessarias
- [VAR_NAME]: [descricao — onde obter]

## Webhook (se aplicavel)
- Endpoint: [path da route]
- Eventos: [quais eventos escutar]
- Verificacao de assinatura: [como]

## Testes esperados
- Mock do client → funcao retorna dados esperados
- Erro do servico → funcao trata gracefully
- Webhook com assinatura valida → processa
- Webhook com assinatura invalida → rejeita 400
```
