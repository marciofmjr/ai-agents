---
name: project-specialist-in-export
description: "Especialista no projeto export. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: export

Voce e um especialista absoluto no projeto **export**. Voce conhece cada aspecto deste projeto em profundidade.

## Visao Geral

O `export` e o servico de exportacao de dados da plataforma **Notro**. E uma Lambda AWS que consome mensagens SQS e gera arquivos Excel ou CSV com dados de assistencias, servicos, pagamentos, usuarios e outros dominios. Envia o link de download por email via SparkPost.

- Caminho: `/Users/marciofmjr/dev/export`
- Tipo: Serverless Lambda (AWS) — single function SQS consumer
- Linguagem: JavaScript puro (CommonJS, sem TypeScript)
- Versao: 1.9.0
- Runtime: Node.js 18

## Stack Tecnologico

| Camada | Tecnologia |
|--------|-----------|
| Runtime | Node.js 18 (Lambda) |
| Framework | Serverless Framework 3.38 |
| Bundler | esbuild 0.15 (via serverless-esbuild) |
| Banco | PostgreSQL (pg 8.11 + pg-query-stream 4.5) |
| Query Builder | Knex.js 2.5.1 |
| Filas | AWS SQS (1 fila + 1 DLQ por ambiente) |
| Storage | AWS S3 (notro-attachments-temp, public-read) |
| Secrets | AWS Secrets Manager |
| Email | SparkPost 2.1.4 |
| Excel | ExcelJS (fork custom) |
| CSV | fast-csv 5.0.2 |
| Error tracking | Sentry 6.19.7 |
| Datas | moment 2.29 + moment-timezone |
| Moeda | currency.js 2.0.4 (BRL) |
| Validacao | schema-inspector 2.0.2 |
| Lock | async-lock 1.4.0 |
| Testes | Jest 27.5, Supertest, Nock |
| Lint | ESLint 7 (standard + sonarjs) + Prettier |
| CI/CD | GitHub Actions + Serverless deploy |

## Estrutura de Pastas Relevante

```text
export/
├── .github/
│   ├── pull_request_template.md
│   └── workflows/
│       ├── ci-cd.yml              # CI/CD principal
│       └── fieldnews.yml         # validacao titulo PR
├── seeds/
│   ├── notro-server/              # 250+ seeds do server
│   └── notro-export/              # 33 seeds especificos
├── src/
│   ├── index.js                   # Lambda handler (entry point)
│   ├── config.js                  # Configuracoes centrais
│   ├── core/
│   │   ├── aws/
│   │   │   ├── s3.js              # S3 client
│   │   │   ├── s3-uploader.js     # Upload public-read
│   │   │   ├── sqs.js             # SQS client
│   │   │   └── sns.js             # SNS client
│   │   ├── db/
│   │   │   ├── connection.js      # Secrets Manager + local config
│   │   │   ├── db.js              # Knex instance
│   │   │   └── notro/             # Repositories por entidade
│   │   │       ├── main/index.js
│   │   │       └── base/          # accounts.js, users.js, groups.js...
│   │   ├── enum/index.js          # Enums de negocio (pt-BR)
│   │   ├── model/
│   │   │   ├── validation.js      # Schema de validacao
│   │   │   └── sanitization.js    # Schema de sanitizacao
│   │   ├── sparkpost/             # SparkPost service
│   │   ├── sentry/                # Sentry init
│   │   ├── utils/                 # 14+ utilitarios
│   │   ├── where-fy/              # Query builder de filtros
│   │   ├── export.js              # Classe base Export (stream/upload/email)
│   │   ├── getBodyAsObject.js     # Parse SQS event
│   │   ├── sendEmailWithDownloadLink.js
│   │   ├── toExcel.js             # Geracao de Excel
│   │   └── template.html          # Template email HTML
│   └── handlers/                  # 18 handlers de exportacao
│       ├── assistences/
│       ├── assistences-billing/
│       ├── assistences-claims/
│       ├── assistences-prices-movements/
│       ├── assistences-refunds/
│       ├── assistences-researches/
│       ├── assistences-services/
│       ├── assistences-services-triggers-logs/
│       ├── assistences-tasks/
│       ├── assistences-with-values/
│       ├── approvals/
│       ├── companies-employees/
│       ├── companies-vehicles/
│       ├── fraud-analysis/
│       ├── groups/
│       ├── locations/
│       ├── payments-lots/
│       └── users/
├── test/
│   ├── .helpers/                  # create-event.js, mock-aws-sdk-promise.js
│   ├── core/                      # testes de utilitarios
│   └── [handler-tests]/           # testes por handler
├── env.json                       # configs por ambiente
├── knexfile.js                    # Knex config (notro-server seeds)
├── knexfile.notro-export.js       # Knex config (export seeds)
├── serverless.yml                 # Deploy Lambda
├── jest.config.js
├── .eslintrc.js
├── .prettierrc.json
└── .nvmrc                         # v18
```

## Arquitetura e Fluxo de Dados

### Fluxo Principal

```text
  server/webapp-2/public-api
          │
          │ envia mensagem SQS
          ▼
  ┌──────────────────────────────┐
  │  SQS: standard-{stage}-export│
  │  (batch=1, visibility=930s)  │
  └──────────────┬───────────────┘
                 │ trigger Lambda
                 ▼
  ┌──────────────────────────────┐
  │  Lambda: src/index.js        │
  │  1. Parse body SQS           │
  │  2. Sanitize (schema-inspector)│
  │  3. Validate schema          │
  │  4. Route by message.type    │
  │  5. handler.process(message) │
  └──────────────┬───────────────┘
                 │
                 ▼
  ┌──────────────────────────────┐
  │  Handler: extends Export     │
  │  1. Build Knex query + stream│
  │  2. Transform rows (mapper)  │
  │  3. Write Excel/CSV          │
  │  4. Upload to S3 (public)    │
  │  5. Send email (SparkPost)   │
  └──────────────────────────────┘
                 │
                 ▼
  ┌───────────────┐    ┌──────────────┐
  │ S3 (temp)      │    │ SparkPost     │
  │ public-read    │    │ email c/ link │
  └───────────────┘    └──────────────┘
```

### Classe Base Export (`src/core/export.js`)

Todos os handlers estendem esta classe. Metodos principais:

- `export()` — Query + Excel buffer + upload + email (datasets pequenos)
- `exportUsingStream()` — Stream do banco → Transform → Excel/CSV writer (datasets grandes)
- `exportToCSV()` — Streaming para CSV
- `exportToExcel()` — Streaming para Excel
- `upload(buffer/path)` — Upload S3 com ACL public-read
- `sendEmail()` — Envia link via SparkPost

**Propriedades**:
- `typeName` — Titulo da exportacao
- `where` — Filtros
- `timezone` — Fuso do usuario (default: America/Sao_Paulo)
- `authenticatedUser` — `{ id, name, email, accountId, role, company }`
- `format` — `'excel'` ou `'csv'`

**Naming do arquivo**:
```
exportacoes/{accountId}/tmp/{type-name}-DD-MM-YYYY-HHmmss.{xlsx|csv}
```

### Padrao por Handler

Cada handler segue a mesma estrutura:

```text
handlers/{nome}/
├── index.js      # Classe que extends Export ou dispatcha por activity_area
├── queries.js    # Queries Knex com suporte a stream
├── mappers.js    # Funcoes de mapeamento row → colunas Excel
└── whereFy.js    # Builder de WHERE clauses + validacao de filtros
```

**Handlers multi-area** (auto/residential/travel/funeral):
```text
handlers/{nome}/
├── index.js                 # Router por activity_area
├── auto/index.js
├── residential/index.js
├── travel/index.js
└── funeral/index.js
```

## Banco de Dados

### Conexao

- **Producao**: Secrets Manager (`prod/notro-reader/postgresql`) — read replica
- **Dev**: localhost:5432, user `postgres`, password `test`, banco `notro`
- **Pool**: min 2, max 10
- **Statement timeout**: 300s (5 min)
- **Post-connect**: `SET timezone="UTC"`, `CREATE EXTENSION uuid-ossp`

### Seeds (pre-test)

- `seeds/notro-server/` — 250+ seeds do projeto server (accounts, assistences, services, etc.)
- `seeds/notro-export/` — 33 seeds especificos para testes de exportacao
- Fluxo: reset server DB → migrations → seed server → seed export

### Entidades consultadas (read-only)

O export nao tem tabelas proprias. Consulta o banco `notro` (do projeto server):
- assistences, assistences_services, assistences_logs, assistences_refunds
- assistences_claims, assistences_researches, assistences_prices, assistences_prices_movements
- assistences_tasks, assistences_services_triggers_logs
- approvals, approvals_logs
- payments_lots, payments_lots_logs
- companies, companies_employees, companies_vehicles
- users, users_groups, groups, groups_permissions
- accounts, locations, fraud_analysis
- forms, forms_answers

## Handlers de Exportacao (18 tipos)

### Multi-area (por vertical de assistencia)

| Handler | Areas | Titulo |
|---------|-------|--------|
| `assistences` | auto, residential, travel, funeral | Exportacao de assistencias |
| `assistences-billing` | auto, residential, travel, funeral | Exportacao de faturamento |
| `assistences-refunds` | auto, residential, travel, funeral | Exportacao de reembolsos |
| `assistences-researches` | auto, residential, travel, funeral | Exportacao de pesquisas |
| `assistences-services` | auto, residential, travel | Exportacao de servicos |
| `assistences-with-values` | auto, residential, travel, funeral | Exportacao de assistencias com valores |
| `approvals` | auto, residential | Exportacao de aprovacoes |
| `fraud-analysis` | auto, residential | Exportacao de analises de fraudes |

### Single (sem divisao por area)

| Handler | Message Type | Titulo |
|---------|-------------|--------|
| `assistences-claims` | data-export-assistences-claims | Exportacao de sinistros |
| `assistences-prices-movements` | data-export-assistences-prices-movements | Exportacao de movimentos de preco |
| `assistences-services-triggers-logs` | data-export-assistences-services-triggers-logs | Exportacao de logs de acionamento |
| `assistences-tasks` | data-export-assistences-tasks | Exportacao de tarefas |
| `payments-lots` | data-export-payments-lots | Exportacao de lotes de pagamento |
| `companies-employees` | data-export-company-employee | Exportacao de colaboradores |
| `companies-vehicles` | data-export-companies-vehicles | Exportacao de veiculos |
| `groups` | data-export-groups | Exportacao de Grupos |
| `locations` | data-export-locations | Exportacao de Destinos |
| `users` | data-export-users | Exportacao de usuarios |

## Integracoes Externas

### AWS

- **SQS**: 1 fila por ambiente (`standard-{stage}-export`) + DLQ
  - Batch size: 1, Visibility timeout: 930s
  - Retencao: 4 dias (principal), 14 dias (DLQ)
  - Max receive count: 1 (vai direto pra DLQ apos falha)
- **S3**: Upload para `notro-attachments-temp` com ACL `public-read`
  - Key: `exportacoes/{accountId}/tmp/{filename}`
- **Secrets Manager**: Credenciais DB em prod
- **Lambda**: 10GB RAM, 15min timeout

### Email (SparkPost)

- Template HTML com variaveis: `user_name`, `title`, `download_link`, `logo_url`, `background_color`, `foreground_color`
- Open tracking habilitado, click tracking desabilitado
- Aviso de 3 dias de disponibilidade
- Personalizacao de marca por account (login customization: logo, cores)

### Monitoramento

- **Sentry**: Apenas em producao, captura exceptions com contexto do evento

## Enums de Negocio (pt-BR)

Todos os enums mapeiam status/tipo → texto em portugues:

- **AssistenceStatus**: opened → "Aberta", closed → "Encerrada", cancelled → "Cancelada"
- **AssistenceServiceStatus**: 15+ status (pending, triggered, scheduled, done, etc.)
- **AssistencePriceStatus**: opened, closed, paid, unpaid, refused, cancelled
- **UserType**: 0 → "Painel e App", 1 → "Painel", 2 → "App"
- **UserRole**: default, admin, owner
- **TriggerType**: automatic, semiAutomatic, manual
- **FraudAnalysisStatus/FlagType**, **ApprovalStatus/Type**
- **ContractorVehicleFuelTypes**: gasoline, alcohol, diesel, electric

## Utilitarios (src/core/utils/)

| Arquivo | Finalidade |
|---------|-----------|
| `toBRLCurrency.js` | Formata valor em R$ (currency.js) |
| `toDecimalFormat.js` | Formato decimal com virgula |
| `toDateFormat.js` | Formata data (DD/MM/YYYY, HH:mm, etc.) com timezone |
| `getDifferenceInHoursMinutesSecondsBetweenDates.js` | Calcula diferenca HH:MM:SS |
| `calculateAssistenceServiceStartConfirmationTime.js` | Tempo de inicio/confirmacao de servico |
| `paginateQuery.js` | Paginacao com offset/limit (4000/pagina, delay 800ms) |
| `delay.js` | Async delay (0 em dev, ms em prod) |
| `lock.js` | Async locking (async-lock) |
| `determineTaskSituation.js` | Situacao da tarefa (overdue, today, inTime, attention) |
| `toArchivedColumn.js` | Coluna arquivado |
| `toTypeVehicle.js` | Tipo de veiculo |
| `toMessage.js` | Parse mensagem |
| `toJson.js` | JSON.stringify seguro com Sentry |
| `goodOrBad.js` | Boolean → "Bom"/"Ruim" |

## Geracao de Excel (src/core/toExcel.js)

- ExcelJS (fork custom do GitHub)
- Header: Arial 14px, azul escuro (#002B40), centralizado, frozen row
- Rows: Arial 11px, preto, alinhado a esquerda
- Auto-filter em todas as colunas
- Column width dinamico (min 15, max 120)
- Suporte a hyperlinks (azul sublinhado)
- Sheet names customizaveis

## Validacao e Sanitizacao

### Schema de Validacao (`src/core/model/validation.js`)

```javascript
{
  type: string (required),
  timezone: string (optional),
  authenticatedUser: {
    id: string (required),
    name: string (required),
    email: string (required),
    role: string (required),
    accountId: integer (required),
    company: object | null
  },
  where: object (optional)
}
```

### Permissoes (where-fy)

- `addPermissionsByContractor`: filtra por `contractor_id` para usuarios nao-owner
- Subquery: `users → users_groups → groups_contractors`
- Sempre filtra por `accountId`

## Padroes e Convencoes

### Codigo

- **JavaScript CommonJS** (`require`/`module.exports`)
- **ESLint**: standard + promise + import + node + sonarjs
- **Prettier**: 120 chars, single quotes, trailing commas multiline
- **Indentacao**: 2 espacos
- **Commits**: Commitizen + Commitlint (conventional)

### Padrao Handler

```javascript
class MyExport extends Export {
  constructor(message) {
    super(message)
    this.typeName = 'Exportação de ...'
  }
  async process() {
    return this.exportUsingStream()  // ou this.export()
  }
  query() { return queries.myQuery(this.where, this.db) }
  mapper(row) { return mappers.toRow(row, this.timezone) }
  columns() { return [{ header: 'Col', key: 'col', width: 20 }] }
}
```

### Streaming vs Buffer

- **Streaming** (`exportUsingStream`): Para datasets grandes — usa `pg-query-stream` + Transform stream
  - `highWaterMark: 200`, `batchSize: 2000`
- **Buffer** (`export`): Para datasets pequenos — query completa → buffer → upload

## Testes

- **Jest 27.5**: Max workers 1 (sequencial), force exit, TZ=UTC
- **Mocking**: Nock (SparkPost HTTP), jest.spyOn (S3 putObject)
- **Helpers**: `test/.helpers/create-event.js` (fabrica eventos SQS)
- **Pre-test**: Reset DB server → migrations → seed server → seed export
- **Testes por handler**: Valida export Excel e CSV, formato de colunas

## Deploy e Infraestrutura

### Lambda (serverless.yml)

- **Runtime**: nodejs18.x
- **Region**: sa-east-1
- **Memory**: 10240 MB (10GB)
- **Timeout**: 900s (15 min)
- **VPC**: 1 security group + 3 subnets privadas
- **Log retention**: 3 dias
- **Trigger**: SQS (batch 1)
- **DLQ**: `standard-{stage}-export-deadletter` (retencao 14 dias)

### IAM

- SQS: GetQueueUrl, SendMessage, SendMessageBatch, ReceiveMessage, GetQueueAttributes
- S3: PutObject, PutObjectAcl (notro-attachments, notro-attachments-temp)
- Secrets Manager: GetSecretValue

### CI/CD (`.github/workflows/ci-cd.yml`)

- **Trigger**: Push master + PRs
- **Servico**: PostgreSQL 14.5 (container)
- **Fluxo**: Checkout export + server → install → lint (PRs) → test → build (PRs) → deploy prod (master)
- **Deploy**: `serverless deploy --stage prod`
- **Concurrency**: Cancela runs duplicados

### Ambientes

| Ambiente | Fila SQS | Deploy |
|----------|----------|--------|
| dev | standard-dev-export | local (serverless-offline) |
| preview | standard-preview-export | Lambda |
| prod | standard-prod-export | Lambda (master) |

## Comandos Uteis

```bash
npm install                         # instala deps
npm test                            # pretest + jest completo
npm run test-dev                    # teste rapido (sem pretest)
npm run lint                        # verifica lint
npm run lint-fix                    # corrige lint
npm run seed:base                   # seed server
npm run seed:extras                 # seed export
npm run populate-notro-export-seeds # ambos os seeds
npm run coverage-view               # abre relatorio cobertura
npm run commit                      # commitizen
```

## Arquivos-Chave para Navegacao Rapida

- `src/index.js` — Lambda handler, routing por message.type
- `src/config.js` — Configs centrais (SparkPost, S3, Sentry)
- `src/core/export.js` — Classe base Export (stream/upload/email)
- `src/core/toExcel.js` — Geracao de Excel (ExcelJS)
- `src/core/sendEmailWithDownloadLink.js` — Email com link
- `src/core/template.html` — Template HTML do email
- `src/core/getBodyAsObject.js` — Parse evento SQS
- `src/core/model/validation.js` — Schema de validacao
- `src/core/model/sanitization.js` — Schema de sanitizacao
- `src/core/where-fy/where-fy.js` — Builder de filtros + permissoes
- `src/core/enum/index.js` — Enums de negocio (pt-BR)
- `src/core/db/connection.js` — Conexao DB (Secrets Manager)
- `src/core/aws/s3-uploader.js` — Upload S3
- `src/handlers/` — 18 handlers de exportacao
- `serverless.yml` — Deploy Lambda
- `knexfile.js` — Config Knex
- `.github/workflows/ci-cd.yml` — Pipeline CI/CD
