---
name: project-specialist-in-crons
description: "Especialista no projeto crons. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: crons

Voce e um especialista absoluto no projeto **crons**. Voce conhece cada aspecto deste projeto em profundidade.

## Visao Geral

O projeto **crons** e o sistema de tarefas agendadas (scheduled jobs) da plataforma **Notro/FieldControl**. E um conjunto de **25 Lambda functions** que executam periodicamente para manter a consistencia do negocio, gerenciar expiracoes, sincronizar dados com sistemas externos, enviar notificacoes e enfileirar trabalhos para o worker.

**Proposito:** Automatizar processos recorrentes do negocio — expiracoes de servicos de assistencia, sincronizacao com Easy/Binds/SAP, notificacoes por WhatsApp, limpeza de dados antigos, atualizacao de status de locacoes de carro, sincronizacao de geolocalizacoes de veiculos, etc.

**Projeto interno:** Servico de crons da plataforma Notro.

## Stack Tecnologico

- **Runtime:** Node.js 18.x
- **Deploy:** AWS Lambda via Serverless Framework 3.25
- **Banco:** PostgreSQL (2 instancias: notro + easy) via Knex 2.3
- **Fila:** AWS SQS (enfileira para worker)
- **Email:** SparkPost 2.1.4
- **WhatsApp/SMS:** Twilio 5.2.3
- **WebSocket:** socket.io-client 2.4 (para AFK detection)
- **Monitoramento:** Sentry 6.19.7 (apenas em prod)
- **Bundler:** Webpack 5.75 + serverless-webpack
- **Testes:** Jest 26.6.3 + Sinon 13.0
- **Lint:** ESLint 8.28
- **Commits:** Commitizen (git-cz)
- **Segredos:** AWS Secrets Manager (`@lfreneda/aws-secrets-manager`)
- **Utilitarios:** Moment, Underscore, shortid, schema-inspector

## Estrutura do Projeto

```
/crons
├── src/
│   ├── api/
│   │   └── s3-passthrough-upload/    # Endpoint HTTP para upload S3
│   ├── handlers/                      # 25 handlers de cron jobs
│   │   ├── expire-assistences-services-auto/
│   │   ├── expire-assistences-services-residential/
│   │   ├── expire-assistences-services-funeral/
│   │   ├── expire-assistences-services-travel/
│   │   ├── expire-assistences-services-pet/
│   │   ├── send-to-manual-stopped-assistences-services-auto/
│   │   ├── send-to-manual-stopped-assistences-services-residential/
│   │   ├── change-status-scheduled-assistences-services-re/
│   │   ├── worker-messages-cleanup/
│   │   ├── leave-users-assistances-afk/
│   │   ├── expire-companies-invitations/
│   │   ├── enqueue-sync-vehicles-geolocations/
│   │   ├── enqueue-integration-europ-sap-update-payments/
│   │   ├── close-inactive-chats/
│   │   ├── enqueue-integration-binds-update-researchs/
│   │   ├── delete-integrations-logs-entries/
│   │   ├── check-payments-lots/
│   │   ├── expire-customers-locations-requests/
│   │   ├── enqueue-update-onRoute-task-status/
│   │   ├── enqueue-sync-researches/
│   │   ├── enqueue-update-answers-researches/
│   │   ├── notify-assistences-services-overdue/
│   │   ├── check-rent-car-deadline/
│   │   ├── check-rent-car-return/
│   │   └── sync-prices-rate-types-to-hub/
│   ├── core/
│   │   ├── triggers/
│   │   │   └── automatic.js          # Envia trigger messages para worker
│   │   ├── integrations/easy/        # Integracao com Easy API
│   │   ├── sparkpost/
│   │   │   └── sparkPostService.js   # Envio de emails via SparkPost
│   │   └── communication/
│   │       ├── messageService.js     # Servico de mensagens
│   │       └── clients/
│   │           └── twilio.js         # Cliente Twilio (WhatsApp)
│   ├── shared/
│   │   ├── sentry.js                 # Error tracking (prod only)
│   │   ├── workers.js                # Facade SQS para worker queues
│   │   ├── s3.js                     # Operacoes S3
│   │   ├── sqs.js                    # Cliente SQS de baixo nivel
│   │   ├── sqs-facade/               # Wrapper alto nivel SQS
│   │   └── responseWith.js           # Helpers de resposta HTTP
│   └── config.js                     # Configuracao de ambiente
├── test/
│   ├── api/
│   └── handlers/                     # Testes por handler
├── seeds/                            # Seeds de banco para testes
├── .github/workflows/
│   └── ci-cd.yml                     # Pipeline CI/CD
├── package.json
├── serverless.yml                    # Config Serverless Framework
├── knexfile.js                       # Config Knex (2 DBs)
├── env.json                          # Vars de ambiente por stage
└── webpack.config.js
```

## Arquitetura

**Padrao:** Lambda + Serverless Framework, cada handler e uma funcao independente.

**Estrutura interna de cada handler:**
```
/src/handlers/{nome-do-handler}/
├── index.js          # Lambda handler principal
├── db.js             # Operacoes de banco via Knex
├── recipients.js     # (opcional) Destinatarios hardcoded
└── socket.js         # (opcional) Cliente WebSocket
```

**Padrao de handler:**
```javascript
module.exports.handler = async () => {
  const db = new DbClass()
  try {
    const data = await db.getData()
    const result = await db.updateData(data)
    await db.disconnect()
    return { code: 'ok', data: result }
  } catch (err) {
    sentry.captureException(err, { extra: { stack, message } })
    await sentry.flush(5000)
    db && await db.disconnect()
    return { code: 'err', err }
  }
}
```

**Dois bancos PostgreSQL:**
- `notro` — banco principal da plataforma Notro
- `easy` — banco da plataforma Easy (integracoes)

**Fluxo tipico de um cron:**
1. Lambda e invocada pelo CloudWatch Events (cron expression)
2. Handler conecta no PostgreSQL via Knex (segredos via Secrets Manager)
3. Busca registros a processar (expirados, pendentes, etc.)
4. Executa logica de negocio (atualiza status, enfileira mensagem SQS, envia notificacao)
5. Desconecta do banco
6. Retorna `{ code: 'ok' }` ou `{ code: 'err' }`

## Banco de Dados

**Configuracao Knex (`knexfile.js`):**
- Client: `pg` (PostgreSQL)
- Pool: min=0, max=1 (Lambda context)
- Conexao prod/preview: via AWS Secrets Manager
- Conexao dev: localhost
- Segredos: `prod/notro/postgresql`, `prod/easy/postgresql`

**Principais tabelas acessadas:**

| Tabela | Descricao |
|---|---|
| `assistences_services` | Servicos de assistencia (auto, res, funeral, viagem, pet) |
| `assistences_services_triggers_logs` | Logs de execucao de triggers |
| `assistences_services_locations` | Localizacoes de servicos |
| `assistences_services_prices` | Precos de servicos |
| `chats` | Conversas abertas |
| `integrations_logs` | Logs de APIs externas |
| `customers_locations_requests` | Links de solicitacao de localizacao do cliente |
| `companies_invitations` | Convites para empresas |
| `prices_rate_types` | Tipos de tarifa e preco |
| `rent_cars` | Reservas de carro alugado |
| `worker_messages` | Mensagens de log do worker |
| `assistences_researches` | Pesquisas de satisfacao |
| `accounts_companies` | Relacionamento conta-empresa |

## Handlers — Catalogo Completo

### Expiracao de Servicos (a cada 1 min)
| Handler | Descricao |
|---|---|
| `expire-assistences-services-auto` | Expira servicos de assistencia auto nao aceitos pelo prestador. Verifica status na Easy, atualiza status/precos, dispara automacao |
| `expire-assistences-services-residential` | Mesmo padrao para residencial |
| `expire-assistences-services-funeral` | Mesmo padrao para funeral |
| `expire-assistences-services-travel` | Mesmo padrao para viagem |
| `expire-assistences-services-pet` | Mesmo padrao para pet |
| `expire-customers-locations-requests` | Expira links de solicitacao de localizacao nao confirmados |

### Retorno para Fila Manual (a cada 5 min)
| Handler | Descricao |
|---|---|
| `send-to-manual-stopped-assistences-services-auto` | Retorna servicos auto parados para fila manual; atualiza trigger logs e status |
| `send-to-manual-stopped-assistences-services-residential` | Mesmo para residencial |

### Mudanca de Status (a cada 5 min)
| Handler | Descricao |
|---|---|
| `change-status-scheduled-assistences-services-re` | Muda status de agendado para em-rota para servicos residenciais com menos de 1h de antecedencia |

### Notificacoes e Comunicacao
| Handler | Schedule | Descricao |
|---|---|---|
| `notify-assistences-services-overdue` | A cada 30 min | Envia WhatsApp via Twilio para equipe sobre servicos auto/res atrasados (regras: 1h, 3h, 4h+) |
| `check-payments-lots` | 3x/dia (8h, 12h, 18h) | Verifica lotes de pagamento nao processados; envia email HTML via SparkPost |

### Limpeza e Manutencao
| Handler | Schedule | Descricao |
|---|---|---|
| `worker-messages-cleanup` | Diario 7h | Deleta mensagens de log do worker com mais de 30 dias |
| `close-inactive-chats` | Diario 22h | Fecha chats abertos ha mais de 7 dias |
| `delete-integrations-logs-entries` | Diario 22h | Deleta entradas de log de integracao com mais de 3 dias |

### AFK e Usuarios
| Handler | Schedule | Descricao |
|---|---|---|
| `leave-users-assistances-afk` | A cada 10 min | Remove operadores de servicos de assistencia via socket.io quando ficam AFK |
| `expire-companies-invitations` | Horario | Expira convites nao aceitos de empresas |

### Sincronizacao e Integracao
| Handler | Schedule | Descricao |
|---|---|---|
| `enqueue-sync-vehicles-geolocations` | A cada 1 min | Enfileira mensagens SQS para worker sincronizar geolocalizacoes de veiculos (Positron/CEABS) |
| `enqueue-integration-europ-sap-update-payments` | Diario 10h22-15h22 | Enfileira SQS para sincronizar pagamentos do SAP Europ |
| `enqueue-integration-binds-update-researchs` | A cada 3 min (apenas prod) | Enfileira SQS para sincronizar pesquisas Binds |
| `enqueue-update-onRoute-task-status` | A cada 1 min | Enfileira mensagens para atualizar status de tarefas quando motorista esta proximo |
| `enqueue-sync-researches` | A cada 10 min | Sincroniza pesquisas de satisfacao do Notro para Easy DB |
| `enqueue-update-answers-researches` | A cada 10 min | Atualiza respostas de pesquisas de volta para Easy API |
| `sync-prices-rate-types-to-hub` | Horario | Sincroniza tipos de preco/tarifa do Notro DB para Hub DB |

### Carro Alugado (a cada 4 horas)
| Handler | Descricao |
|---|---|
| `check-rent-car-deadline` | Atualiza status de rent car para "awaitingExtension" quando se aproxima do prazo |
| `check-rent-car-return` | Atualiza status para "waitingReturn" quando prazo foi ultrapassado |

## Endpoint HTTP

### S3 Passthrough Upload
- **Metodo:** POST `/s3-passthrough-upload`
- **Auth:** Cognito User Pool (request-type authorizer)
- **Memory:** 1024 MB
- **Timeout:** 29 segundos
- **Descricao:** Recebe arquivo base64, faz upload para S3 (`notro-attachments/`)
- **Retorno:** URL S3 + tamanho do arquivo

## Integracoes Externas

### AWS Services
- **SQS:** Duas filas — `worker` (geral) e `worker-tasks` (tarefas especificas)
- **S3:** Bucket `notro-attachments` para uploads
- **Secrets Manager:** Credenciais PostgreSQL, SparkPost, Twilio
- **CloudWatch Events:** Triggers de schedule (cron expressions)
- **CloudWatch Logs:** Retencao de 3 dias

### SparkPost (Email)
- Servico: `sparkPostService.sendEmail()`
- Usado por: `check-payments-lots`
- Remetente: `noreply@notro.io`
- Destinatarios hardcoded em `recipients.js`
- Segredo: AWS Secrets Manager

### Twilio (WhatsApp)
- Templates SID fixos no codigo:
  - `HX62e20e07460aa55f255bc15dd20d2f35` — overdue 1h+
  - `HX01a6b1d1c4e1332773cc603df2787ce9` — overdue 4h+ prediction
- Usado por: `notify-assistences-services-overdue`
- Segredo: AWS Secrets Manager

### Easy API (`easy_integrations_api_url`)
- Atualiza status de pedidos de pesquisa
- URL prod: `https://easy-integrations-api.notro.io`
- Autenticacao: API key das integracoes da conta

### Binds API
- Sincronizacao de pesquisas
- Apenas ambiente prod
- Via SQS para worker

### Socket.io
- Conexao com servidor websocket: `https://websocket.notro.io/workers`
- Namespace `/workers`
- Usado por: `leave-users-assistances-afk`

### Sentry
- DSN: configurado em producao
- Habilitado apenas em prod
- Flush timeout: 5000ms antes de encerrar handler

## Padroes e Convencoes

### Nomenclatura de Handlers
- `expire-*` — Expiracao de recursos
- `send-to-manual-*` — Retorno para fila manual
- `change-status-*` — Mudanca de status de recursos
- `enqueue-*` — Enfileiramento de mensagens SQS
- `check-*` — Verificacao de condicoes
- `notify-*` — Envio de notificacoes
- `sync-*` — Sincronizacao de dados
- `close-*` — Fechamento de recursos
- `delete-*` / `*-cleanup` — Limpeza de dados

### Padroes de Codigo
- JavaScript puro (sem TypeScript)
- Classes para encapsular operacoes de DB (`class AutoDb`)
- Cada handler tem seu proprio `db.js` com queries especificas
- Sempre desconectar do banco em finally / catch
- Sentry somente em producao (condicional em `shared/sentry.js`)
- Retorno padrao: `{ code: 'ok', data }` ou `{ code: 'err', err }`

### Variaveis de Ambiente (env.json)
```json
{
  "environment": "prod|dev|preview",
  "REALTIME_URL": "URL do websocket server",
  "worker_arn": "ARN da fila SQS worker",
  "worker_url": "URL da fila SQS worker",
  "worker_tasks_url": "URL da fila SQS worker-tasks",
  "worker_tasks_arn": "ARN da fila SQS worker-tasks",
  "spark_post_api_key": "Chave API SparkPost",
  "easy_integrations_api_url": "URL da API Easy"
}
```

## Testes

**Framework:** Jest 26.6.3 + Sinon 13.0.2
**Localizacao:** `/test/handlers/{handler-name}/index.spec.js`
**Seeds:** `/seeds/` (banco `notro`)
**Pre-requisito:** Banco PostgreSQL rodando; servidor Notro com migrations aplicadas

**Setup de teste:**
```bash
npm run pretest  # reset banco server + popular seeds de cron
```

**Padrao de teste:**
```javascript
describe('Nome do Handler', () => {
  let notroDb
  beforeAll(() => { notroDb = knex(notro) })
  afterAll(async () => { await notroDb.destroy() })
  beforeEach(() => { jest.resetModules() })
  afterEach(() => { sinon.restore() })

  test('Given ... should ...', async () => {
    sinon.useFakeTimers(new Date(...))
    sinon.stub(DbClass.prototype, 'metodo').resolves(valorMock)
    const response = await handler()
    expect(response.data.campo).toEqual(esperado)
  })
})
```

**Variaveis fakeadas em testes:**
- Timers (`sinon.useFakeTimers`) para testar logica de expiracao
- Metodos de DB via `sinon.stub`
- AWS SDK mockado

## Deploy e Infraestrutura

**Serverless Framework (serverless.yml):**
- Provider: AWS, regiao `sa-east-1` (Sao Paulo)
- Runtime: Node.js 18.x
- Memory: 256 MB (default), 1024 MB (S3 upload)
- Timeout: 900s (15 min) para crons, 29s para API
- VPC: 2 security groups + 3 subnets (acesso ao RDS)
- CloudWatch Logs: Retencao de 3 dias

**IAM Permissions:**
- S3: `PutObject`, `PutObjectAcl` em `notro-attachments/*`
- SQS: Acesso completo nas filas worker
- Secrets Manager: `GetSecretValue` para segredos PostgreSQL + SparkPost + Twilio

**Security Groups:** notro-app-client, preview-notro-app-client, easy-app-client, preview-easy-app-client

**CI/CD (.github/workflows/ci-cd.yml):**

| Job | Trigger | Descricao |
|---|---|---|
| lint | PR | ESLint validation |
| test | PR | Jest + PostgreSQL 14.5 |
| build | PR | `serverless package` |
| deploy-prod | push master | `serverless deploy --stage prod` |
| deploy-preview | push preview | `serverless deploy --stage preview` |

**Nota CI/CD:** O workflow faz checkout do repositorio `notroapp/server` (master) antes de instalar dependencias do crons, pois os testes dependem do banco do projeto server.

**Stages:** `dev`, `preview`, `prod`

## Comandos Uteis

```bash
# Testes
npm test                            # Suite completa (TZ=UTC, --runInBand)
npm run test-dev                    # Handler especifico (editar o path no script)
npm run test-debug                  # Debug com knex verbose
npm run pretest                     # Apenas setup de banco

# Lint
npm run lint                        # Verificar
npm run lint-fix                    # Corrigir automaticamente

# Deploy (via Serverless)
npx serverless deploy --stage prod
npx serverless deploy --stage preview

# Banco
npx knex seed:run --env notro       # Popular seeds de cron

# Commits
npm run commit                      # git-cz (commitizen)
```

## Arquivos-Chave

| Arquivo | Descricao |
|---|---|
| `serverless.yml` | Definicao de todas as 25 funcoes Lambda + schedules + API |
| `env.json` | Variaveis de ambiente por stage (dev/preview/prod) |
| `knexfile.js` | Configuracao Knex para 2 bancos PostgreSQL |
| `src/config.js` | Configuracao geral do app |
| `src/shared/sentry.js` | Error tracking condicional (prod only) |
| `src/shared/workers.js` | Facade SQS para filas worker e worker-tasks |
| `src/shared/sqs.js` | Cliente SQS de baixo nivel |
| `src/core/triggers/automatic.js` | Envia trigger messages para o worker |
| `src/core/sparkpost/sparkPostService.js` | Envio de emails via SparkPost |
| `src/core/communication/clients/twilio.js` | Cliente Twilio para WhatsApp |
| `.github/workflows/ci-cd.yml` | Pipeline CI/CD completo |

## Quando Voce Deve Ser Usado

- Para consultar qualquer aspecto do projeto `crons`
- Para saber qual handler gerencia cada tipo de tarefa agendada
- Para entender o schedule (frequencia) de cada cron
- Para saber como adicionar um novo handler/cron seguindo os padroes do projeto
- Para entender como as integracoes externas (Twilio, SparkPost, Easy API) funcionam
- Para mapear impacto de mudancas no banco nos crons existentes
- Para entender como testes sao escritos e como mockar dependencias
- Para saber como deployar e configurar novas funcoes no serverless.yml
