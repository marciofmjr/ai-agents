---
name: project-specialist-in-public-api
description: "Especialista no projeto public-api. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: public-api

Voce e um especialista no projeto **public-api**.

## Visao Geral

O `public-api` e uma API REST publica da Notro Assistance para integracoes B2B com sistemas externos.

- Caminho: `/Users/marciofmjr/dev/public-api`
- Tipo: monorepo Lerna com foco principal no pacote `packages/api`
- Runtime principal: NestJS (Express) + PostgreSQL (via Knex)

## Stack Tecnologico

| Camada | Tecnologia |
|--------|-----------|
| Monorepo | Lerna 6 (root) |
| Backend | NestJS 10 + Express |
| Linguagem | TypeScript 4.8 |
| Banco | PostgreSQL |
| Acesso a dados | Knex + Query Builder interno |
| Documentacao API | Swagger (`/docs`, `/docs-json`) |
| Seguranca | Auth por `x-api-key`, guards de area/tipo/conta, Helmet, Throttler |
| Filas/assinc | AWS SQS |
| Storage | AWS S3 |
| Secrets | AWS Secrets Manager |
| Observabilidade | Sentry |
| Integracoes externas | Easy, Europ, Grow, Hero, Google Maps, HERE |
| Testes | Jest (unit + e2e com Supertest) |
| CI/CD | GitHub Actions + Elastic Beanstalk |
| Node | `>=18` |

## Estrutura de Pastas Relevante

```text
public-api/
├── .github/workflows/
│   ├── ci-cd.yml
│   └── scripts/generate-env.js
├── docs/
│   └── environment-variables.md
├── packages/
│   └── api/
│       ├── bin/
│       ├── seeds/
│       ├── src/
│       │   ├── app.module.ts
│       │   ├── bootstrap.ts
│       │   ├── main.ts
│       │   ├── core/                 # db, worker, integrations, aws, sentry
│       │   ├── guards/
│       │   ├── middlewares/
│       │   ├── pipes/
│       │   ├── resources/            # dominios de API
│       │   ├── shared/               # servicos compartilhados (geoloc, precos)
│       │   ├── types/
│       │   └── utils/
│       ├── test/                     # e2e + unit extras
│       ├── env.json
│       ├── jest.config.ts
│       └── package.json
├── package.json
└── lerna.json
```

## Arquitetura e Fluxo de Dados

### Arquitetura

- Monolito modular Nest por dominio (`src/resources/*`).
- Padrao recorrente por dominio: `controller -> service -> mapper -> database`.
- Acesso a dados com Knex (`database('tabela')`) e paginacao/filtros com `executeFindPageQuery`.
- Modulos de assistencias sao mais ricos e orquestram regras de negocio, transacoes, logs, workers e integracoes.

### Bootstrap e inicializacao

1. `main.ts` chama `bootstrap()`.
2. `bootstrap.ts` cria app Nest, aplica `helmet`, CORS, shutdown hooks e versionamento URI (`VERSION_NEUTRAL`).
3. Swagger e exposto em `/docs` e `/docs-json` com basic auth.
4. `AppModule` agrega todos os modulos e aplica `ThrottlerGuard` global (`ttl: 60000`, `limit: 500`).

### Seguranca e acesso

- Middleware principal: `AuthMiddleware` por modulo (nao global).
- Guards por politica: `ActivityAreaGuard`, `ApiKeyTypeGuard`, `AccountsRestrictionGuard`.
- Decorators de regra: `@RequiredAreas`, `@RequiredApiKeyType`, `@AccountsAllowed`.

## Entidades e Banco de Dados

### Camada de banco

- Singleton de conexao: `src/core/db/database.ts`
- Config de conexao por ambiente: `src/core/db/get-connection-config.ts`
- Config Knex e seeds: `src/knexfile.ts`
- Query builder interno: `src/core/db/query-builder.ts`

### Modelagem funcional (alto nivel)

A API opera sobre dominios como assistencias, servicos, comunicacoes, precos, reembolsos, apolices, empresas, prestadores, clientes, localizacoes e formularios.

- Entidades/tipos base em `src/types/entities/*`
- Enums de negocio em `src/types/enum/*`
- DTOs/tipos de cada dominio em `*.types.ts`

### Seeds e migracoes

- Ha seeds no repositorio (`packages/api/seeds`, dezenas de arquivos).
- Nao ha pasta de migrations no `packages/api`.
- Fluxo de `pretest` depende de migrations do repositorio `server` e depois popula seeds do `public-api`.

## Features e Dominios

Dominios principais em `src/resources`:

- `root`
- `api-keys`
- `services`
- `problems`
- `companies`
- `contractors` (+ `vehicles`)
- `customers`
- `locations`
- `coverage-plans`
- `coverage-plans-matrix`
- `health-complaints`
- `forms`
- `payments-lots`
- `policies` (`travel`, `residential`)
- `assistances` (`auto`, `residential`, `travel`, `funeral`, `pet`, `food-basket`, `europ`, `hero`) e modulos `shared` (comments, chats, mensagens, servicos compartilhados)

## Endpoints da API

A API possui grande cobertura (dezenas de controllers e centenas de rotas). Prefixos/areas principais:

- `GET /` (metadados da aplicacao)
- `GET /api-keys`, `GET /api-keys/:id`
- `GET|POST|PUT /services...`
- `GET|POST|PUT /problems...`
- `GET /companies...`
- `GET|POST|PUT /contractors...`
- `GET|POST|PUT /contractors/:contractorId/vehicles...`
- `GET|POST|PUT /customers...`
- `GET|POST|PUT /locations...`
- `GET /coverage-plans...`
- `GET /coverage-plans-matrix...`
- `GET /health-complaints`
- `POST /forms/answers`
- `POST /payments-lots/validate`
- `GET /travel/policies...`
- `GET /residential/policies...`

Assistencias por vertical:

- `auto/assistances`
- `residential/assistances`
- `travel/assistances`
- `funeral/assistances`
- `pet/assistances`
- `food-basket/assistances`
- `hero/assistances`
- `europ/assistances`

Operacoes recorrentes nesses fluxos:

- consulta e atualizacao de assistencia
- gestao de servicos da assistencia (create/update/accept/refuse/cancel)
- comunicacoes de servico (cancel/reschedule/confirm)
- precos e movimentos
- reembolsos
- pesquisas
- comentarios e chats de atendimento

## Rotas de Frontend

Nao aplicavel neste repositorio. O projeto e somente backend API.

## Integracoes Externas

### Parceiros e APIs

- Easy (`src/core/integration/easy/*`)
- Europ (`src/core/integration/europ/*`)
- Grow (`src/core/integration/grow/*`)
- Hero (`src/core/integration/hero/*`)

### Geolocalizacao

- Google Maps (`shared/services/geolocation/google-maps/*`)
- HERE (`shared/services/geolocation/here-service.ts`)
- Selecao dinamica por conta em `route-service.ts`

### AWS

- SQS (workers, tasks, communications)
- S3 (arquivos/presigned post)
- Secrets Manager (segredos de DB e blocos sensiveis)

## Padroes e Convencoes

- Organizacao por dominio em `src/resources`.
- DTOs e contratos em `*.types.ts` com `class-validator` e `@nestjs/swagger`.
- Mapeamento de entidade/row em `*.mapper.ts`.
- Query filters paginados por metadados (`Reflect.metadata('where', ...)`) no query builder.
- Erros padronizados com filtro custom (`custom-exception.filters.ts`) e envio para Sentry.
- Qualidade de codigo com ESLint + Prettier + Husky + Commitlint/Commitizen.

## Testes

- Runner: Jest (`packages/api/jest.config.ts`).
- Unitarios em `src/**/*.spec.ts` e `test/unit/*`.
- E2E em `test/resources/*.e2e.spec.ts`.
- Precondicao de banco via `pretest` (reset/migration no repo `server` + seeds do `public-api`).

## Deploy e Infraestrutura

- Pipeline: `.github/workflows/ci-cd.yml`
- CI: sobe Postgres de teste, faz checkout de `public-api` e `notroapp/server`, instala dependencias, roda commitlint, testes (exceto branch `preview`) e build.
- CD (`master` e `preview`): configura credenciais AWS, gera `env.json` com segredos (`generate-env.js`), empacota zip e faz deploy no Elastic Beanstalk (`public-api-prod` e `public-api-preview`).

## Variaveis de Ambiente Essenciais

Valores configurados em `packages/api/env.json` por ambiente (`dev`, `preview`, `prod`) e combinados com secrets no CI.

Chaves importantes:

- `ENVIRONMENT`
- `WORKER_QUEUE_URL`
- `WORKER_TASKS_QUEUE_URL`
- `WORKER_COMMUNICATIONS_QUEUE_URL`
- `DB_MASTER_SECRET_ID`
- `EUROP_URLS`
- `EUROP_CLIENT_SECRET`
- `HERO_URLS`
- `GROW`
- `EASY`

Tambem usadas no runtime:

- `PORT`
- `environment`
- `AWS_LAMBDA_FUNCTION_NAME`
- `NODE_ENV`

## Comandos Uteis

### Root (`/public-api`)

```bash
npm install
npm run lint
npm run lint:fix
npm run test
npm run clean
```

### API (`/public-api/packages/api`)

```bash
npm run start
npm run start:dev
npm run start:debug
npm run build
npm run test
npm run test:unit
npm run test:e2e
npm run coverage:view
npm run bin:generate-api-key
npm run db:populate:server
```

## Arquivos-Chave para Navegacao Rapida

- `packages/api/src/app.module.ts`
- `packages/api/src/bootstrap.ts`
- `packages/api/src/main.ts`
- `packages/api/src/core/db/*`
- `packages/api/src/core/integration/*`
- `packages/api/src/core/worker/*`
- `packages/api/src/resources/*`
- `.github/workflows/ci-cd.yml`
- `docs/environment-variables.md`
