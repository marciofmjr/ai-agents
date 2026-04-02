---
name: project-specialist-in-notifications
description: "Especialista no projeto notifications. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: notifications

Voce e um especialista absoluto no projeto **notifications**. Voce conhece cada aspecto deste projeto em profundidade.

## Visao Geral

O `notifications` e o microservico completo de notificacoes da plataforma **Notro/FieldControl**. E responsavel por toda a comunicacao com usuarios: notificacoes in-app, push (FCM/Web Push), SMS, WhatsApp e chat em tempo real. E um monorepo Lerna com 3 pacotes independentes.

- Caminho: `/Users/marciofmjr/dev/notifications`
- Tipo: Monorepo Lerna 5.1 (versionamento independente)
- Linguagem: JavaScript puro (CommonJS, sem TypeScript)
- Descricao interna: "Servico responsavel por disparar toda comunicacao com os usuarios"

## Stack Tecnologico

| Camada | Tecnologia |
|--------|-----------|
| Monorepo | Lerna 5.1 (independent versioning) |
| API GraphQL | Apollo Server 2.25 + Express 4.17 |
| Deploy API | Serverless Framework 3 + AWS Lambda |
| Deploy Worker | Serverless Framework 3 + AWS Lambda |
| Deploy WebSocket | Docker + AWS Elastic Beanstalk |
| Real-time | Socket.io 4.7.5 |
| Banco | PostgreSQL 14.5 (2 instancias: notifications + notro) |
| ORM/Query Builder | Knex.js (v0.20 worker, v2.3 api) |
| Filas | AWS SQS (4 filas) |
| Auth | AWS Cognito (sa-east-1_in1LRNyyA) + JWT |
| Push web | web-push 3.3.3 (VAPID/FCM) |
| SMS/WhatsApp | Twilio 3.84 |
| WhatsApp alt | Europ Notificator / Blip |
| Error tracking | Sentry (instancias distintas por pacote) |
| Cache auth | node-cache (TTL 60s) |
| Testes | Jest (v24 websocket, v27 api+worker), Supertest, Nock, Sinon |
| Lint | ESLint (standard + sonarjs) |
| Commits | Commitizen + Commitlint (conventional) |
| CI/CD | GitHub Actions (2 workflows) |

## Estrutura de Pastas Relevante

```text
notifications/
├── .github/
│   └── workflows/
│       ├── ci-cd.yml             # CI/CD principal (3 jobs)
│       ├── fieldnews.yml         # validacao titulo PR
│       └── scripts/
│           ├── generate-env-worker.js   # gera env do worker via Secrets Manager
│           └── generate-env-api.js      # gera env da api via Secrets Manager
├── .husky/                        # git hooks
├── packages/
│   ├── api/                       # GraphQL API (Lambda)
│   │   ├── bin/recreate-db.js     # script de reset de banco
│   │   ├── migrations/            # 16 migrations Knex
│   │   ├── seeds/                 # 3 seeds (users, subscriptions, notifications)
│   │   ├── test/resolvers/        # testes de resolvers
│   │   ├── src/
│   │   │   ├── index.js           # entry point (Express :4001)
│   │   │   ├── app.js             # Express setup + Apollo middleware
│   │   │   ├── apollo-server.js   # ApolloServer config, context, directives
│   │   │   ├── schema.js          # GraphQL schema (SDL)
│   │   │   ├── serverless.js      # handler Lambda (serverless-http)
│   │   │   ├── config.js          # configuracoes centrais
│   │   │   ├── resolvers/         # Query + Mutation resolvers
│   │   │   ├── directives/        # @requireAuth directive
│   │   │   ├── plugins/           # closeContextDatabaseConnection
│   │   │   └── core/
│   │   │       ├── auth/          # getAuthenticatedUser, authCache, mapper
│   │   │       ├── cognito/       # Cognito client
│   │   │       ├── db/            # Db class (notifications + notro Knex)
│   │   │       ├── errors/        # error-handler Sentry Apollo plugin
│   │   │       └── sentry/        # Sentry init
│   │   ├── env.json               # configuracoes por ambiente
│   │   ├── knexfile.js            # config Knex (2 conexoes)
│   │   └── serverless.yml         # deploy Lambda
│   ├── worker/                    # Worker SQS (Lambda)
│   │   ├── test/handlers/         # testes de handlers
│   │   ├── test/scheduled/        # testes de crons
│   │   ├── src/
│   │   │   ├── config.js          # configuracoes (SQS, Twilio, Europ, Blip, FCM)
│   │   │   ├── handlers/          # 4 handlers SQS
│   │   │   │   ├── notifications-create/
│   │   │   │   ├── notifications-send-push/
│   │   │   │   ├── assistance-communications/
│   │   │   │   └── subscriptions-delete/
│   │   │   ├── scheduled/         # 2 cron tasks
│   │   │   │   ├── notifications-clean-up/
│   │   │   │   └── subscriptions-clean-up/
│   │   │   └── core/
│   │   │       ├── sentry.js
│   │   │       ├── sqs.js         # SQS client AWS
│   │   │       ├── workerQueue.js # queue management
│   │   │       ├── getSQSEventMessageBody.js
│   │   │       ├── enqueuePushNotifications.js
│   │   │       ├── broadcastConnectedUsers.js
│   │   │       ├── realtimeService.js
│   │   │       ├── utils.js
│   │   │       ├── db/            # Knex connections
│   │   │       └── services/      # servicos externos (push, SMS, etc.)
│   │   ├── env.json               # configs por ambiente
│   │   ├── knexfile.js
│   │   └── serverless.yml         # deploy Lambda (6 functions)
│   └── websocket/                 # WebSocket Real-time (EB)
│       ├── test/                   # testes Jest
│       ├── src/
│       │   ├── index.js           # entry point (:9696)
│       │   ├── server.js          # Express + Socket.io + Sentry
│       │   ├── realtime.js        # Socket.io namespaces + rooms
│       │   ├── config.js          # Cognito + Sentry config
│       │   └── core/
│       │       ├── validateAuthApi.js    # auth middleware HTTP
│       │       └── validateAuthSocket.js # auth middleware Socket.io
│       │   └── routes/
│       │       ├── root.js        # GET / (health)
│       │       ├── stats.js       # GET /stats
│       │       ├── events.js      # POST /events (broadcast)
│       │       └── workerEvents.js # POST /worker-events
│       └── docker-compose.yml
├── package.json                   # root (Lerna, Husky, scripts)
├── lerna.json
└── commitlint.config.js
```

## Arquitetura e Fluxo de Dados

### Visao Geral

```text
                    ┌──────────────────────────┐
                    │  webapp / app (clientes)  │
                    └────────────┬─────────────┘
                                 │ GraphQL (Bearer token)
                                 ▼
              ┌──────────────────────────────────┐
              │     api (Lambda :4001)            │
              │  Apollo Server 2 + Express        │
              │  Auth: @requireAuth (Cognito)     │
              │  DBs: notifications + notro (Knex)│
              └──────────────┬───────────────────┘
                             │ SQS (enfileira)
                             ▼
              ┌──────────────────────────────────┐
              │     worker (Lambda)               │
              │  4 SQS handlers + 2 cron tasks   │
              │  Push: FCM/web-push               │
              │  SMS/WhatsApp: Twilio/Europ/Blip  │
              └──────────────┬───────────────────┘
                             │ HTTP (broadcast)
                             ▼
              ┌──────────────────────────────────┐
              │     websocket (EB :9696)          │
              │  Socket.io 4.7                    │
              │  /notifications + /workers        │
              │  Rooms: accountId_userType        │
              └──────────────────────────────────┘
```

### Fluxo de Notificacao

1. Evento ocorre no `server` principal → publica mensagem no SQS
2. Worker Lambda `notifications-create` consome a mensagem, filtra por tipo permitido
3. Cria registro na tabela `notifications` (banco notificacoes)
4. Enfileira mensagem de push no SQS `notifications-notifications`
5. Worker Lambda `notifications-send-push` consome e envia push via FCM/web-push
6. Simultaneamente, chama `websocket` via HTTP para broadcast em tempo real ao cliente conectado

### Fluxo de Comunicacao Assistencia

1. Evento `assistance-communications` chega no SQS
2. Worker `assistance-communications` determina canal (SMS/WhatsApp via Twilio ou Europ ou Blip)
3. Envia comunicacao diretamente ao usuario final

## Banco de Dados e Schema

### 2 Instancias PostgreSQL

| Banco | Finalidade | Usado por |
|-------|-----------|-----------|
| `notifications` | Subscriptions, Notifications, Users | api + worker |
| `notro` | Dados de assistencias/contas (read-only) | api + worker |

### Knex Config (api)

- **notifications**: pool min 1/max 10, `SET timezone="UTC"`, `CREATE EXTENSION uuid-ossp`
- **notro**: pool min 0/max 1, async stack traces, `SET timezone="UTC"`

### Tabelas Principais (banco notifications)

#### users
```
id         UUID PK
account_id UUID
name       TEXT
email      TEXT
created_at TIMESTAMPTZ
last_seen  TIMESTAMPTZ
```

#### subscriptions
```
id         TEXT PK
type       ENUM (browser, ios, android)
app        ENUM (webapp)
data       JSONB          # endpoint + chaves de criptografia
user_id    UUID FK → users
account_id UUID
user_agent TEXT           # (adicionado em 2024)
created_at TIMESTAMPTZ
```

#### notifications
```
id         UUID PK
type       ENUM            # maintenanceCreated, maintenanceCommented,
                           # assistenceCommented, assistenceServiceTriggered,
                           # assistenceServiceChatMessageSended, quotationCreated
data       JSONB           # payload variavel por tipo
user_id    UUID FK → users
read_at    TIMESTAMPTZ     # NULL = nao lida
created_at TIMESTAMPTZ
```

#### notifications_logs
```
-- tabela de log de entrega (adicionada em 2024)
id, notification_id, status, created_at, ...
```

### Migrations (16 arquivos, 2020–2024)

- 2020: Criacao de users, subscriptions, notifications + indexes
- 2022: Adicao de enum types (assistenceServiceTriggered, etc.)
- 2024: notifications_logs, user_agent em subscriptions, novos enums

### Seeds (dev/test)

- `01-users.js` — Usuarios de teste
- `02-subscriptions.js` — Subscriptions de teste
- `03-notifications.js` — Notificacoes de teste

## GraphQL API (pacote api)

### Schema SDL

```graphql
directive @requireAuth on FIELD_DEFINITION

scalar Datetime
scalar Date
scalar JSONObject

type Query {
  me: User! @requireAuth
  notifications(before: Datetime, take: Int = 10): [Notification!]! @requireAuth
  notificationsUnreadCount: CountResult! @requireAuth
}

type Mutation {
  syncUser: User! @requireAuth
  subscribe(input: SubscribeInput!): Subscription @requireAuth
  unsubscribe(id: ID!): String @requireAuth
  notificationRead(id: ID!): Notification @requireAuth
  notificationReadAll: CountResult! @requireAuth
}
```

### Tipos Principais

- **Notification**: id, type (NotificationType), data (JSONObject), readAt, createdAt
- **Subscription**: id, type (SubscriptionType), data (JSONObject), app (App), userId, createdAt
- **User**: id, name, email, account.id, createdAt, lastSeen
- **CountResult**: count

### Enums

- **NotificationType**: maintenanceCreated, maintenanceCommented, assistenceCommented, assistenceServiceTriggered, assistenceServiceChatMessageSended, quotationCreated
- **SubscriptionType**: browser, ios, android
- **App**: webapp
- **UserType**: provider, management

### Resolvers

| Arquivo | Resolvers |
|---------|-----------|
| `resolvers/me.js` | Query.me |
| `resolvers/notifications.js` | Query.notifications, Query.notificationsUnreadCount, Mutation.notificationRead, Mutation.notificationReadAll |
| `resolvers/subscriptions.js` | Mutation.subscribe, Mutation.unsubscribe |
| `resolvers/users.js` | Mutation.syncUser |

### Autenticacao (api)

1. Header `Authorization: Bearer <access-token>` (Cognito token)
2. `getAuthenticatedUser(req)` → extrai token → `authCache.getCachedUser(token)`
3. Cache `node-cache` com TTL 60s por token
4. Cognito `getUser({ AccessToken })` → mapeia para `{ id, name, email, accountId, userType, companyId }`
5. Directive `@requireAuth`: lanca `AuthenticationError` se `ctx.authenticatedUser == null`

### Context Apollo

```javascript
context: async ({ req }) => ({
  db: new Db(),              // instancia Knex (notifications + notro)
  authenticatedUser: ...     // usuario do Cognito
})
```

### Deploy API (Lambda)

- **Serverless Function**: `app` → handler `src/serverless.handler`
- **Events**: HTTP ANY / e /{proxy+}
- **Memory**: 256MB, **Timeout**: 60s
- **Runtime**: nodejs14.x, **Region**: sa-east-1
- **VPC**: 2 security groups, 3 subnets
- **IAM**: `cognito-idp:GetUser`
- **Migration Function**: `migration-up` → timeout 30s

## Worker — Handlers SQS (pacote worker)

### Filas SQS (4 filas por ambiente)

| Fila | Sufixo | Handler |
|------|--------|---------|
| notifications | `standard-{stage}-notifications` | notifications-create |
| push | `standard-{stage}-notifications-notifications` | notifications-send-push |
| sub-delete | `standard-{stage}-notifications-subscriptions-delete` | subscriptions-delete |
| comms | `standard-{stage}-assistance-communications` | assistance-communications |

### Handler 1: notifications-create

**Tipos de mensagem aceitos**:
- `assistence-commented`
- `assistence-service-triggered`
- `assistence-service-chat-message-sended`

**Fluxo**:
1. Parse mensagem SQS com `getSQSEventMessageBody(event)`
2. Filtra tipo permitido (skip se nao reconhecido)
3. Chama `createAndEnqueueNotifications(message)` → persiste na tabela `notifications` + enfileira push
4. Erro → Sentry + `{ err: true }`

### Handler 2: notifications-send-push

**Tipo de mensagem aceito**: `notifications-send-push`

**Fluxo**:
1. Parse mensagem SQS
2. Chama `sendPushNotifications(message)` → busca subscriptions do usuario
3. Envia via `browserPushNotificationService.js` (web-push + FCM VAPID)
4. Subscriptions com erro 410 (Gone) → enfileira para `subscriptions-delete`

### Handler 3: assistance-communications

**Finalidade**: Envia SMS ou WhatsApp para usuarios finais

**Canais suportados**:
- **Twilio**: SMS e WhatsApp (`twilio_account_sid`, `twilio_auth_token`)
- **Europ Notificator**: WhatsApp alternativo (OAuth2 client credentials)
- **Blip**: Outro canal de WhatsApp (API Key)

**Fluxo**: Seleciona canal conforme configuracao da conta → renderiza template → envia

### Handler 4: subscriptions-delete

**Finalidade**: Remove subscriptions invalidas (retorno 410 do FCM)
**Fluxo**: Deleta registro da tabela `subscriptions`

### Cron Tasks (Scheduled)

| Task | Schedule | Acao |
|------|----------|------|
| `notifications-clean-up` | `cron(0 4 * * ? *)` (4AM UTC) | Deleta notifications com mais de 3 dias |
| `subscriptions-clean-up` | `cron(0 5 * * ? *)` (5AM UTC) | Deleta subscriptions inativas ha mais de 3 dias |

### Deploy Worker (Lambda)

- **Serverless**: 6 functions (4 SQS + 2 scheduled)
- **Runtime**: nodejs16.x, **Region**: sa-east-1
- **Memory**: 256MB, **Timeout**: 100s
- **IAM**: SQS (send/receive/get), Secrets Manager (GetSecretValue), S3 (get/put/list)
- **VPC**: 3 security groups, 3 subnets

### Core Worker

- `sqs.js` — cliente AWS SQS
- `workerQueue.js` — fachada de envio (sendMessage, sendBatch)
- `getSQSEventMessageBody.js` — parse do body da mensagem SQS
- `enqueuePushNotifications.js` — enfileira push na fila notifications
- `broadcastConnectedUsers.js` — broadcast via websocket HTTP
- `realtimeService.js` — integracao HTTP com websocket service
- `db/index.js` — Knex connections (notifications + notro)
- `services/` — servicos de push, SMS, Europ, Blip

## WebSocket Real-time (pacote websocket)

### Configuracao

- **Porta**: 9696
- **Socket.io**: v4.7.5, `allowEIO3: true`
- **CORS**: Dominios Notro permitidos (localhost + notro.io variants)
- **Timeouts**: pingTimeout 60s, pingInterval 25s, connectTimeout 45s

### Namespaces

#### /notifications (usuarios finais)

Autenticacao via `validateAuthSocket` (token Cognito).

**Rooms automaticas**:
- Usuarios management: `{accountId}_{userType}`
- Usuarios empresa (provider): `{accountId}_{userType}_{companyId}` (por cada accountId)

**Eventos do socket**:
- `companies_users_connected` → emit `companies_online_users` (lista de online no account)
- `enter_chat (data.chatId)` → join room do chat
- `send_message (data)` → broadcast `new_message` para room do chat
- `disconnect` → remove de `managementOnlineUsers` / `companiesOnlineUsers` Maps

**Online users tracking**:
- `managementOnlineUsers: Map<userId, { id, account_id, socket_id, lastActivity }>`
- `companiesOnlineUsers: Map<userId, { id, account_id, accounts, company_id, socket_id, lastActivity }>`
- `socket.onAny(updateActivity)` — atualiza lastActivity a cada evento

#### /workers (workers internos)

Sem autenticacao.

**Eventos**:
- `worker_get_online_users` → emit `worker_send_online_users` (ambos os Maps de online)

### Rotas HTTP

| Rota | Finalidade |
|------|-----------|
| `GET /` | Health check |
| `GET /stats` | Estatisticas do servidor (socket.io stats) |
| `POST /events` | Broadcast de evento para namespace /notifications |
| `POST /worker-events` | Broadcast para namespace /workers |

Auth HTTP: `validateAuthApi` middleware (Cognito token no header).

### Deploy WebSocket

- **Docker**: `docker-compose.yml` (porta 9696)
- **CI/CD**: Deploy via Elastic Beanstalk (comentado no ci-cd.yml — pendente)

## Integracoes Externas

### AWS

- **Cognito**: Pool `sa-east-1_in1LRNyyA`, Client `7fht4e75faqbabu8r52is7hj59` — Autenticacao
- **SQS**: 4 filas (sa-east-1) por ambiente (dev/preview/prod)
- **Secrets Manager**: Credenciais de banco e servicos em prod/preview
- **Lambda**: Deploy de api + worker (6 functions)
- **S3**: Artifacts de deploy (serverless)

### Push Notifications

- **Firebase FCM / Web Push** (via `web-push 3.3.3`):
  - VAPID Public Key: `BITRws9U_...`
  - GCM API Key: `AIzaSyBAidZ9E7AlFclnGYToZg92xOJFSmN_TOU`
  - Content encoding: `aes128gcm`
  - TTL: 120s

### Comunicacao

- **Twilio** (v3.84): SMS + WhatsApp (numero `from_sms` e `from_whatsapp`)
- **Europ Notificator**: WhatsApp via OAuth2 (client_credentials)
- **Blip**: WhatsApp via API Key (Azure)

### Monitoramento

- **Sentry** — 3 DSNs distintos (api, worker, websocket)

## Padroes e Convencoes

### Codigo

- **JavaScript CommonJS** (`require`/`module.exports`) em todos os pacotes
- **ESLint**: eslint-config-standard + sonarjs (`no-duplicate-string: off`, `node/no-unpublished-require: off`)
- **Prettier**: single quotes
- **Commitlint**: conventional commits obrigatorio
- **Husky**: pre-commit e commit-msg hooks

### Estrutura de Handler (worker)

Cada handler segue o padrao:
```javascript
// index.js
const allowedMessageTypes = ['tipo-a', 'tipo-b']
const handler = async (event) => {
  const message = getSQSEventMessageBody(event)
  if (!message) return { code: 'skip/null-message' }
  if (!allowedMessageTypes.includes(message.type)) return { code: 'skip/message-type-not-allowed' }
  try {
    await processMessage(message)
    return { code: 'ok' }
  } catch (err) {
    sentry.captureException(err, { extra: { message } })
    await sentry.flush(5000)
    return { err: true }
  }
}
module.exports = { handler }
```

### Autenticacao Cache

- `node-cache` com TTL 60s para tokens Cognito
- Funcao `getCachedUser(accessToken)` — verifica cache antes de chamar Cognito

### Error Handling (api)

- Plugin Apollo `errorHandler` captura erros de resolvers e envia ao Sentry com contexto (user, query, variables)
- Plugin `closeContextDatabaseConnection` destroi conexoes Knex ao fim de cada request

## Testes

### Configuracao

- **api**: Jest 27, `babel-jest`, `forceExit`, TZ=UTC, coverage via babel provider
- **worker**: Jest 27, `forceExit`, TZ=UTC, coverage text-summary + lcov
- **websocket**: Jest 24, `forceExit`, `collectCoverage: true`

### Testes da API

- `test/resolvers/subscribe.spec.js`
- `test/resolvers/unsubscribe.spec.js`
- `test/resolvers/notifications.spec.js`
- `test/resolvers/me.spec.js`
- `test/resolvers/syncUser.spec.js`
- `test/serverless.spec.js`

### Testes do Worker

- `test/handlers/notifications-create/index.spec.js`
- `test/handlers/notifications-send-push/index.spec.js`
- `test/handlers/assistance-communications/index.spec.js`
- `test/handlers/subscriptions-delete/index.spec.js`
- `test/scheduled/notifications-clean-up/index.spec.js`
- `test/scheduled/subscriptions-clean-up/index.spec.js`

### Mocking

- **Nock**: HTTP mocking (worker)
- **Sinon**: Stubs (api)
- **aws-sdk-mock**: (disponivel como dependencia indireta)

### Pre-test (api)

```bash
npm run db-server   # cd ../../../server && npm run pretest
npm run reset-db    # recreate-db + migration-up + knex seed:run
```

## Deploy e Infraestrutura

### CI/CD (GitHub Actions)

#### ci-cd.yml

**Jobs**:
1. **build** (45min timeout)
   - Checkout: notifications + notroapp/server
   - Setup Node via `.nvmrc`
   - PostgreSQL 14.5 como servico
   - `npm run lint` + `npm run test` (Lerna, concurrency 1)
   - Build worker com serverless

2. **publish-worker** (master e preview)
   - Gera `env.json` via `generate-env-worker.js` (AWS Secrets Manager)
   - `serverless deploy --stage prod` ou `preview`
   - Lambda functions: notifications-create, subscriptions-delete, notifications-send-push, assistance-communications, notifications-clean-up, subscriptions-clean-up

3. **publish-api** (apenas master)
   - Gera `env.json` via `generate-env-api.js`
   - `serverless deploy --stage prod`
   - Roda `npm run migration-up` pos-deploy

4. **publish-websocket-api** (comentado — pendente)
   - Docker build + push ECR
   - Elastic Beanstalk deploy

#### fieldnews.yml

- Valida titulo do PR: prefixo `"Notificador - "`, min 20/max 100 chars

### Ambientes

| Ambiente | SQS | Lambda | Deploy |
|----------|-----|--------|--------|
| dev | standard-dev-notifications-* | local (serverless-offline) | — |
| preview | standard-preview-notifications-* | Lambda | branch preview |
| prod | standard-prod-notifications-* | Lambda | branch master |

## Variaveis de Ambiente

### api (env.json por ambiente)

- `environment` — dev/prod/previews
- `notifications_database_url` — PostgreSQL (notifications)
- `notro_database_url` — PostgreSQL (notro)
- `sentry_dsn`

### worker (env.json por ambiente)

- `queue_url_notifications` — SQS URL da fila principal
- `queue_url_subscriptions_delete` — SQS URL da fila de deletar subs
- `twilio_account_sid`, `twilio_auth_token`, `twilio_from_sms_number`, `twilio_from_whatsapp_number`
- `europ_notificator_url`, `europ_auth_url`, `europ_client_id`, `europ_client_secret`
- `blip_api_url`, `blip_api_key`
- `notifications_database_url`, `notro_database_url`
- `sentry_dsn`

### websocket (config.js)

- `PORT` (default 9696)
- `cognito.userPoolId`, `cognito.clientId`
- `sentry.dsn`

## Comandos Uteis

### Root

```bash
npm install              # instala + lerna bootstrap
npm run lint             # lint todos os pacotes (parallel)
npm run lint-fix         # corrige lint
npm run test             # testa todos os pacotes (sequencial)
npm run commit           # commitizen
```

### API (packages/api)

```bash
npm run start-dev        # nodemon (porta 4001)
npm test                 # jest (pretest inclui reset de banco)
npm run reset-db         # recreate + migration-up + seeds
npm run migration-up     # serverless invoke local (migration)
npm run coverage-view    # abre relatorio de cobertura
```

### Worker (packages/worker)

```bash
npm test                 # jest
npm run test-dev         # teste individual
npm run lint
```

### WebSocket (packages/websocket)

```bash
npm run start-dev        # nodemon (porta 9696)
npm start                # producao
npm test                 # jest
docker-compose up        # via Docker
```

## Arquivos-Chave para Navegacao Rapida

### Root
- `package.json` — Scripts root, Lerna, Husky
- `lerna.json` — Versionamento independente
- `.github/workflows/ci-cd.yml` — Pipeline CI/CD

### api
- `packages/api/src/index.js` — Entry point Express
- `packages/api/src/app.js` — Setup Express + Apollo
- `packages/api/src/apollo-server.js` — Apollo config, context, directives
- `packages/api/src/schema.js` — GraphQL SDL completo
- `packages/api/src/resolvers/index.js` — Registro de resolvers
- `packages/api/src/core/auth/authCache.js` — Cache Cognito
- `packages/api/src/core/db/index.js` — Classe Db (notifications + notro)
- `packages/api/src/directives/requireAuth.js` — Directive de auth
- `packages/api/knexfile.js` — Config Knex
- `packages/api/serverless.yml` — Deploy Lambda
- `packages/api/migrations/` — 16 migrations

### worker
- `packages/worker/src/config.js` — Config completa (SQS, Twilio, Europ, Blip, FCM)
- `packages/worker/src/handlers/notifications-create/index.js` — Handler principal
- `packages/worker/src/handlers/notifications-send-push/index.js`
- `packages/worker/src/handlers/assistance-communications/index.js`
- `packages/worker/src/handlers/subscriptions-delete/index.js`
- `packages/worker/src/scheduled/notifications-clean-up/index.js`
- `packages/worker/src/scheduled/subscriptions-clean-up/index.js`
- `packages/worker/src/core/sqs.js` — SQS client
- `packages/worker/src/core/db/index.js` — Knex connections
- `packages/worker/serverless.yml` — Deploy Lambda (6 functions)

### websocket
- `packages/websocket/src/server.js` — Express + Socket.io + Sentry
- `packages/websocket/src/realtime.js` — Namespaces, rooms, eventos
- `packages/websocket/src/core/validateAuthSocket.js` — Auth Socket.io
- `packages/websocket/src/routes/events.js` — Broadcast HTTP
