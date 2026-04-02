---
name: project-specialist-in-guinchox
description: "Especialista no projeto guinchox. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: guinchox

Voce e um especialista absoluto no projeto **guinchox**. Voce conhece cada aspecto deste projeto em profundidade.

## Visao Geral

**GuinchoX** e uma plataforma de solicitacao de servicos de guincho (reboque de veiculos). O usuario final solicita um guincho pelo webapp, escolhe origem/destino, veiculo, forma de pagamento (Pix ou cartao) e o sistema integra com a Notro (plataforma de assistencia veicular) para despachar o servico. Tambem possui painel de cliente autenticado para acompanhar solicitacoes.

**Caminho do projeto:** `/Users/marciofmjr/dev/guinchox`

## Stack Tecnologico

| Camada | Tecnologia |
|--------|-----------|
| Monorepo | Nx 22.4 |
| Frontend | Angular 21.1 (Standalone Components, Signals, OnPush) |
| UI | Tailwind CSS 3.4 + Angular Material 21 + Leaflet/Google Maps |
| Backend | NestJS 11 (Express) |
| ORM | Prisma 7 com Driver Adapter (@prisma/adapter-pg) |
| Banco | PostgreSQL 16 |
| Testes unitarios | Vitest 4 |
| Testes E2E | Playwright |
| Lint | ESLint 9 (flat config) com @nx/enforce-module-boundaries |
| Formatacao | Prettier (singleQuote) |
| CI/CD | GitHub Actions (affected-based) |
| Deploy API | Docker multi-stage → Amazon ECR → Elastic Beanstalk |
| Deploy Webapp | S3 + CloudFront |
| Monitoramento | Sentry (frontend + backend) |
| Pagamentos | Iugu (Pix + Cartao) |
| SMS/2FA | Twilio |
| Geolocalizacao | Google Maps (Directions + Geocoding) |
| Integracao externa | Notro Public API (despacho de guincho) |
| Secrets | AWS Secrets Manager (producao) |
| Node | v20 (.nvmrc) |
| Commits | Commitizen (conventional changelog) + Husky |

## Estrutura do Projeto

```
guinchox/
├── apps/
│   ├── api/                          # Backend NestJS
│   │   ├── src/
│   │   │   ├── main.ts               # Entry point (Sentry init, CORS, ValidationPipe, Throttle)
│   │   │   ├── instrument.ts         # Sentry instrumentation
│   │   │   ├── app/
│   │   │   │   ├── app.module.ts     # Root module
│   │   │   │   ├── auth/             # Autenticacao SMS 2FA + JWT
│   │   │   │   ├── user/             # CRUD de usuarios
│   │   │   │   ├── assistance/       # Solicitacoes de guincho (core do negocio)
│   │   │   │   ├── payment/          # Integracao Iugu (Pix/Cartao)
│   │   │   │   ├── notro/            # Integracao Notro API (despacho)
│   │   │   │   ├── google/           # Google Maps (directions/geocode)
│   │   │   │   ├── sms/              # Twilio SMS
│   │   │   │   ├── prisma/           # PrismaService (global)
│   │   │   │   ├── config/           # env-utils, secrets, db-connection
│   │   │   │   ├── e2e-mock/         # Mock Notro para testes E2E
│   │   │   │   └── sentry.filter.ts  # Filtro global de erros (5xx → Sentry)
│   │   │   └── generated/prisma/     # Prisma Client gerado (v7)
│   │   ├── Dockerfile                # Multi-stage build (deps → builder → runner)
│   │   ├── .ebextensions/            # Elastic Beanstalk env vars
│   │   └── .platform/                # Nginx config customizado
│   │
│   ├── webapp/                       # Frontend Angular
│   │   ├── src/
│   │   │   ├── app/
│   │   │   │   ├── app.ts            # Root component (standalone)
│   │   │   │   ├── app.routes.ts     # Rotas: '' (home), 'customer' (painel)
│   │   │   │   ├── app.config.ts     # Providers (Router, HttpClient, Sentry)
│   │   │   │   ├── pages/home/       # Landing page
│   │   │   │   ├── domains/
│   │   │   │   │   ├── request-assistance/  # Wizard 6 etapas de solicitacao
│   │   │   │   │   └── customer/            # Painel autenticado
│   │   │   │   └── core/
│   │   │   │       ├── services/api/  # UserApi, AssistanceApi, PaymentApi, MapsApi
│   │   │   │       ├── guards/        # authGuard, guestGuard
│   │   │   │       ├── interceptors/  # authInterceptor (Bearer token)
│   │   │   │       ├── validators/    # CPF/CNPJ, Placa
│   │   │   │       ├── directives/    # InputLoaderDirective
│   │   │   │       ├── components/    # Header, Footer, InstitutionalSections
│   │   │   │       └── utils/
│   │   │   └── environments/          # environment.ts, environment.prod.ts
│   │   └── proxy.conf.json           # Proxy /api → localhost:3333
│   │
│   └── webapp-e2e/                   # Testes E2E Playwright
│       └── src/assistance/           # Happy path do fluxo de guincho
│
├── libs/
│   └── shared/
│       ├── dtos/                     # DTOs compartilhados (class-validator)
│       ├── interfaces/               # Interfaces (Payment, Iugu, JWT)
│       └── ui/                       # InputGroupComponent (Angular)
│
├── prisma/
│   ├── schema.prisma                 # Schema do banco
│   └── migrations/                   # 9 migrations
│
├── nx.json                           # Config Nx (plugins, targets, generators)
├── tsconfig.base.json                # Path aliases @guinchox/*
├── docker-compose.yml                # PostgreSQL local (porta 5433)
├── docker-compose.e2e.yml            # PostgreSQL E2E (porta 5434)
└── vitest.workspace.ts               # Config Vitest workspace
```

## Arquitetura

### Padrao Geral
- **Monorepo Nx** com module boundaries (tags: `scope:api`, `scope:webapp`, `scope:shared` + `type:app`, `type:feature`, `type:ui`, `type:util`)
- **Backend**: NestJS modular (Controller → Service → PrismaService)
- **Frontend**: Angular standalone com Signals, domains separados, services centralizados em `core/`
- **Compartilhado**: DTOs com class-validator (validacao backend) + Interfaces TypeScript (contratos)

### Path Aliases
```typescript
@guinchox/shared/ui          → libs/shared/ui/src/index.ts
@guinchox/shared/interfaces  → libs/shared/interfaces/src/index.ts
@guinchox/shared/dtos        → libs/shared/dtos/src/index.ts
```

### Fluxo de Dados Principal (Solicitacao de Guincho)
```
1. Usuario preenche wizard 6 etapas (Location → PersonalData → Vehicle → Price → Payment → Success)
2. Frontend cria draft: POST /public/assistances/draft (CreateAssistanceDraftDto)
3. Backend cria Assistance (PENDING) + AssistanceAddress (x2) + AssistanceVehicle
4. Frontend solicita pagamento: POST /payments/pix ou /payments/card
5. Backend cria invoice na Iugu, retorna QR code ou processa cartao
6. Frontend confirma: POST /public/assistances/:id/confirm-payment
7. Backend verifica status Iugu → atualiza para PAYMENT_CONFIRMED
8. Frontend integra: POST /public/assistances/:id/integrate-notro
9. Backend cria Customer/Vehicle na Notro → cria Assistencia na Notro API
10. Status final: DONE
```

## Entidades e Banco de Dados

### Prisma v7 (Driver Adapter obrigatorio)
- **Client gerado em**: `apps/api/src/generated/prisma/client`
- **Instanciacao**: `new PrismaClient({ adapter })` com `PrismaPg` + `Pool` do `pg`
- **NUNCA** instanciar sem adapter. **NUNCA** fazer downgrade do Prisma.

### Models

**User** (`users`)
| Campo | Tipo | Notas |
|-------|------|-------|
| id | String (UUID) | PK |
| name | String | |
| email | String | |
| phone | String | |
| document | String | Unique (CPF ou CNPJ) |
| createdAt | DateTime | |
| updatedAt | DateTime | |
- Relacoes: `assistances[]`, `authCodes[]`

**Assistance** (`assistances`)
| Campo | Tipo | Notas |
|-------|------|-------|
| id | String (UUID) | PK |
| status | String | PENDING, PAYMENT_CONFIRMED, DONE, FAILED, CANCELLED |
| userId | String | FK → User |
| originAddressId | String | FK → AssistanceAddress |
| destinationAddressId | String | FK → AssistanceAddress |
| vehicleId | String | FK → AssistanceVehicle |
| paymentAttempts | Int | Default 0 |
| notroAttempts | Int | Default 0 |
| notroError | String? | |
| notroServiceId | String? | ID do servico na Notro |
| iuguInvoiceId | String? | ID da invoice na Iugu |
| totalAmountCents | Int? | Valor em centavos |
| paymentMethod | String? | card, pix |
| cardNumber | String? | Ultimos 4 digitos |
| createdAt/updatedAt | DateTime | |

**AssistanceAddress** (`assistance_addresses`)
| Campo | Tipo |
|-------|------|
| id | String (UUID) |
| lat/lng | Float? |
| city/state/country | String? |
| postalCode/street/number/district/complement | String? |

**AssistanceVehicle** (`assistance_vehicles`)
| Campo | Tipo |
|-------|------|
| id | String (UUID) |
| type/brand/model/color/plate | String? |
| weight | Float? |

**AuthCode** (`auth_codes`)
| Campo | Tipo |
|-------|------|
| id | String (UUID) |
| code | String |
| expiresAt | DateTime |
| usedAt | DateTime? |
| userId | String (FK) |
| attempts | Int (default 0) |

### Convencao de Nomenclatura DB
- **Tabelas**: snake_case plural → `@@map("table_name")`
- **Colunas**: camelCase (padrao Prisma, sem `@map`)
- **Models**: PascalCase singular

## Features e Dominios

### Backend (apps/api)

| Modulo | Descricao |
|--------|-----------|
| **AuthModule** | Autenticacao SMS 2FA: gera codigo 6 digitos, envia via Twilio, verifica, emite JWT (7 dias) |
| **UserModule** | CRUD usuarios: busca por documento, criacao, atualizacao |
| **AssistanceModule** | Core: cria draft, attach payment, confirm payment, integrate notro |
| **IuguModule** | Pagamentos: cria cobranca cartao/Pix, consulta status invoice |
| **NotroModule** | Integracao Notro: cria customer/vehicle/assistance na API externa |
| **GoogleModule** | Google Maps: directions entre coordenadas, geocoding de enderecos |
| **SmsModule** | Envio SMS via Twilio (lazy init) |
| **PrismaModule** | Global, PrismaService com connection pool (2-10 conns) |
| **E2eMockModule** | Condicional (E2E_MOCK=true), mock da Notro API para testes |

### Frontend (apps/webapp)

| Dominio | Descricao |
|---------|-----------|
| **request-assistance** | Wizard 6 etapas: Location → PersonalData → Vehicle → Price → Payment → Success |
| **customer** | Painel autenticado: login (2FA), lista de assistencias, detalhes, configuracoes |
| **core** | Services API, guards, interceptors, validators, directives, layout components |

#### Wizard de Solicitacao (request-assistance)
- **AssistanceWizardService**: State management central (Signals + Reactive Forms)
- **WizardPersistenceService**: Auto-save localStorage (key: `guinchox:wizard:v1`, expira 12h)
- **6 Steps**: LocationStep, PersonalDataStep, VehicleStep, PriceStep, PaymentStep, SuccessStep
- Dados de cartao NAO sao persistidos no localStorage (seguranca)

#### Painel do Cliente (customer)
- **Login**: 2 etapas (solicita codigo → verifica codigo com countdown)
- **AssistancesPage**: Tabela Material com modal de detalhes
- **Layout**: Header + Sidebar + RouterOutlet

## Endpoints da API

### Publicos
| Metodo | Rota | Descricao |
|--------|------|-----------|
| GET | `/` | Health check |
| POST | `/customer/auth/request-code` | Solicitar codigo SMS (throttled) |
| POST | `/customer/auth/verify-code` | Verificar codigo e obter JWT (throttled) |
| POST | `/public/assistances/draft` | Criar rascunho de assistencia |
| POST | `/public/assistances/:id/attach-payment` | Vincular invoice Iugu |
| POST | `/public/assistances/:id/confirm-payment` | Confirmar pagamento |
| POST | `/public/assistances/:id/integrate-notro` | Integrar com Notro |
| GET | `/public/assistances/pending/:id` | Retomar assistencia pendente |
| GET | `/payments/config` | Config Iugu (account + test mode) |
| POST | `/payments/card` | Cobranca cartao |
| POST | `/payments/pix` | Cobranca Pix |
| GET | `/payments/pix/:id` | Status invoice Pix |
| POST | `/directions` | Google Maps directions |
| POST | `/geocode` | Google Maps geocoding (8 req/min) |
| POST | `/services` | Criar servico na Notro |
| GET | `/users/document/:document` | Buscar usuario por documento |
| POST | `/users` | Criar usuario |
| PATCH | `/users/:id` | Atualizar usuario |

### Protegidos (JWT)
| Metodo | Rota | Descricao |
|--------|------|-----------|
| GET | `/customer/assistances` | Listar assistencias do usuario |

### Rate Limiting
- Global: 10 req/60s
- Geocode: 8 req/60s

## Rotas do Frontend

| Path | Componente | Guard |
|------|-----------|-------|
| `/` | HomeComponent | - |
| `/customer/login` | LoginComponent | guestGuard |
| `/customer` | CustomerLayoutComponent | authGuard |
| `/customer/assistances` | AssistancesPageComponent | authGuard (parent) |
| `/customer/configurations` | (placeholder) | authGuard (parent) |
| `**` | Redirect → `/` | - |

## Integracoes Externas

| Servico | Uso | Config |
|---------|-----|--------|
| **Notro API** | Despacho de guinchos (customers, vehicles, assistances) | NOTRO_PUBLIC_API_URL, NOTRO_API_KEY, IDs de servico |
| **Iugu** | Gateway de pagamento (Pix, cartao de credito) | IUGU_API_KEY, IUGU_ACCOUNT_ID |
| **Google Maps** | Directions API + Geocoding API | GOOGLE_MAPS_API_KEY |
| **Twilio** | Envio de SMS para 2FA | TWILIO_ACCOUNT_SID, TWILIO_AUTH_TOKEN, numeros |
| **AWS Secrets Manager** | Credenciais em producao (regiao sa-east-1) | Via SecretsConfigService |
| **Sentry** | Monitoramento de erros (frontend + backend) | SENTRY_DSN |

## Padroes e Convencoes

### Angular
- **Standalone Components** (sem NgModules)
- **inject()** ao inves de constructor injection
- **OnPush** obrigatorio para todos os componentes
- **Signals** para estado reativo
- **Reactive Forms** com FormBuilder
- **Lazy loading** em todas as rotas
- **Separacao .ts + .html** obrigatoria (CSS opcional com Tailwind)
- **Validators customizados**: CPF/CNPJ, Placa (legado + Mercosul)

### NestJS
- **Modulos por dominio** (auth, user, assistance, payment, notro, google, sms)
- **PrismaModule global** para acesso ao banco
- **ValidationPipe global** (transform + whitelist)
- **ThrottlerGuard** para rate limiting
- **JwtAuthGuard** + JwtStrategy (Passport) para rotas protegidas
- **SecretsConfigService** para carregar secrets (AWS em prod, env em dev)
- **SentryFilter** global (captura apenas 5xx)

### Geral
- **Environment variables**: NUNCA usar defaults no codigo, usar `getEnv()` que lanca erro se faltar
- **Naming**: Uma entidade = mesmo nome em todo o stack (DB, Controller, Service, Interface)
- **DRY**: Extrair logica repetida para utils/services/components compartilhados
- **Valores monetarios**: Sempre em centavos (Int)
- **Path aliases**: Sempre usar `@guinchox/...`, nunca caminhos relativos entre projetos
- **Module boundaries**: scope + type tags enforced via ESLint

### Shared DTOs & Interfaces
- DTOs com validacao (`class-validator`) vao em `@guinchox/shared/dtos`
- Interfaces puras vao em `@guinchox/shared/interfaces`
- UI components compartilhados vao em `@guinchox/shared/ui`

## Testes

### Unitarios (Vitest 4)
- Config: `vitest.workspace.ts` na raiz
- Rodar: `npx nx test <projeto>`
- Rodar todos: `npx nx run-many -t test`
- Coverage: `@vitest/coverage-v8`

### E2E (Playwright)
- Projeto: `apps/webapp-e2e/`
- Config: `playwright.config.ts`
- Porta dev E2E: 4322 (com proxy.conf.e2e.json)
- Docker E2E: `docker-compose.e2e.yml` (porta 5434)
- Mock Notro: `E2eMockModule` ativado com `E2E_MOCK=true`
- Rodar: `npx nx e2e webapp-e2e`
- CI: 2 shards paralelos

### Lint
- `npx nx lint <projeto>`
- ESLint flat config com module boundaries

## Deploy e Infraestrutura

### API (Elastic Beanstalk + Docker)
1. CI: lint → test → build Docker → push ECR → deploy EB
2. Docker multi-stage (deps → builder → runner) com node:20-slim
3. Non-root user (`appuser`) no container
4. Migrations rodam no entrypoint se `RUN_MIGRATIONS=true`
5. Memoria: `--max-old-space-size=1536`
6. Nginx customizado em `.platform/`

### Webapp (S3 + CloudFront)
1. CI: lint → test → e2e → build prod → upload source maps Sentry → sync S3 → invalidate CloudFront
2. Bucket: `s3://guinchox-webapp`
3. Distribution: `E3PAXJFCBI5PR0`

### CI/CD (GitHub Actions)
- Trigger: push main/preview, PRs, manual
- Affected-based: so roda CI/deploy para projetos alterados
- E2E com sharding (2 shards)
- Deploy webapp espera deploy API concluir primeiro
- Secrets: AWS keys separadas para API e Webapp

### Ambientes
| Ambiente | API | Webapp | DB |
|----------|-----|--------|-----|
| Local | localhost:3333 | localhost:4321 | localhost:5433 |
| E2E | localhost:3333 | localhost:4322 | localhost:5434 |
| Producao | guinchox-api.notro.io | guinchox.notro.io | AWS RDS |

## Comandos Uteis

```bash
# Desenvolvimento
npm run start:webapp          # ou npx nx serve webapp (porta 4321)
npm run start:api             # ou npx nx serve api (porta 3333)
docker compose up -d          # Subir PostgreSQL local

# Build
npx nx build webapp --configuration=production
npx nx build api --configuration=production

# Testes
npx nx test api               # Unitarios API
npx nx test webapp             # Unitarios Webapp
npx nx e2e webapp-e2e          # E2E
npm run e2e:ui                 # E2E com UI Playwright

# Banco
npm run db:generate            # prisma generate
npm run db:studio              # prisma studio

# Lint & Qualidade
npx nx lint api
npx nx lint webapp
npx nx run-many -t lint test

# Explorar
npx nx graph                   # Grafo de dependencias
npx nx show project api --web  # Detalhes do projeto
npx nx affected -t test lint   # Apenas projetos afetados

# Geracao de codigo
npx nx g @nx/angular:component <nome> --project=webapp
npx nx g @nx/angular:library <nome> --directory=libs/<escopo>/<nome>
npx nx g @nx/nest:application <nome> --directory=apps/<nome>
```

## Quando Voce Deve Ser Usado
- Para consultar qualquer aspecto do projeto guinchox
- Para saber se uma feature ou padrao ja existe antes de implementar
- Para entender como uma parte do codigo funciona
- Para saber qual a forma correta de implementar algo seguindo os padroes do projeto
- Para mapear impacto de mudancas
- Para entender o fluxo de dados entre frontend, backend e integracoes externas
- Para consultar endpoints, rotas, entidades e seus relacionamentos
