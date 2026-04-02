---
name: twilio-nestjs-messaging-architect
description: Senior Architect especialista em integracao Twilio Messaging com SMS e WhatsApp usando NestJS em producao enterprise. Use para desenhar controllers, services, filas, workers, webhook guards, idempotencia, status callbacks, multi-tenant com subaccounts, observabilidade e escalabilidade em backends NestJS.
---

# Twilio Messaging + NestJS Architecture Specialist

Twilio e orientado a eventos. NestJS e orientado a arquitetura. Integrar os dois como sistema distribuido, nunca como simples chamada HTTP.

## Mindset
- Webhook e entrada publica critica.
- Mensageria e assincrona.
- Status callbacks atualizam o estado interno.
- Idempotencia e obrigatoria.
- Processamento pesado nunca acontece no controller.
- Filas sao parte da arquitetura, nao opcional.
- Seguranca comeca validando assinatura Twilio.
- Observabilidade e obrigatoria em producao.

## Arquitetura ideal
```text
Client / System
        ↓
NestJS API (Controller)
        ↓
Application Service
        ↓
Queue (SQS / Kafka / BullMQ)
        ↓
Worker (NestJS Microservice)
        ↓
Twilio Messaging Service
        ↓
Carrier / WhatsApp
        ↓
Webhook Controller (NestJS)
        ↓
Event Processor
        ↓
Database Update
```

## Estrutura de pastas recomendada
```text
src/
 ├── modules/
 │   ├── messaging/
 │   │   ├── messaging.module.ts
 │   │   ├── messaging.controller.ts
 │   │   ├── messaging.service.ts
 │   │   ├── messaging.webhook.controller.ts
 │   │   ├── messaging.processor.ts
 │   │   ├── twilio.client.ts
 │   │   ├── dto/
 │   │   ├── guards/
 │   │   └── interceptors/
 │   └── queue/
 ├── common/
 │   ├── guards/
 │   ├── filters/
 │   ├── interceptors/
 └── config/
```

## Componentes arquiteturais

### 1. Twilio client provider
Nao instanciar o client diretamente no service.

```ts
@Injectable()
export class TwilioClientProvider {
  private client: Twilio;

  constructor(private config: ConfigService) {
    this.client = new Twilio(
      config.get('TWILIO_API_KEY'),
      config.get('TWILIO_API_SECRET'),
      { accountSid: config.get('TWILIO_ACCOUNT_SID') }
    );
  }

  getClient() {
    return this.client;
  }
}
```

### 2. Messaging service na application layer
```ts
@Injectable()
export class MessagingService {
  constructor(
    private readonly twilioProvider: TwilioClientProvider,
  ) {}

  async sendSMS(dto: SendMessageDto) {
    const client = this.twilioProvider.getClient();

    return client.messages.create({
      to: dto.to,
      from: dto.from,
      body: dto.body,
      messagingServiceSid: dto.messagingServiceSid,
      statusCallback: dto.statusCallback,
    });
  }
}
```

Nao chamar isso diretamente do controller HTTP.

### 3. Thin controller pattern
```ts
@Post()
async send(@Body() dto: SendMessageDto) {
  await this.queueService.enqueue(dto);
  return { status: 'queued' };
}
```

Controller nao envia direto.

## Webhook seguro

### Validacao de `X-Twilio-Signature`
Criar guard dedicado:

```ts
@Injectable()
export class TwilioWebhookGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    const signature = request.headers['x-twilio-signature'];

    const isValid = validateRequest(
      process.env.TWILIO_AUTH_TOKEN,
      signature,
      request.originalUrl,
      request.body
    );

    return isValid;
  }
}
```

Aplicar no webhook controller e isolar a validacao em camada clara de seguranca.

## WhatsApp no NestJS

### Estrategia da janela de 24 horas
```ts
async canSendFreeMessage(userId: string): Promise<boolean> {
  const lastInbound = await this.repo.getLastInbound(userId);
  return dayjs().diff(lastInbound, 'hour') < 24;
}
```

Se nao puder enviar mensagem livre, usar template aprovado.

## Idempotencia
Webhook pode ser reenviado. Tratar como comportamento normal.

Estrategia:
- criar tabela ou store de `webhook_events`
- salvar `MessageSid` e status
- ignorar evento ja processado

## Retry strategy
Quando ocorrer `429` ou erro `5xx`:
- usar retry exponencial
- enviar para DLQ apos limite definido
- nunca aplicar retry infinito

## Fila com BullMQ
```ts
@Processor('messaging')
export class MessagingProcessor {
  @Process()
  async handle(job: Job<SendMessageDto>) {
    await this.messagingService.sendSMS(job.data);
  }
}
```

## Multi-tenant com subaccounts
Estrategia enterprise:
- uma subaccount por cliente quando isolamento for requisito
- API key isolada por subaccount
- billing isolado
- rate limit isolado

Evitar misturar tenants sensiveis no mesmo account SID sem justificativa operacional forte.

## Observabilidade
Implementar:
- interceptor de logging
- logs estruturados
- metricas por status
- metrica de erro por carrier
- alertas para `30007`
- monitoramento de `429`

## Decision frameworks

### Direct send vs queue
| Situacao | Arquitetura |
|---|---|
| MVP | Direct |
| Producao | Queue |
| Alto volume | Queue + worker cluster |

### MessagingService vs from number
| Cenario | Use |
|---|---|
| Escala | MessagingService |
| Teste rapido | From number |

## Boas praticas

### Arquitetura
- thin controllers
- services focados em aplicacao
- filas obrigatorias em producao
- webhook validado
- idempotencia

### Seguranca
- API Keys
- subaccounts
- guard para webhook
- `ConfigModule`

### Performance
- cluster NestJS quando necessario
- horizontal scaling
- rate limit awareness
- monitoramento de throughput

## Anti-patterns
- controller enviando direto
- nao usar Messaging Service em cenario de escala
- ignorar A2P quando aplicavel
- nao tratar status callbacks
- webhook sem validacao
- nao usar fila
- nao tratar erro de carrier

## Checklist de review
- [ ] Messaging Service configurado
- [ ] Queue implementada
- [ ] Webhook guard ativo
- [ ] Idempotencia ativa
- [ ] Retry strategy definida
- [ ] Observabilidade ativa
- [ ] Subaccounts isoladas quando necessario
- [ ] A2P registrado
- [ ] Templates WhatsApp aprovados

## Quality control loop
Antes de producao:

1. Testar envio de SMS.
2. Testar envio de WhatsApp.
3. Testar janela de 24h.
4. Testar fallback para template.
5. Simular retry de webhook.
6. Simular erro `429`.
7. Simular erro `30007`.
8. Testar alto volume.
9. Validar assinatura manualmente.
10. Monitorar logs e metricas.

Sem isso, nao considerar pronto para producao.

## Quando usar esta skill
- SaaS multi-tenant
- plataforma de notificacoes
- sistemas OTP
- WhatsApp enterprise
- alto volume
- arquitetura distribuida
- backend NestJS robusto

## Mentalidade final
Twilio + NestJS bem feito e:
- event-driven
- modular
- escalavel
- seguro
- observavel

Nao reduzir a integracao a:

`controller -> client.messages.create()`

## Criterios de qualidade
- Priorizar desenho modular e desacoplado.
- Separar claramente webhook ingress, fila, worker e atualizacao de estado.
- Incluir exemplos de NestJS apenas quando ajudarem a concretizar a arquitetura recomendada.
