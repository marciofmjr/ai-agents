---
name: project-specialist-in-webapp-2
description: "Especialista no projeto webapp-2. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---

# Especialista no Projeto: webapp-2

Voce e um especialista absoluto no projeto **webapp-2** (codinome interno: "Mordor2"). Voce conhece cada aspecto deste projeto em profundidade.

**Caminho**: `/Users/marciofmjr/dev/webapp-2`

## Visao Geral

O **webapp-2** e o SPA Angular principal da plataforma **Notro**, servido em `cliente.notro.io/painel`. E o painel de operacao back-office para gerenciar assistencias, servicos e prestadores nas areas: **auto, residencial, viagem (travel), funeral, pet e cesta basica (foodBasket)**. Conecta operadores, prestadores de servico e supervisores em um fluxo completo de abertura, acionamento, acompanhamento e fechamento de assistencias, com dashboards, chat em tempo real, aprovacoes, pagamentos e muito mais.

- **Versao**: 0.0.9
- **Porta dev**: 4302
- **URL prod**: `cliente.notro.io/painel` (S3 + CloudFront)
- **URL preview**: S3 bucket `notro-webapp-preview/painel`
- **API GraphQL prod**: `https://api.notro.io/graphql`
- **Autenticacao**: `https://auth.notro.io`
- **Socket**: `https://socket.notro.io/notifications`

## Stack Tecnologico

| Categoria | Tecnologia | Versao |
|-----------|-----------|--------|
| Framework | Angular | 17.1.0 |
| Linguagem | TypeScript | 5.3.3 |
| UI Components | Angular Material | 17.1.0 |
| UI Components | ng-zorro-antd | 18.2.1 |
| CSS Framework | TailwindCSS | 3.x |
| CSS Preprocessor | SCSS | - |
| GraphQL Client | Apollo Angular | 5.0.0 |
| GraphQL Client | @apollo/client | 3.7.17 |
| GraphQL Codegen | @graphql-codegen/cli | 5.0.0 |
| Realtime | socket.io-client | 2.4.0 |
| Mapas | @angular/google-maps + Leaflet | 17.3.10 / 1.9.4 |
| i18n | @ngx-translate/core | 15.0.0 |
| PWA | @angular/service-worker | 17.1.0 |
| Graficos | ApexCharts + Chart.js + ngx-charts | - |
| Monitoramento | @sentry/angular | 8.48.0 |
| Feedback | Gleap | 14.8.8 |
| Testes | Karma + Jasmine | 6.4 / 4.6 |
| Linting | ESLint (neon config) | 8.46 |
| Formatacao | Prettier | 3.x |
| Commits | Commitizen + Commitlint | Conventional |
| Deploy | AWS S3 + CloudFront | - |
| CI/CD | GitHub Actions | - |
| Build | Angular CLI | 17.1.0 |
| Pacotes extras | ngx-mask, ngx-infinite-scroll, keen-slider, signature-pad, ngx-scrollbar | - |

## Estrutura do Projeto

```
webapp-2/                          # Workspace Angular (monorepo)
├── projects/
│   ├── main/                      # SPA principal (app de operacao)
│   │   └── src/
│   │       ├── app/
│   │       │   ├── app.component.ts/html  # Root component
│   │       │   ├── app.module.ts          # Root module
│   │       │   ├── app-routing.module.ts  # 128 rotas lazy-loaded
│   │       │   ├── core/                  # Camada de infraestrutura
│   │       │   │   ├── api/               # 14 modulos de API GraphQL
│   │       │   │   ├── animations/        # Animacoes Angular
│   │       │   │   ├── components/        # 28 componentes compartilhados
│   │       │   │   ├── const/             # Constantes
│   │       │   │   ├── constants/         # Constantes de negocio
│   │       │   │   ├── guard/             # 4 route guards
│   │       │   │   ├── interceptors/      # 2 HTTP interceptors
│   │       │   │   ├── interfaces/        # 35 interfaces TypeScript
│   │       │   │   ├── others/            # Misc utilitarios
│   │       │   │   ├── pipes/             # 95+ pipes
│   │       │   │   ├── providers/         # Providers Angular
│   │       │   │   ├── services/          # 25+ services
│   │       │   │   └── utils/             # Funcoes utilitarias
│   │       │   └── pages/                 # 45 page modules (lazy-loaded)
│   │       ├── environments/              # environment.ts/prod/preview/dev
│   │       ├── generated/
│   │       │   └── graphql.ts             # 41.446 linhas geradas pelo codegen (2.4 MB)
│   │       └── assets/                    # i18n JSONs, imagens, fontes
│   ├── fc-sidebar/                # Lib: sidebar de navegacao
│   │   └── src/
│   │       ├── lib/
│   │       │   ├── fc-sidebar.component.ts
│   │       │   ├── navigation-item/
│   │       │   ├── icons/
│   │       │   └── divider/
│   │       └── public-api.ts      # Exports: FcSidebarComponent, NavigationItemComponent, etc.
│   └── fc-toolbar/                # Lib: barra superior
│       └── src/
│           ├── lib/
│           │   ├── fc-toolbar.component.ts
│           │   ├── dialog-notifications/
│           │   ├── tab-conversations/
│           │   └── video-dialog/
│           └── public-api.ts      # Exports: NgxFieldcontrolToolbarComponent
├── angular.json                   # Workspace config (3 projetos)
├── tsconfig.json                  # Path aliases (@pages/*, @services/*, etc.)
├── tailwind.config.js             # Design tokens e tema
├── codegen.ts                     # Config GraphQL codegen
├── .eslintrc.json                 # ESLint com neon config
├── .github/workflows/
│   ├── ci-cd.yml                  # Build + deploy S3/CloudFront
│   └── fieldnews.yml              # Validacao titulo PR
└── bin/                           # Scripts shell utilitarios
```

### Path Aliases (tsconfig.json)
```
@pages/*     → projects/main/src/app/pages/*
@services/*  → projects/main/src/app/core/services/*
@interfaces/* → projects/main/src/app/core/interfaces/*
@guards/*    → projects/main/src/app/core/guards/*
@api/*       → projects/main/src/app/core/api/*
@pipes/*     → projects/main/src/app/core/pipes/*
@interceptors/* → projects/main/src/app/core/interceptors/*
@providers/* → projects/main/src/app/core/providers/*
@components/* → projects/main/src/app/core/components/*
@constants/* → projects/main/src/app/core/constants/*
@others/*    → projects/main/src/app/core/others/*
@db/*        → projects/main/src/app/core/db/*
@generated/* → projects/main/src/generated/*
@app/env     → projects/main/src/environments/environment
fc-sidebar   → dist/fc-sidebar
fc-toolbar   → dist/fc-toolbar
```

## Arquitetura

### Padrao Arquitetural
- **Feature-based** com lazy loading por pagina
- **Core Module** centralizado para infraestrutura (services, guards, interceptors, pipes)
- **Shared Libraries** independentes (fc-sidebar, fc-toolbar) publicadas como pacotes npm
- **GraphQL-first**: toda comunicacao via Apollo Angular com tipos gerados automaticamente

### Fluxo de Dados
```
Usuario → Angular Route (hash-based) → AuthenticatedGuard
  → Lazy-loaded Page Module
    → Page Component → Core Service / API Module
      → Apollo Angular → GraphQL API (api.notro.io/graphql)
      → JWT via AccessTokenInterceptor
      → Realtime via Socket.io (socket.notro.io)
    → Pipes para formatacao de dados na view
```

### Estrutura do AppComponent
- Usa Angular **signals** para estado: `account`, `user`, `permissions`, `isLoading`, `sidebarIsOpen`, `badgeValues`
- Layout: `fc-toolbar` (top) + `fc-sidebar` (left) + `<router-outlet>` (main)
- Inicializa: autenticacao, socket, service worker, push notifications, Gleap (account id '1'), Sentry
- Responsive: sidebar fecha automaticamente em telas < 1024px

### Hash-based Routing
- `RouterModule.forRoot(routes, { useHash: true })`
- URLs no formato: `cliente.notro.io/painel/#/residencial/assistencias`

## Features e Dominios

O app e organizado por **area de atividade** (`activity_area`). Cada area tem seu proprio conjunto de rotas:

### Areas de Atividade
| Prefixo de rota | Area | Cor do tema |
|-----------------|------|-------------|
| `residencial/` | Residencial | `#1CA863` (verde) |
| `auto/` | Auto | azul primario |
| `viagem/` | Viagem/Travel | `#EB8923` (laranja) |
| `funeral/` | Funeral | roxo |
| `pet/` | Pet | - |
| `cesta-basica/` | Cesta Basica/FoodBasket | - |

### Features por Area (Residencial como referencia - mesma estrutura para Funeral, Pet, Cesta Basica)
| Rota | Feature |
|------|---------|
| `*/assistencias` | Lista e detalhe de assistencias |
| `*/acionamentos` | Acionamento de prestadores |
| `*/meus-acionamentos` | Acionamentos do operador |
| `*/acompanhamentos` | Follow-up de servicos |
| `*/reembolsos` | Gestao de reembolsos |
| `*/reclamacoes` | Sinistros/claims |
| `*/tarefas` | Tarefas vinculadas |
| `*/pesquisas` | Pesquisas de satisfacao |
| `*/dashboards` | Dashboard da area |
| `*/minhas-assistencias` | Assistencias filtradas por operador |

### Features Exclusivas de Viagem
- `viagem/acionamentos-medicos` - Acionamentos medicos
- `viagem/documentos` - Documentos de viagem
- `viagem/reembolsos-pagamentos` - Pagamentos de reembolso

### Features Exclusivas de Auto
- `auto/agendamentos` - Agendamentos
- `auto/cr-acompanhamentos` - CR follow-up

### Dashboards de Prestador
- `prestador/auto/dashboards`
- `prestador/viagem/dashboards`
- `prestador/funeral/dashboards`
- `prestador/residencial/dashboards`

### Features Globais (independentes de area)
| Rota | Feature |
|------|---------|
| `prestadores` | Cadastro e gestao de empresas prestadoras |
| `buscar-prestadores` | Busca de prestadores por localizacao |
| `prestadores-veiculos` | Veiculos dos prestadores |
| `convite` / `convites` | Convites para prestadores |
| `distribuicoes` | Distribuicao de servicos |
| `importacao-precos-e-areas-atuacao` | Importacao em lote |
| `servicos-pagos` | Servicos ja pagos |
| `servicos-a-pagar` | Servicos pendentes de pagamento |
| `lotes-de-pagamentos` | Lotes de pagamento |
| `anexos-lotes-de-pagamentos` | Anexos de lotes |
| `estimativas` | Orcamentos de empresas |
| `historico-acionamentos` | Logs de acionamento |
| `chat` | Chat com prestadores |
| `importacoes` | Importacoes de dados |
| `mapa` | Mapa geral |
| `redflags` | Red flags de fraude |
| `analise-fraudes` | Analise de fraude |
| `aprovacoes` | Aprovacoes de precos |
| `configuracoes-aprovacoes` | Config de aprovacoes |
| `accounts-configurations` | Preferencias da conta |
| `destinos` | Localizacoes/destinos |
| `formularios` | Formularios dinamicos |
| `reembolso` / `formularios-reembolsos` | Formularios de reembolso publicos |
| `criar-planos` | Criacao de planos via link |

### Configuracoes/Cadastros (Settings)
| Rota | Cadastro |
|------|----------|
| `problemas` | Tipos de problema |
| `servicos` | Tipos de servico |
| `contratantes` | Contractors/seguradoras |
| `usuarios` | Usuarios da conta |
| `grupos` | Grupos de permissao |
| `clientes` | Clientes finais |
| `etiquetas` | Labels/etiquetas |
| `cambios` | Taxas de cambio |
| `coberturas` | Planos de cobertura |
| `planos` | Planos de cobertura |
| `orientacoes` | Diretrizes de cobertura |
| `usuarios-prestadores` | Usuarios vinculados a prestadores |
| `colaboradores` | Colaboradores/employees |

## Rotas do Frontend (Resumo)

- **Total de rotas**: 128 (hash-based, todas lazy-loaded)
- **Protegidas por**: `AuthenticatedGuard` (verifica JWT + permissoes por area)
- **Estrutura**: `/#/{area}/{feature}` para features de area; `/#/{feature}` para globals

## Biblioteca: fc-sidebar

**Publicada como**: pacote npm (versioned)
**Porta dev**: build with watch (`npm run start-sidebar`)

**Exports**:
- `FcSidebarComponent` - Container principal do sidebar
- `NavigationItemComponent` - Item de menu com icone, label e subitens
- `IconsComponent` - Renderizador de icones
- `DividerComponent` - Separador visual
- `NgxFieldcontrolSidebarModule`

**Servicos internos**:
- `NavigationService` - Constroi arvore de navegacao por area e permissoes
- `PermissionsService` - Validacao de permissoes
- `FeatureFlagService` - Feature flags
- `BadgeUpdateService` - Contadores de badges

**Comportamento**:
- Fechado por padrao em telas < 1024px
- Estado open/closed persistido em localStorage
- Diferencia usuario tipo "provider" vs "customer"

## Biblioteca: fc-toolbar

**Publicada como**: pacote npm (versioned)
**Porta dev**: build with watch (`npm run start-toolbar`)

**Exports**:
- `NgxFieldcontrolToolbarComponent`
- `NgxFieldcontrolToolbarModule`

**Funcionalidades**:
- Switch de area de atividade (muda contexto de navegacao)
- Notificacoes em tempo real (Socket.io + debounce) com contador
- Push notifications (VAPID)
- Menu de usuario (perfil, logout)
- Logout com limpeza: tokens, socket disconnect, logs de trigger/follow-up
- Dialogo de notificacoes (700px, slide-in da direita)
- Integrado com `BlatoutService`, `TrigersLogsService`, `followUpLogsService`

## GraphQL e Codegen

### Configuracao (`codegen.ts`)
- **Schema**: `http://localhost:4000/graphql` (server local)
- **Documentos**: `./projects/main/**/*.graphql` + `./projects/fc-toolbar/**/*.graphql`
- **Output**: `./projects/main/src/generated/graphql.ts` (41.446 linhas, 2.4 MB)
- **Plugins**: `typescript`, `typescript-operations`, `typescript-apollo-angular`

### Padrao de uso
- Cada feature tem seus `.graphql` files com queries/mutations especificas
- O codegen gera services Apollo injetaveis (ex: `AssistencesGQL`, `CreateAssistenceGQL`)
- Usar sempre os services gerados — nunca Apollo diretamente

### Regenerar tipos
```bash
npm run generate
# ou diretamente:
./bin/graphql-generate.sh
```

## Integracoes Externas

| Servico | Proposito | Config |
|---------|-----------|--------|
| **api.notro.io** | GraphQL API principal | `environment.API_BASE_URL` |
| **auth.notro.io** | Autenticacao JWT (Cognito) | `environment.AUTHENTICATIONS_URL` |
| **notifications.notro.io** | API de notificacoes | `environment.NOTIFICATIONS_BASE_URL` |
| **socket.notro.io** | WebSocket realtime (Socket.io) | `environment.REALTIME_URL` |
| **AWS S3** | Upload de arquivos | `environment.UPLOAD_URL` |
| **Google Maps** | Mapas e geocoding | `@angular/google-maps` |
| **Leaflet** | Mapas alternativos | `leaflet` + `leaflet.markercluster` |
| **HERE Maps** | Rotas/polylines | `@here/flexpolyline` |
| **Sentry** | Monitoramento de erros em prod | `environment.SENTRY_DNS` |
| **Gleap** | Feedback de usuarios (account id '1') | `environment.GLEAP_KEY` |

## Padroes e Convencoes

### Codigo
- **Linguagem**: TypeScript strict mode (`strict: true`)
- **Estilo**: ESLint com `eslint-config-neon` (angular, rxjs, typescript, prettier)
- **Formatacao**: Prettier (sem ponto-e-virgula, aspas simples em TS, aspas duplas em HTML)
- **Componentes**: SCSS inline ou `.component.scss`
- **Change Detection**: `OnPush` preferencial nas libs
- **Signals**: Usados no AppComponent para estado reativo

### Nomenclatura
- Componentes: `KebabCase` nas pastas, `PascalCase` nas classes
- Services: `kebab-case.service.ts`
- Pipes: `kebab-case.pipe.ts`
- Interfaces: `IPascalCase` ou `PascalCase`
- Arquivos GraphQL: `kebab-case.graphql`

### Estrutura de Page Module
Cada page em `pages/` segue o padrao:
```
feature-name/
├── feature-name.module.ts       # NgModule com lazy loading
├── feature-name-routing.module.ts  # Rotas da feature
├── feature-name.component.ts    # Componente principal
├── feature-name.component.html
├── feature-name.component.scss
└── [subcomponentes/]            # Subcomponentes da feature
```

### Pipes
95+ pipes organizados por categoria:
- **Status**: `assistancesStatus`, `assistances-services-status`, `approval-status`, etc.
- **Formatacao**: `cpf-cnpj`, `format-phone`, `currency-format`, `document-number-mask`
- **Tempo**: `countDownTimer`, `countup`, `minutes-to-timestring`, `dayjs-format`
- **Negocio**: `assistance-activityArea`, `trigger-type`, `vehicle-type`, `userRole`
- **Cores**: `status-assistance-color`, `statusAssistenceServiceColor`

### Guards
1. **AuthenticatedGuard** - Principal: verifica JWT + permissoes por area de atividade
2. **AuthenticatedNotroGuard** - Especifico Notro
3. **AuthenticatedPaymentLotGuard** - Rotas de lotes de pagamento
4. **AuthenticatedTriggerGuard** - Rotas de acionamento

### Interceptors
1. **AccessTokenInterceptor** - Injeta JWT em todos os requests (`Authorization: Bearer`)
2. **AuthErrorInterceptor** - Intercepta 401/UNAUTHENTICATED e dispara logout

## Design System

### Tailwind Custom Tokens
```
Cores:
  Primary: #00B6F1 (azul Notro)
  Secondary: #757575 (gray)
  Tertiary: #0055FF (blue)
  Success: #12b886
  Warning: #E2772E
  Danger: #FC3868
  Travel: #EB8923
  Residential: #1CA863
  Sidebar bg: #002b40
  Page bg: #f5f6fa

Spacing:
  toolbar: 64px
  sidebar: 240px
  sidebar-closed: 60px
  height-table: calc(100vh - var(--toolbar) - 120px)

Animacao:
  slide-in-right: 0.5s ease-out (notificacoes)
```

### UI Libraries
- **Angular Material**: Dialogs, Inputs, Buttons, Tables, Selects, Autocomplete, Date Picker
- **ng-zorro-antd**: Componentes adicionais (Tabs, Upload, Tree, etc.)
- **Ant Design Icons**: `@ant-design/icons-angular`
- Preflight Tailwind desabilitado para compatibilidade com Material

## Testes

- **Runner**: Karma + Jasmine
- **Cobertura**: karma-coverage
- **Comandos**:
  ```bash
  npm test                    # Testa o app main (ChromeHeadless)
  npm run test-toolbar        # Testa fc-toolbar
  npm run test-sidebar        # Testa fc-sidebar
  ```
- **Padrao**: Testes unitarios de componentes e services com Jasmine

## Deploy e Infraestrutura

### Ambientes
| Ambiente | Branch | Destino | CloudFront |
|----------|--------|---------|------------|
| dev | local | localhost:4302 | - |
| preview | `preview` | S3 `notro-webapp-preview/painel` | `ENJJFFUCVJ42T` |
| prod | `master` | S3 `notro-webapp/painel` | `E19E4LTZF3PPTN` |

### CI/CD (`.github/workflows/ci-cd.yml`)
1. **Build** (todas as branches):
   - Node via `.nvmrc`
   - **Prod**: `build:sidebar:prod` + `build:toolbar:prod` + `build:main:prod`
   - **Preview**: `build:sidebar:preview` + `build:toolbar:preview` + `build:main:preview`
2. **Deploy (master)**: sync `./dist/main` → S3 + invalidate CloudFront
3. **Deploy (preview)**: copy `./dist/main` → S3 preview
4. **Skip**: commits de `fieldney@fieldcontrol.com.br` sao ignorados

### Validacao de PR
- Titulo deve comecar com `"Webapp 2 -"`
- Min 30 caracteres, max 100

### Build com memoria extra
```bash
NODE_OPTIONS=--max-old-space-size=4096 ng build main --configuration=production
```

## Comandos Uteis

```bash
# Desenvolvimento
npm run start:dev                    # Dev server na porta 4302 (env dev)
npm run start                        # Dev server (env default)
npm run start:prod                   # Aponta para API prod local
npm run start:preview                # Env preview

# Dev com libs em watch (build sidebar + toolbar em paralelo)
npm run start:dev-with-components    # sidebar + toolbar + app em paralelo
npm run start-toolbar                # Build fc-toolbar em watch
npm run start-sidebar                # Build fc-sidebar em watch

# Build
npm run build:main:prod              # Build main para producao
npm run build:main:preview           # Build main para preview
npm run build:toolbar:prod           # Build fc-toolbar para producao
npm run build:sidebar:prod           # Build fc-sidebar para producao

# Publicar libs
npm run publish:sidebar              # Publica fc-sidebar no npm
npm run publish:toolbar              # Publica fc-toolbar no npm

# GraphQL
npm run generate                     # Regenera tipos GraphQL (codegen)

# Qualidade
npm run lint                         # ESLint + Prettier check
npm run lint:fix                     # Corrige ESLint + Prettier
npm run prettier                     # Formata apenas HTMLs

# Testes
npm test                             # Testa main (ChromeHeadless)
npm run test-toolbar                 # Testa fc-toolbar
npm run test-sidebar                 # Testa fc-sidebar

# Git
npm run commit                       # Commit interativo (commitizen)
npm run commit-retry                 # Retry ultimo commit
```

## Quando Voce Deve Ser Usado

- Para consultar qualquer aspecto do projeto webapp-2
- Para saber se uma feature, componente ou rota ja existe antes de implementar
- Para entender a estrutura de uma pagina (page module, routing, componentes)
- Para saber qual a forma correta de adicionar uma nova rota/pagina
- Para entender como GraphQL e Apollo Angular sao usados no projeto
- Para saber como regenerar os tipos GraphQL apos mudancas no backend
- Para mapear impacto de mudancas (pipes, guards, interceptors)
- Para entender o sistema de permissoes e guards de rota
- Para saber quais tokens de design usar (cores, espacamentos)
- Para entender como as libs fc-sidebar e fc-toolbar funcionam e sao consumidas
- Para saber como os testes sao escritos e como rodar
- Para entender o pipeline de CI/CD e deploy (S3 + CloudFront)
- Para entender a estrutura de areas de atividade e como adicionar suporte a uma nova area
