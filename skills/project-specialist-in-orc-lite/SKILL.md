---
name: project-specialist-in-orc-lite
description: "Especialista no projeto orc-lite. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: orc-lite

Voce e um especialista absoluto no projeto **orc-lite**. Voce conhece cada aspecto deste projeto em profundidade.

## Visao Geral

O `orc-lite` e a plataforma **Easy** — um sistema completo de gestao de ordens de servico, tarefas de campo, integracoes com parceiros e comunicacao em tempo real. E um monorepo Lerna com 12 pacotes cobrindo backend (APIs GraphQL e REST), frontend (Angular web e Ionic mobile), workers de processamento assincrono, notificacoes push, geolocalizacao, autenticacao e crons.

- Caminho: `/Users/marciofmjr/dev/orc-lite`
- Tipo: Monorepo Lerna (versionamento independente)
- Nome interno: `easy`
- Runtime: Node.js (v18-v22 conforme pacote)

## Stack Tecnologico

| Camada | Tecnologia |
|--------|-----------|
| Monorepo | Lerna 4 (independent versioning) |
| Backend principal | NestJS 11 + Apollo Server 4 (GraphQL) |
| API publica REST | NestJS 10 + Swagger |
| Geolocalizacao API | Fastify 5 |
| Notificacoes API | Apollo Server 2 + Express |
| Autenticacao | Serverless Framework + AWS Lambda + Cognito |
| Workers | SQS consumers (notifications, exports) |
| WebSocket | Express + Socket.io 4.7 |
| Crons | Serverless + EventBridge |
| Frontend web | Angular 19 + ng-zorro-antd 19 + Tailwind 4 + Apollo Angular 8 |
| Frontend mobile | Ionic 8 + Angular 19 + Capacitor 6 |
| Banco | PostgreSQL 14.5 (3 instancias: easy, notifications, geolocations) |
| ORM/Query Builder | Knex.js (v1-v3 conforme pacote) |
| Autenticacao | AWS Cognito + JWT + MFA (TOTP) |
| Storage | AWS S3 |
| Filas | AWS SQS |
| Push | Firebase FCM + Apple APN + Web Push (VAPID) |
| Email | SparkPost |
| Maps | Leaflet + Node-geocoder |
| Charts | ApexCharts 4.7 |
| i18n | @ngx-translate |
| Error tracking | Sentry |
| Testes | Jest (v24-v29 conforme pacote), ts-jest, Supertest, Nock |
| Lint | ESLint 9 + TypeScript ESLint |
| Commits | Commitizen + Commitlint (conventional) |
| CI/CD | GitHub Actions (11 workflows) + Elastic Beanstalk + S3/CloudFront |
| Dev local | Docker Compose (PostgreSQL x3, LocalStack, Cognito Local, DynamoDB Admin) |
| Linguagem | TypeScript 5.8 (backend/frontend), JavaScript (alguns pacotes legados) |

## Estrutura de Pastas Relevante

```text
orc-lite/
├── .github/
│   ├── pull_request_template.md
│   └── workflows/
│       ├── 1-main.yml                   # orquestrador (chama os demais)
│       ├── 2-api-ci-cd.yml              # server
│       ├── 3-authentications-ci-cd.yml
│       ├── 4-integrations-ci-cd.yml
│       ├── 5-webapp-ci-cd.yml
│       ├── 6-geolocations-ci-cd.yml
│       ├── 7-notifications-ci-cd.yml
│       ├── 8-websocket-ci-cd.yml
│       ├── 9-crons-ci-cd.yml
│       ├── 10-exports-ci-cd.yml
│       └── notro-patterns.yml
├── packages/
│   ├── server/                          # NestJS GraphQL API principal
│   ├── webapp/                          # Angular 19 SPA
│   ├── app/                             # Ionic 8 mobile
│   ├── integrations-api/                # NestJS REST API publica
│   ├── authentications-api/             # Serverless Lambda auth
│   ├── authentications-triggers/        # Lambda triggers Cognito
│   ├── geolocations-api/               # Fastify REST API
│   ├── notifications-api/              # Apollo Server GraphQL
│   ├── notifications-worker/           # Worker SQS push
│   ├── websocket/                       # Express + Socket.io
│   ├── crons/                           # Scheduled tasks
│   └── exports/                         # Worker SQS export
├── skills/                              # Copilot/AI skills
├── docker-compose.yml
├── configure-containers.sh
├── configure-localtunnel.sh
├── configure-local-stack.sh
├── deploy.md
├── package.json                         # root (Lerna, ESLint, Husky)
├── lerna.json
├── tsconfig.json
├── eslint.config.mjs
└── commitlint.config.js
```

## Arquitetura e Fluxo de Dados

### Visao Geral da Arquitetura

```text
                    ┌─────────────┐     ┌──────────────┐
                    │   webapp     │     │     app       │
                    │ (Angular 19) │     │ (Ionic 8)     │
                    └──────┬──────┘     └──────┬────────┘
                           │ GraphQL            │ GraphQL
                           ▼                    ▼
              ┌────────────────────────────────────┐
              │           server (NestJS 11)         │
              │        GraphQL API :4012              │
              │  32+ modules, AuthGuard, MfaGuard     │
              └──────────┬────────────┬───────────┘
                         │            │
            ┌────────────┘            └──────────────┐
            ▼                                        ▼
  ┌──────────────────┐                    ┌──────────────────┐
  │  PostgreSQL (easy) │                    │    AWS SQS        │
  │    via Knex        │                    │  (async jobs)     │
  └──────────────────┘                    └────────┬─────────┘
                                                   │
                         ┌─────────────────────────┼──────────┐
                         ▼                         ▼          ▼
              ┌──────────────────┐    ┌───────────────┐  ┌────────┐
              │ notifications-   │    │    exports      │  │  ...   │
              │ worker (push)    │    │ (Excel/CSV)    │  │        │
              └──────────────────┘    └───────────────┘  └────────┘

  ┌────────────────────────┐    ┌─────────────────────┐
  │ integrations-api :3005  │    │ geolocations-api    │
  │ REST (x-api-key)        │    │ Fastify :8080        │
  └────────────────────────┘    └─────────────────────┘

  ┌────────────────────────┐    ┌─────────────────────┐
  │ notifications-api :4001 │    │ websocket (Socket.io)│
  │ GraphQL subscriptions   │    │ real-time events     │
  └────────────────────────┘    └─────────────────────┘

  ┌────────────────────────┐    ┌─────────────────────┐
  │ authentications-api     │    │ authentications-     │
  │ Lambda + Cognito + MFA  │    │ triggers (Lambda)    │
  └────────────────────────┘    └─────────────────────┘

  ┌────────────────────────┐
  │ crons (EventBridge)     │
  │ 7 scheduled tasks       │
  └────────────────────────┘
```

### Pacote: server (API GraphQL Principal)

- **Framework**: NestJS 11 + Apollo Server 4
- **Porta**: 4012
- **Entry**: `packages/server/src/main.ts`
- **App Module**: `packages/server/src/app.module.ts` — 32+ feature modules
- **Guards globais**: `AuthGuard` (JWT via Cognito) + `MfaGuard` (TOTP)
- **Guard de permissao**: `PermissionGuard` (decorator `@Permissions()`, cache 1h)
- **Database**: `DatabaseModule` com Knex (KnexConnection + KnexConnectionReader para read replicas)
- **Worker SQS**: `src/core/worker/sqs.ts` + `src/core/aws/sqs-facade/`

**Feature Modules (32+)**:
- `users`, `me`, `companies`, `employees`
- `orders`, `order-rental-car`, `order-chats`, `orders-attachments`, `orders-prices`, `orders-signatures`, `orders-prices-logs`, `orders-researches`
- `services`, `services-requests`
- `tasks`, `tasks-comments`, `tasks-logs`
- `customers`
- `forms`, `forms-answers`
- `approvals`, `approvals-orders-prices`, `approvals-orders-prices-attachments`, `approvals-logs`
- `payments-lots`, `payments-lots-logs`
- `prices-rates-types`
- `credentials`, `dashboards`
- `groups`, `permissions`
- `address`

**Padrao por modulo**:
```text
src/modules/<nome>/
├── <nome>.module.ts
├── resolvers/
│   ├── <nome>-operations.resolver.ts    # Queries + Mutations
│   └── <nome>-fields.resolver.ts        # Field resolvers (relacionamentos)
├── services/
│   ├── <nome>.query.service.ts          # Leitura
│   ├── <nome>.mutation.service.ts       # Escrita
│   └── <nome>.*-fy.service.ts           # Filtros/ordenacao
├── entities/
│   └── <nome>.entity.ts                 # GraphQL ObjectType
└── dto/
    ├── <nome>.where.input.ts            # Filtros
    └── <nome>-order-by.inputs.ts        # Ordenacao
```

### Pacote: integrations-api (REST Publica)

- **Framework**: NestJS 10 + Express
- **Porta**: 3005
- **Auth**: `x-api-key` header → `AuthMiddleware`
- **Swagger**: `/docs` (basic auth: notro:notro@docs)
- **Throttle**: 500 req/60s
- **Entry**: `packages/integrations-api/src/main.ts`

**Resources REST**:
- orders (+ prices, chats, forms-answers, locations, tasks)
- customers, services, employees, vehicles
- tasks (+ comments)
- forms, services-forms
- approvals, approvals-orders-prices
- payments-lots
- service-request, price-rate-types
- users, accounts, api-keys, root

**Padrao por resource**:
```text
src/resources/<nome>/
├── <nome>.controller.ts     # Rotas HTTP
├── <nome>.service.ts        # Logica de negocio
├── <nome>.types.ts          # Interfaces TS
├── <nome>.mapper.ts         # Mapeamento DTO
└── <nome>.schema.ts         # Validacao
```

### Pacote: geolocations-api

- **Framework**: Fastify 5
- **Porta**: 8080
- **Auth**: Middleware validando token via Cognito
- **Resources**: locations (employees, vehicles)

### Pacote: notifications-api

- **Framework**: Apollo Server 2 + Express
- **Porta**: 4001
- **GraphQL**: Queries, mutations e subscriptions
- **Auth**: Directive `requireAuth`
- **Databases**: notificationsDb + easyDb (2 conexoes Knex)

### Pacote: notifications-worker

- **Tipo**: Worker SQS
- **Push**: Firebase FCM, Apple APN, Web Push (VAPID)
- **Handlers de criacao**: taskToEmployeeNotifications, serviceRequestCreated, orderCanceledNotification, chatMessageCreated, checkinTaskByGPSProximity, checkinTaskByAppProximity
- **Scheduled**: subscriptions-clean-up, notifications-clean-up

### Pacote: websocket

- **Framework**: Express + Socket.io 4.7
- **Rotas**: `/` (health), `/stats`, `/events` (real-time), `/worker-events`
- **Auth**: Valida token via API

### Pacote: authentications-api

- **Tipo**: Serverless Framework (AWS Lambda)
- **Auth**: AWS Cognito + DynamoDB
- **Endpoints**: create-user, sign-in-with-recovery-code, admin-change-password, regenerate-token
- **MFA**: generate, verify, disable (TOTP)
- **Email**: SparkPost

### Pacote: authentications-triggers

- **Tipo**: AWS Lambda triggers para Cognito
- **Handlers**: CustomMessage_AdminCreateUser, CustomMessage_ForgotPassword
- **Email**: SparkPost templates

### Pacote: crons

- **Tipo**: Serverless + EventBridge scheduled
- **7 handlers**:
  1. `change-scheduled-task-to-waiting` — Converte tarefas scheduled → waiting
  2. `delete-employee-geolocation` — Limpa geolocalizacoes antigas
  3. `sync-vehicles-geolocations-by-positron` — Sync GPS Positron
  4. `sync-vehicles-geolocations-by-ceabs` — Sync GPS CEABS
  5. `expire-services-requests` — Expira solicitacoes de servico
  6. `check-rent-car-deadline` — Verifica prazos de devolucao
  7. `check-rent-car-return` — Verifica devolucoes de carro alugado

### Pacote: exports

- **Tipo**: Worker SQS
- **Handlers**: data-export-orders, data-export-services-requests, data-export-orders-researches, data-export-vehicles
- **Output**: Excel (<=50k rows) ou CSV (>50k rows)
- **Email**: SparkPost para enviar link de download

### Pacote: webapp

- **Framework**: Angular 19
- **UI**: ng-zorro-antd 19 (Ant Design) + Angular Material 19 + Tailwind 4
- **GraphQL**: Apollo Angular 8
- **Maps**: Leaflet 1.9 + marker clustering
- **Charts**: ApexCharts 4.7
- **i18n**: @ngx-translate
- **PWA**: Service Worker + Sentry
- **Real-time**: Socket.io Client 4.8

### Pacote: app

- **Framework**: Ionic 8 + Angular 19 + Capacitor 6
- **Plataformas**: iOS + Android
- **Features nativas**: Background Geolocation, Barcode Scanning (MLKit), File Picker, Video Editor, Signature Pad
- **OTA**: LiveUpdate
- **Real-time**: Socket.io Client

## Entidades e Banco de Dados

### 3 Instancias PostgreSQL

| Instancia | Banco | Porta (dev) | Pacotes que usam |
|-----------|-------|------------|-----------------|
| Principal | easy | 5432 | server, integrations-api, authentications-api, crons, exports |
| Notificacoes | notifications | 5433 | notifications-api, notifications-worker |
| Geolocalizacao | geolocations | 5434 | geolocations-api |

### Entidades Principais (banco easy)

- **orders** — Ordens de servico (com `custom_data` JSON)
- **orders_attachments** — Anexos de ordens
- **orders_chats** — Chat de ordens
- **orders_prices** — Precos de ordens
- **orders_prices_logs** — Historico de precos
- **orders_researches** — Pesquisas de ordens
- **orders_signatures** — Assinaturas digitais
- **tasks** — Tarefas de campo
- **tasks_comments** — Comentarios de tarefas
- **tasks_logs** — Logs de tarefas
- **services** — Definicoes de servico
- **services_requests** — Solicitacoes de servico
- **customers** — Clientes
- **employees** — Funcionarios
- **users** — Usuarios do sistema
- **users_groups** — Associacao usuario-grupo
- **groups** — Grupos de permissao
- **groups_permissions** — Permissoes por grupo
- **permissions** — Definicoes de permissao
- **companies** — Empresas
- **vehicles** — Veiculos
- **forms** — Formularios
- **forms_answers** — Respostas de formularios
- **approvals** — Aprovacoes
- **approvals_orders_prices** — Aprovacoes de precos
- **approvals_orders_prices_attachments** — Anexos de aprovacoes
- **approvals_logs** — Logs de aprovacoes
- **payments_lots** — Lotes de pagamento
- **payments_lots_logs** — Logs de lotes
- **prices_rates_types** — Tipos de taxa/preco
- **credentials** — Credenciais de integracao
- **dashboards** — Configuracoes de dashboard
- **address** — Enderecos

### Enums Principais

- **OrderStatus**: opened, closed
- **TaskStatus**: waiting, scheduled, onRoute, inProgress, done, canceled, reported, organized, inOrganization, rentalCarTriggered, awaitingReturn, awaitingExtension, pendingAddress
- **TaskCompletedByAction**: app, controlPanel, managementPanel
- **UserRole**: default, admin

### Relacionamentos

- Orders → Services, Customers, Tasks, Prices, Payments, Approvals
- Tasks → Employees, Orders, Comments, Logs
- Approvals → Orders, Prices, Attachments
- Users → Users_Groups → Groups → Groups_Permissions → Permissions

## Fluxo de Autenticacao e Autorizacao

1. **Login**: User → Cognito (username/password) → JWT token
2. **Request**: JWT no header `Authorization: Bearer <token>` ou `x-access-token`
3. **AuthGuard**: Valida token, chama `getCachedUser()`, retorna `{ userId, accountId, type }`
4. **MfaGuard**: Verifica `users.mfa_active` e config da conta. Decorator `@BypassMfa()` para exceções
5. **PermissionGuard**: Decorator `@Permissions('nome')`, cache por token (1h TTL). Fluxo: users → users_groups → groups_permissions → permissions
6. **@Public()**: Decorator para rotas sem auth

## Integracoes Externas

### AWS

- **Cognito**: Autenticacao, MFA, user management
- **S3**: Storage de arquivos/anexos
- **SQS**: Filas de mensagens (notifications, exports, worker tasks)
- **SNS**: Notificacoes
- **Lambda**: Serverless functions (auth, triggers, crons)
- **DynamoDB**: Auth API local dev
- **Secrets Manager**: Credenciais de banco em prod/preview
- **Elastic Beanstalk**: Deploy de APIs
- **CloudFront + S3**: Deploy de webapp
- **EventBridge**: Scheduling de crons

### Comunicacao

- **SparkPost**: Emails transacionais (auth, exports, notificacoes)
- **Firebase FCM**: Push notifications Android
- **Apple APN**: Push notifications iOS
- **Web Push (VAPID)**: Push notifications web
- **Socket.io**: Eventos real-time

### Geolocalizacao

- **Positron API**: Sync GPS de veiculos
- **CEABS API**: Sync GPS de veiculos
- **Node-geocoder**: Geocoding no server

### Monitoramento

- **Sentry**: Error tracking em todos os pacotes

## Padroes e Convencoes

### Codigo

- **TypeScript 5.8** para a maioria dos pacotes
- **JavaScript** em pacotes legados (websocket, authentications-api, notifications-api)
- **ESLint 9** com TypeScript ESLint: `no-explicit-any: error`, `semi: always`, `quotes: single`, `prefer-optional-chain`
- **Commitlint**: conventional commits obrigatorio
- **Husky 4**: pre-commit (lint + test), commit-msg (commitlint)

### NestJS (server + integrations-api)

- Modulos por dominio em `src/modules/` (server) ou `src/resources/` (integrations-api)
- `@Injectable()` services com injecao de dependencia
- `ValidationPipe` global com `transform: true`
- Errors: `UnprocessableEntityException`, `UnauthorizedException`, `NotFoundException`, `BadRequestException`
- `CustomExceptionFilter` com eventId + timestamp

### GraphQL (server)

- Schema auto-gerado em `schema.gql`
- Resolvers separados: operations (queries/mutations) + fields (relacionamentos)
- DataLoader para batch loading
- Input types com `Where` (filtros) e `OrderBy` (ordenacao)

### REST (integrations-api)

- Controllers com decorators Swagger
- Pipes: `coerceNullQueryStringPipe`, `trimPipe`, `validationPipe`
- Mappers para conversao entidade → DTO
- Schemas para validacao

### Banco

- Knex.js em todos os pacotes backend
- Transactions para consistencia (`database.transaction(async (trx) => { ... })`)
- Read replicas no server (`KnexConnection` + `KnexConnectionReader`)
- Migrations por pacote, seeds para dev/test

### Workers/Filas

- SQS facade com `sendMessage()` e `sendMessageBatch()` (chunks de 10)
- Handlers recebem mensagem com `{ type, ... }` e roteiam por tipo
- Dead letter queue para falhas

## Testes

### Configuracao

- **Jest** em todos os pacotes (versoes variadas: 24-29)
- **ts-jest** para TypeScript
- **Supertest** para testes HTTP
- **Nock** para mocking HTTP externo
- **aws-sdk-mock** para AWS

### Padroes de Teste

- **server**: `Test.createTestingModule()` com mock providers, DataLoader mocking
- **integrations-api**: Unit + E2E (`test/resources/*.e2e.spec.ts`)
- **geolocations-api**: Coverage threshold 100% (branches, functions, lines, statements)
- **Mock de auth**: `jest.spyOn(AuthJWT, 'getUserByIdToken')`
- **Mock de banco**: Knex connections mockadas nos providers

### Comandos

```bash
# Root
npm test                     # lerna run test (sequencial)

# Por pacote
cd packages/server && npm test
cd packages/integrations-api && npm test
cd packages/geolocations-api && npm test
```

## Deploy e Infraestrutura

### CI/CD (GitHub Actions)

**Orquestrador**: `.github/workflows/1-main.yml`
- Triggers: push master/preview, PRs
- Chama 10 workflows filhos em paralelo

**Padrao de cada workflow**:
1. `files-changed`: Detecta mudancas no pacote (dorny/paths-filter)
2. `build`: Install deps, lint, test (PostgreSQL service container)
3. `publish-prod` (master): Build + zip + deploy Elastic Beanstalk
4. `publish-preview` (preview): Deploy para ambiente preview
5. Force deploy: mensagem de commit com `[force-api]`

### Ambientes

| Pacote | Prod | Preview |
|--------|------|---------|
| server | easy-api-prod (EB) | easy-api-preview (EB) |
| integrations-api | EB | EB |
| webapp | S3 + CloudFront | S3 + CloudFront |
| geolocations-api | EB | EB |
| websocket | EB | EB |

### Docker (dev local)

```yaml
# docker-compose.yml
postgres (easy):        porta 5432
postgres (notifications): porta 5433
postgres (geolocations): porta 5434
localstack:             porta 4566 (S3, DynamoDB, SQS, SNS, Lambda, API Gateway, Cognito, IAM)
cognito-local:          porta 9229
dynamodb-admin:         porta 8000
```

Setup: `npm run docker:configure-stack`

## Variaveis de Ambiente

### server

- `PORT` (default 4012)
- `NODE_ENV` (dev/preview/production)
- `DB_SECRET_ID` (Secrets Manager)
- `JWT_SECRET`
- `AWS_REGION` (sa-east-1)
- `SENTRY_DSN`
- `S3_BUCKET`
- `SQS_QUEUE_URL`

### integrations-api

- `PORT` (default 3005)
- `DB_SECRET_ID`
- Mesmas AWS vars

### Node Versions (.nvmrc)

| Pacote | Node |
|--------|------|
| server | v22 |
| webapp | v18 |
| app | v20 |
| integrations-api | v18 |
| geolocations-api | v18 |
| authentications-api | v18 |
| authentications-triggers | v18 |
| notifications-api | v18 |
| notifications-worker | v18 |
| websocket | v18 |
| crons | v18 |
| exports | v18 |

## Comandos Uteis

### Root

```bash
npm install                          # instala deps root
npm run lint                         # ESLint em todos os pacotes
npm run lint:fix                     # corrige lint
npm test                             # testa todos os pacotes (sequencial)
npm start                            # lerna run start-dev
npm run docker:configure-stack       # sobe Docker + configura AWS local
npm run commit                       # commitizen
```

### Por Pacote

```bash
cd packages/server
npm install && npm run start:dev     # dev mode
npm test                             # testes
npm run build                        # build producao

cd packages/webapp
npm install && npm start             # dev mode Angular

cd packages/app
npm install && ionic serve           # dev mode Ionic

cd packages/integrations-api
npm install && npm run start:dev
npm test

cd packages/geolocations-api
npm install && npm run start:dev
npm test
```

## Arquivos-Chave para Navegacao Rapida

### Root
- `package.json` — Config monorepo, scripts, deps
- `lerna.json` — Versionamento independente
- `docker-compose.yml` — Stack local completa
- `eslint.config.mjs` — Regras ESLint
- `.github/workflows/1-main.yml` — Orquestrador CI/CD

### server
- `packages/server/src/main.ts` — Bootstrap NestJS
- `packages/server/src/app.module.ts` — Root module (32+ imports)
- `packages/server/src/guards/auth/auth.guard.ts` — Auth JWT
- `packages/server/src/guards/mfa/mfa.guard.ts` — MFA enforcement
- `packages/server/src/guards/permission/permission.guard.ts` — RBAC
- `packages/server/src/core/database.module.ts` — Knex connections
- `packages/server/src/core/worker/sqs.ts` — SQS config
- `packages/server/knexfile.ts` — DB config
- `packages/server/src/modules/` — 32+ feature modules

### integrations-api
- `packages/integrations-api/src/main.ts` — Bootstrap
- `packages/integrations-api/src/bootstrap.ts` — Swagger + guards
- `packages/integrations-api/src/app.module.ts` — Resources
- `packages/integrations-api/src/middlewares/auth.middleware.ts` — x-api-key
- `packages/integrations-api/src/resources/` — 15+ REST resources

### geolocations-api
- `packages/geolocations-api/src/app.ts` — Fastify setup
- `packages/geolocations-api/src/routes.ts` — Route registration

### notifications
- `packages/notifications-api/src/index.js` — Apollo Server setup
- `packages/notifications-worker/src/core/workerQueue.js` — SQS polling
- `packages/notifications-worker/src/handlers/` — Push handlers

### auth
- `packages/authentications-api/src/main/auth/` — Auth endpoints
- `packages/authentications-api/src/main/mfa-settings/` — MFA endpoints
- `packages/authentications-api/src/shared/cognitoFacade.js` — Cognito wrapper

### websocket
- `packages/websocket/src/server.js` — Socket.io setup
- `packages/websocket/src/realtime.js` — Event handlers

### crons
- `packages/crons/src/handlers/` — 7 cron handlers

### exports
- `packages/exports/src/handlers/` — 4 export handlers
