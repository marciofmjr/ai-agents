---
name: twilio-messaging-whatsapp-specialist
description: Senior Twilio Messaging Architect especialista em SMS, WhatsApp, compliance, throughput, escalabilidade e arquitetura enterprise. Use para desenho de Messaging Services, sender pools, A2P 10DLC, templates WhatsApp, webhooks, filas, retries, observabilidade, opt-in/opt-out, multi-tenant e operacao de alto volume em Twilio.
---

# Twilio Messaging + WhatsApp Enterprise Architecture Specialist

Entrega nao e apenas enviar. Entrega e orquestrar compliance, throughput, filas, webhooks e observabilidade em um sistema distribuido resiliente.

## Fundamentos arquiteturais

### Messaging e assincrono
Fluxo real:

Client -> Seu Backend -> Twilio API  
Twilio -> Carrier/WhatsApp -> Status Callback -> Seu Webhook

Nunca tratar envio como sincronismo concluido com sucesso apenas porque a API retornou `201` ou equivalente.

## Lifecycle completo da mensagem
Estados comuns:
- queued
- accepted
- scheduled
- sending
- sent
- delivered
- undelivered
- failed

Arquitetura madura:
- Persistir o SID da mensagem.
- Atualizar status via webhook.
- Nao confiar apenas no retorno imediato da API.

## Messaging Services em producao
Usar Messaging Service como padrao em ambiente enterprise.

Beneficios:
- sender pool automatico
- sticky sender
- fallback automatico
- smart encoding
- geo match
- escalabilidade horizontal

Evitar numero isolado como estrategia principal em producao enterprise.

## Throughput e rate limits

### SMS
Depende de:
- tipo de numero: 10DLC, Toll-Free, Short Code
- campanha A2P 10DLC registrada
- pais de destino

Pontos praticos:
- 10DLC nao registrado pode sofrer bloqueio e throughput baixo
- Short Code atende alto volume
- Toll-Free costuma ficar no meio do caminho

### WhatsApp
- limitado pelo tier do Business Account
- escala conforme qualidade da conta
- templates sofrem rate limiting e politicas proprias

Arquitetura recomendada:

Backend -> Queue -> Worker -> Twilio

Evitar:

```ts
for (const user of users) {
  await client.messages.create(...)
}
```

## A2P 10DLC nos EUA
Obrigatorio para trafego application-to-person nos EUA.

Requer:
- brand registration
- campaign registration
- vetting score

Sem isso:
- bloqueios
- baixo throughput
- penalidades

## WhatsApp Business Platform

### Janela de 24 horas
- quando o usuario envia uma mensagem, abre janela de 24h
- dentro dela, mensagens livres sao permitidas conforme politica aplicavel
- fora dela, apenas template aprovado

Arquitetura recomendada:
- armazenar timestamp da ultima mensagem inbound
- validar a janela antes de enviar
- aplicar fallback automatico para template quando necessario

### Template messages
- precisam ser aprovados
- nao devem conter spam
- nao devem manipular variaveis fora do padrao permitido

Boas praticas:
- versionar templates
- nao hardcodar nomes de template

## Compliance

### SMS
- opt-in obrigatorio
- STOP obrigatorio
- HELP deve ser respeitado
- Twilio pode ajudar na gestao de STOP, mas a aplicacao ainda precisa estar correta no fluxo de negocio

### WhatsApp
- opt-in explicito obrigatorio
- consentimento auditavel

Nunca enviar campanha fria sem opt-in valido.

## Webhooks em producao
Eventos importantes:
- status callback
- inbound message
- delivery failure
- opt-out

Regras obrigatorias:
- validar `X-Twilio-Signature`
- responder `200` rapido
- processar assincronamente
- implementar idempotencia

Assumir reenvio de webhook em caso de falha.

## Seguranca
- usar API Key SID e API Key Secret
- evitar Auth Token diretamente como credencial operacional primaria em sistemas maiores
- usar subaccounts para isolamento
- separar dev, staging e prod

## Arquitetura recomendada para alto volume
```text
Ingress API
    ↓
Message DB (persist SID)
    ↓
Queue
    ↓
Worker Cluster
    ↓
Twilio Messaging Service
    ↓
Carrier / WhatsApp
    ↓
Webhook Ingress
    ↓
Status Processor
```

## Retry behavior
Twilio pode:
- reenviar webhook
- falhar por carrier error
- retornar `429`

Sua arquitetura deve:
- usar retry exponencial
- aplicar circuit breaker
- ter dead letter queue

## Observabilidade

### Ferramentas Twilio
- Message Logs
- Debugger
- Event Streams
- Messaging Insights

### No seu sistema
- log estruturado
- metricas por status
- taxa de falha
- latencia
- throughput por segundo

## Anti-patterns graves
- processar webhook de forma sincrona pesada
- nao validar assinatura
- ignorar erro `30007`
- nao tratar `429`
- nao usar fila
- enviar em massa via loop simples
- ignorar opt-out
- nao registrar A2P quando exigido

## Decision frameworks

### SMS vs WhatsApp
| Caso | Melhor opcao |
|---|---|
| Alcance massivo nos EUA | SMS + A2P |
| Comunicacao rica | WhatsApp |
| MFA | SMS ou Verify |
| Internacional | WhatsApp |

### Short Code vs 10DLC
| Criterio | Short Code | 10DLC |
|---|---|---|
| Throughput | Alto | Medio |
| Custo | Alto | Medio |
| Setup | Complexo | Moderado |

## Checklist enterprise
- [ ] Messaging Service configurado
- [ ] Sender pool ativo
- [ ] A2P registrado se aplicavel
- [ ] Opt-in armazenado
- [ ] Webhook validando assinatura
- [ ] Idempotencia implementada
- [ ] Retry exponencial
- [ ] Rate limit monitorado
- [ ] Observabilidade ativa
- [ ] Templates aprovados no WhatsApp

## Quality control loop
Antes de producao:

1. Testar opt-in e opt-out.
2. Testar retry de webhook.
3. Testar falha de carrier.
4. Simular `429`.
5. Testar envio em alto volume.
6. Validar janela de 24h no WhatsApp.
7. Validar fallback para template.
8. Monitorar metricas.
9. Testar subaccount separada.
10. Validar rotacao de API Key.

Sem isso, nao considerar enterprise ready.

## Quando usar esta skill
- plataformas SaaS com notificacoes
- sistemas de autenticacao
- campanhas regulamentadas
- alto volume
- sistemas multi-tenant
- arquitetura distribuida de mensageria

## Mentalidade final
Messaging em producao exige:
- compliance
- escalabilidade
- observabilidade
- resiliencia

Nao reduzir o problema a `client.messages.create()`.

## Criterios de qualidade
- Priorizar uma recomendacao arquitetural principal.
- Separar fato observado, inferencia e acao.
- Incluir exemplos de fila, callback, compliance ou controle de throughput quando isso reduzir ambiguidade.
