---
name: guincho-x-teste-prod
description: "Smoke test do fluxo feliz do GuinchoX em produção via browser (Playwright MCP). Solicita um guincho ponta a ponta com dados de teste predefinidos e confirma que a assistência foi criada com sucesso no Notro. Use após qualquer deploy em produção."
---

# /guincho-x-teste-prod — Smoke test do fluxo feliz em produção

Você executa o **fluxo feliz completo** do GuinchoX em produção via Playwright MCP: abre o site, preenche o wizard com dados de teste predefinidos, paga (ou pelo menos chega ao pagamento), e confirma que a assistência foi criada com sucesso no banco do GuinchoX e integrada ao Notro.

O objetivo é responder a uma pergunta única: **"depois desse último deploy, o caminho feliz de pedir um guincho ainda funciona em produção?"**

## ⚠️ Avisos importantes

Esse procedimento gera efeitos REAIS em produção:

- Cria registro real em `assistances` no banco do GuinchoX
- Pode gerar cobrança Iugu real (PIX ou cartão)
- Quando o pagamento é confirmado, cria service real em `assistences_services` no Notro
- Pode acionar prestador real e enviar WhatsApp pro celular usado no teste

**Antes de começar**:
- Confirme com o usuário até onde levar o teste (Cenário 1, 2 ou 3 abaixo)
- Use sempre os dados de teste padrão da seção [Dados de teste](#dados-de-teste) — não invente
- Se for pagar (Cenário 3), confirme com o usuário antes de submeter o pagamento

## 🛠️ Customização do teste e mudanças no painel Notro (regra crítica)

As instruções desta skill descrevem o teste **default** (caminho feliz padrão). Variações são permitidas e esperadas — cada feature nova exige um cenário próprio (ex: timezone exige agendamento e origem fora de SP, round-trip exige distância > tarifa base, etc.).

**Você PODE alterar o que precisar no painel Notro** (ex: `companies_estimates` do guincho leve, `activity_area` do prestador, criar/desativar prestadores de teste, ajustar planos, etc.) **DESDE QUE**:

1. **Comunique ao usuário ANTES de alterar** — descreva exatamente o que vai mudar, em qual entidade, com quais valores. Aguarde aprovação explícita antes de executar a mudança.
2. **Capture o estado original** — antes de alterar qualquer campo, leia e anote os valores atuais (snapshot, screenshot ou cópia textual). Sem isso, não tem como reverter com confiança.
3. **Reverta TUDO ao estado original ao final do teste** — independente do resultado (sucesso, falha, abortado), antes de encerrar a sessão você restaura cada alteração feita. Isso é **inegociável**: produção precisa ficar exatamente como estava antes do teste.

**Por quê**: dados de produção são compartilhados com clientes reais. Estimates, activity_area, planos e prestadores afetam cotação e roteamento ao vivo. Qualquer alteração não revertida vira regressão silenciosa que aparece em P0 dias depois.

**Checklist de fim de teste** (obrigatório quando houver alteração no Notro):
- [ ] Listei todas as mudanças que fiz no painel Notro durante o teste?
- [ ] Restaurei cada uma para o valor original capturado antes da alteração?
- [ ] Confirmei visualmente no painel que cada campo voltou pro estado anterior?
- [ ] Reportei ao usuário a lista de mudanças feitas + confirmação de reversão?

## Quando usar

- Após qualquer deploy em produção do guinchox-api ou webapp
- Para smoke test pré-release de feature crítica
- Para reproduzir bug que só aparece em prod

## Quando NÃO usar

- Substituir testes E2E automatizados (use specs em `apps/webapp-e2e/` no CI)
- Testar features que ainda não foram deployadas (use ambiente local)
- Sem confirmação explícita do usuário sobre o escopo

## Cenários de execução

O usuário deve escolher um:

### Cenário 1 — Smoke leve (sem cobrança)
Vai até a tela de pagamento e VOLTA, sem submeter. Valida cotação, dados pessoais, veículo, e que a tela de PIX carrega.
- ❌ Não cria assistance no banco
- ❌ Não dispara cobrança Iugu
- ❌ Não integra Notro
- **Custo zero**, ideal pra smoke test diário/CI futuro.

### Cenário 2 — Cobrança gerada, sem pagamento (recomendado pra smoke test após deploy)
Vai até gerar o QR PIX e PARA. Cria a assistance no banco com status `PENDING`, gera invoice Iugu, mas não paga.
- ✅ Cria 1 row em `assistances` (status PENDING — expira sozinha em ~30 min)
- ✅ Cria 1 invoice Iugu (status pending)
- ❌ Não integra Notro (só após pagamento confirmado)
- **Custo zero** (PIX não pago não cobra). Lixo no banco se autolimpa.

### Cenário 3 — Fluxo completo com pagamento
Cria assistance, paga PIX, valida integração Notro completa.
- ✅ Cria 1 row em `assistances`
- ✅ Cria invoice Iugu paga
- ✅ Integra Notro (cria `assistences_services`)
- ✅ Dispara WhatsApp pro telefone usado no teste
- **Custo R$ 100** (PIX). Requer pagar manual via app bancário no celular, copiando código PIX gerado pelo browser.

## Dados de teste

**Sempre use estes dados** (não troque, não invente). Atualize aqui se precisar mudar.

```yaml
pessoa:
  cpf: 423.762.608-33         # CPF do dono do projeto (Marcio). Sempre usar este por padrão.
                              # Override apenas quando o usuário pedir explicitamente um CPF diferente.
                              # Como o CPF já tem cadastro, a API auto-preenche nome/telefone/email
                              # após validar o CPF — não precisa preencher esses campos manualmente.
  nome: Cliente Teste Producao   # ⚠️ FALLBACK: usado SÓ se o backend não auto-preencher
  telefone: 11999998888           # ⚠️ FALLBACK
  email: teste-prod@guinchox.com.br  # ⚠️ FALLBACK

origem:
  endereco_busca: "Av Paulista 1000"
  esperado: "Av. Paulista, 1000 - Bela Vista, São Paulo - SP, 01310-100, Brazil"
  lat: -23.563
  lng: -46.692
  cidade: São Paulo
  uf: SP

destino:
  endereco_busca: "Rua Domingos de Morais 100, Vila Mariana"
  esperado: "R. Domingos de Morais, 100 - Paraíso, São Paulo - SP, 04010-000, Brazil"
  lat: -23.547
  lng: -46.638
  cidade: São Paulo
  uf: SP

veiculo:
  busca: Civic
  selecionar: "Honda / Civic"
  cor: Preto
  placa: ABC1234
```

## URLs

- Webapp: `https://guinchox.com.br`
- API (consulta direta): `https://guinchox-api.notro.io`
- Banco GuinchoX (validação pós-teste): conexão definida em `apps/api/.env` ou Secrets Manager `prod/guinchox/postgresql`

## Procedimento

> Esta seção é preenchida progressivamente conforme o procedimento é executado e validado em prod. A primeira vez que rodar, vá documentando cada passo bem-sucedido.

### Passo 0 — Setup do browser

Use Playwright MCP. Não precisa setup adicional — abra navegação direto.

#### Resetar o formulário no meio do teste

Se durante o teste precisar **começar do zero** (ex: trocar endereços pra outro estado, refazer com outro veículo, descartar dados parciais), o jeito mais simples é clicar na **logo do GuinchoX no header** — isso reseta o wizard inteiro e volta pro Step 1 com todos os campos vazios, sem persistir nada do que tinha sido digitado.

```
mcp__playwright__browser_click
  target: a[href="/"] img, header img[alt*="Logo"]
```

Use isso entre runs do mesmo cenário (ex: validar round-trip com 2 destinos diferentes na sequência) ou quando errar dado de teste e quiser recomeçar limpo.

### Monitoramento contínuo (durante todos os passos)

> **Regra geral**: durante toda a execução do smoke test (tanto no GuinchoX quanto no painel Notro), **monitore proativamente** o console do browser e as requisições de rede. Não espere o final pra checar — qualquer erro fora do padrão deve ser reportado ao usuário no momento em que aparecer.

**Console** — após cada ação importante (clique de Avançar, geração de QR, `pagar()`, abertura de tela), checar erros:

```
mcp__playwright__browser_console_messages
  level: error
```

- **0 errors** é o esperado em todos os passos
- Warnings esperados (ignorar): deprecation do Google Maps (Marker, DirectionsRenderer), avisos de cookie/CSP de terceiros
- Qualquer outro `error` (ex: `TypeError`, `ReferenceError`, falha de fetch, `Uncaught (in promise)`) — reportar ao usuário com a mensagem completa antes de prosseguir

**Network** — após disparar requests críticos (avançar do step de Preço, gerar QR Code, `pagar()`, navegação para detalhes da assistência), inspecionar respostas:

```
mcp__playwright__browser_network_requests
```

Endpoints críticos a monitorar:

| Endpoint | Quando | Esperado |
|---|---|---|
| `POST /public/assistances/draft` | Avançar Step 5 (Preço) | 200/201, response com `id` da assistance |
| `POST /public/assistances/:id/attach-payment` | Gerar QR Code | 200, payload PIX retornado |
| `POST /public/assistances/:id/confirm-payment` | `window.pagar()` | 200 |
| `POST /public/assistances/:id/integrate-notro` | após confirm-payment | 200, sinaliza integração com Notro |
| Chamadas `cliente.notro.io/api/...` | navegar painel Notro | 200/401 (401 só se token expirou) |

Qualquer status **4xx ou 5xx** em endpoints críticos = falha de smoke test, reportar com URL completa, status e response body. Não tentar prosseguir disfarçando o erro.

### Passo 1 — Abrir site de produção

```
mcp__playwright__browser_navigate
  url: https://guinchox.com.br
```

**Validações esperadas no carregamento**:
- `Page Title`: `GuinchoX - Solicite seu guincho agora`
- 0 erros no console (warnings do Google Maps deprecation são esperados — Marker e DirectionsRenderer)
- Wizard visível com inputs `Local de partida *` e `Local de destino *`
- Botão `Avançar` aparece como `[disabled]` (esperado — campos vazios)

**Como capturar o estado**:
```
mcp__playwright__browser_snapshot
```
Confirme que o snapshot mostra:
- combobox `"Local de partida *"`
- combobox `"Local de destino *"`
- button `"Avançar"` com atributo `[disabled]`

Se a página não carregou ou o wizard não está visível, pare e investigue antes de avançar.

### Passo 2 — Step de Localização (origem + destino + modo de serviço)

#### Regras de comportamento

**Modo de serviço (Preciso agora vs Agendar)**:
- **Padrão**: usar **"Preciso agora"** (imediato). Já vem selecionado por default na UI, então normalmente não precisa fazer nada.
- **Override**: só usar **"Agendar"** quando o usuário pedir explicitamente (ex: "testar agendado pra amanhã 09:00", "testar agendamento no AC pra 15:00"). Nesse caso clica no botão `Agendar` e preencha a data/hora pedida.

**Endereços (origem e destino)**:
- **Padrão**: usar os endereços de SP da seção [Dados de teste](#dados-de-teste) (Av. Paulista 1000 → R. Domingos de Morais 100).
- **Override**: se o usuário pedir teste com endereço de outro estado/cidade (ex: "testar com origem em Manaus"), substitua os 2 endereços conforme a instrução, mantendo o formato `<rua nome+número, bairro, cidade>`.

#### Como preencher o input (combobox com autocomplete)

A UI usa autocomplete que dispara o Google Geocoding após debounce de 800ms. O método mais robusto:

```
mcp__playwright__browser_type
  target: #origin
  text: "Av Paulista 1000"
  slowly: true
```

Aguarde debounce, depois selecione a sugestão:
```
mcp__playwright__browser_press_key
  key: ArrowDown
mcp__playwright__browser_press_key
  key: Enter
```

⚠️ **Importante**: `Enter` direto NÃO seleciona — o input fica com o texto digitado em vez do endereço completo. **Sempre use `ArrowDown` antes do `Enter`** pra navegar até o item da lista.

Confirme que o input ficou com o endereço completo formatado:
```
mcp__playwright__browser_evaluate
  function: () => document.querySelector('#origin')?.value
```
Esperado: algo como `"Av. Paulista, 1000 - Bela Vista, São Paulo - SP, 01310-100, Brazil"` — se ficou só o texto digitado, repita o `ArrowDown` + `Enter`.

Repita pra `#destination`.

#### Validações

Após preencher os 2 inputs:
- O texto do input deve mostrar o endereço completo formatado (ex: `Av. Paulista, 1000 - Bela Vista, São Paulo - SP, 01310-100, Brazil`).
- O mapa deve renderizar o trajeto (linha entre os 2 pontos, com label da distância carregada — ex: "3.3 km").
- O card "Tempo estimado de chegada ao local de partida" deve aparecer abaixo dos inputs (vem do endpoint `/public/estimates/arrival`, normalmente "40 minutos" se a Regi não cadastrou estimate específica).
- O botão `Avançar` fica enabled.

#### Avançar

```
mcp__playwright__browser_click
  target: button:has-text("Avançar"):not([disabled])
```

### Passo 3 — Step de Dados Pessoais (CPF + nome/telefone/email)

#### Comportamento da UI

Esse step aparece em **2 sub-passos**:

1. **Sub-passo 3a**: só o campo `CPF/CNPJ` é exibido. Outros campos só aparecem após avançar com CPF preenchido.
2. **Sub-passo 3b**: backend valida o CPF via `GET /api/users/document/<cpf>`:
   - **Se CPF já cadastrado**: backend devolve 200 com os dados, frontend **auto-preenche** Nome / Telefone / E-mail e desabilita o CPF (vira `disabled: true`).
   - **Se CPF novo**: campos vêm vazios e precisam ser preenchidos manualmente com os fallbacks da seção [Dados de teste](#dados-de-teste).

#### Como executar

```
mcp__playwright__browser_type
  target: input[placeholder*="CPF"]
  text: 42376260833      # CPF padrão (ver Dados de teste). Sem pontos/traços — máscara aplica sozinha.
```

Avançar pra disparar a validação do CPF e expor os outros campos:
```
mcp__playwright__browser_click
  target: button:has-text("Avançar"):not([disabled])
```

Verificar se auto-preencheu:
```
mcp__playwright__browser_evaluate
  function: () => Array.from(document.querySelectorAll('input')).map(i => ({ placeholder: i.placeholder, value: i.value, disabled: i.disabled }))
```

**Se nome/telefone/email vieram preenchidos**: nada a fazer, basta clicar em **Avançar** novamente pra ir pro próximo step.

**Se vieram vazios**: preencher cada um com os valores da seção [Dados de teste](#dados-de-teste) (Cliente Teste Producao / 11999998888 / teste-prod@guinchox.com.br) e avançar.

#### Validações

- CPF formatado com máscara (`423.762.608-33`)
- CPF com `disabled: true` após reconhecimento
- 4 inputs visíveis no DOM: CPF, Nome, Telefone, E-mail
- Botão `Avançar` enabled quando todos os 3 campos editáveis estão preenchidos

### Passo 4 — Step de Veículo (modelo + cor + placa)

#### Veículo (combobox com autocomplete)

Mesmo padrão do step de localização: digite + `ArrowDown` + `Enter`.

```
mcp__playwright__browser_type
  target: input[placeholder*="tipo, marca"]
  text: Civic
  slowly: true

mcp__playwright__browser_press_key
  key: ArrowDown

mcp__playwright__browser_press_key
  key: Enter
```

Esperado: input vira `"Honda / Civic"` (formato `<Marca> / <Modelo>`).

#### Cor (dropdown nativo de opções fixas)

Diferente do veículo, a cor é um dropdown com lista pré-definida (Amarelo, Azul, Bege, Branco, Cinza, Dourado, Laranja, Marrom, Prata, Preto, Rosa, Roxo, Verde, Vermelho, Vinho, Outra). Clicar abre a lista; clicar na opção fecha.

```
mcp__playwright__browser_click
  target: input[placeholder*="cor"]

mcp__playwright__browser_click
  target: [role="option"]:has-text("Preto")
```

#### Placa (input com máscara)

```
mcp__playwright__browser_type
  target: input[placeholder*="placa"]
  text: ABC1234
```

Máscara aplica automaticamente o hífen → `ABC-1234`.

#### Validações

```
mcp__playwright__browser_evaluate
  function: () => Array.from(document.querySelectorAll('input')).map(i => ({ placeholder: i.placeholder, value: i.value }))
```

Esperado:
- `Honda / Civic`
- `Preto`
- `ABC-1234`

Avançar:
```
mcp__playwright__browser_click
  target: button:has-text("Avançar"):not([disabled])
```

### Passo 5 — Step de Preço

#### Comportamento da UI

Após avançar do step de Veículo, o wizard exibe a tela de confirmação de preço (read-only — não há campos de input). Os dados são calculados pela `public-api` do Notro a partir do veículo + origem/destino:

- **Header**: paragraph "Confirme os detalhes da solicitação de guincho e prossiga para o pagamento."
- **Card de total**: `R$ <valor>` em destaque + label "A pagar" (ex: `R$ 100,00`)
- **Resumo do trajeto**: origem, destino, "Distância <X>km", "Estimativa de chegada <Y> minutos"
- **Detalhamento** (vem aberto por padrão, com botão "Esconder detalhes"):
  - `Tarifa base (até 40km)`: valor base
  - `KM adicional (acima de 40km)`: valor extra (R$ 0,00 quando trajeto está dentro da tarifa base)
  - `Total`: soma final
- **Botões**: `Voltar`, `Avançar`

#### Validações

```
mcp__playwright__browser_snapshot
```

Confirme que aparece:
- Algum valor monetário no formato `R$ X,XX` no card de total
- Os endereços de origem e destino exibidos corretamente
- Distância em km e estimativa de chegada em minutos
- Linha de "Total" dentro do detalhamento

Para o cenário padrão (Av. Paulista 1000 → R. Domingos de Morais 100, Honda Civic), o valor esperado é **R$ 100,00** com **11km** e **30 minutos** de estimativa.

#### Avançar

```
mcp__playwright__browser_click
  target: button:has-text("Avançar"):not([disabled])
```

⚠️ **Atenção**: ao clicar Avançar aqui, o frontend chama `POST /public/assistances/draft` no backend, criando uma row em `assistances` com status `PENDING`. **Esse é o primeiro passo destrutivo** — não há custo, mas há registro no banco. Confirme com o usuário antes de avançar nos cenários sensíveis.

### Passo 6 — Step de Pagamento (selecionar PIX + gerar QR + bypass)

#### Comportamento da UI

Após avançar do step de Preço, o wizard exibe a tela de Pagamento com 2 opções:
- `Cartão` (selecionado por padrão) — exibe formulário de cartão (número, validade, CVV, nome, CPF/CNPJ)
- `Pix` — exibe apenas a explicação "Ao prosseguir um QR Code será gerado para a realização do pagamento"

Para **99% dos testes** (smoke test default), use **PIX + bypass `pagar()`** — não pague de verdade. Override apenas se o usuário pedir explicitamente teste de cartão ou pagamento real.

#### Selecionar PIX

```
mcp__playwright__browser_click
  target: button:has-text("Pix")
```

Confirme: o botão `Pix` fica `[active]`, formulário de cartão some, aparece "Gerar QR Code" no lugar do "Pagar e solicitar".

#### Gerar QR Code

```
mcp__playwright__browser_click
  target: button:has-text("Gerar QR Code")
```

Aguarde ~3 segundos. O frontend dispara em sequência:
1. `POST /public/assistances/draft` → cria row em `assistances` com status `PENDING`
2. `POST /public/assistances/:id/attach-payment` → cria invoice na Iugu

Tela esperada após geração:
- Texto: "Escaneie o QR Code no seu aplicativo bancário ou utilize a opção Pix Copia e Cola"
- Timer "O tempo para você pagar acaba em **04:50**" (5 min, decrementa em tempo real)
- `<img>` com QR Code Pix renderizado
- Payload PIX (BR Code) visível como texto, prefixo `00020101...br.gov.bcb.pix...qr.iugu.com/public/payload/v2/cobv/...` + botão `Copiar`

#### Bypass do pagamento — `window.pagar()`

> ⚠️ **Esse bypass existe apenas em produção do GuinchoX como conveniência pra testes** — pula a confirmação real de pagamento na Iugu e dispara direto o `confirm-payment` + `integrate-notro`. Custo zero, abre assistência real no Notro.

```
mcp__playwright__browser_evaluate
  function: () => { const result = window.pagar?.(); return { hasFunction: typeof window.pagar, result: String(result) }; }
```

Esperado: `{ hasFunction: "function", result: "undefined" }` (a função existe e foi chamada com sucesso — `undefined` é retorno normal).

Aguarde ~5 segundos pro backend processar `confirm-payment` + `integrate-notro` (cria service real em `assistences_services` no Notro).

```
mcp__playwright__browser_wait_for
  time: 5
```

### Passo 7 — Validação de sucesso (tela final)

Após o `pagar()`, o wizard avança automaticamente pra tela de confirmação:

```
mcp__playwright__browser_snapshot
```

Validações esperadas:
- **`<h1>` `"Guincho solicitado!"`** — título de sucesso
- Paragraph: `"Fique atento! A empresa responsável por te atender entrará em contato."`
- Card de valor: `R$ 100,00` com label `"Pago"` (não mais "A pagar")
- Endereços de origem e destino exibidos
- Botão `"Ver detalhes"` (expande resumo)
- Card "Chegada estimada em **40 minutos**" (vem do endpoint `/public/estimates/arrival` — pode variar por cidade/região)

Se chegou nessa tela com `R$ <valor> Pago` e `Guincho solicitado!`, o **smoke test passou**: o GuinchoX criou a assistance no banco, gerou invoice Iugu, processou o confirm-payment e integrou ao Notro com sucesso.

#### Variantes de override (não-default)

- **Pagar PIX de verdade (Cenário 3)**: pular a chamada `pagar()` e copiar o payload BR Code → pagar via app bancário no celular. O frontend faz polling no status da invoice e avança sozinho quando a Iugu confirma. Use só quando explicitamente pedido (R$ 100 reais por teste).
- **Cartão (raríssimo)**: deixar Cartão selecionado, preencher os 4 campos do formulário, clicar `Pagar e solicitar`. CPF do titular já vem pré-preenchido se foi reconhecido no Step 3. **Custo R$ 100 reais**.

## Validação pós-teste

A tela "Guincho solicitado!" + `R$ <valor> Pago` no GuinchoX é o sinal de sucesso do **lado GuinchoX**, mas o smoke test só está completo quando confirmar que a assistência foi **integrada corretamente no Notro**. Essa segunda parte é feita no painel da Notro (webapp-2).

### Passo 8 — Login no painel Notro

Abrir nova aba pra não perder o estado da aba do GuinchoX (útil pra comparar campos lado a lado se algo der errado):

```
mcp__playwright__browser_tabs
  action: new
  url: https://cliente.notro.io
```

Login em 2 sub-passos (UI separa email + senha):

```
mcp__playwright__browser_type
  target: input[placeholder="Digite aqui"]
  text: marcio+guinchox@notro.io

mcp__playwright__browser_click
  target: button:has-text("Continuar"):not([disabled])
```

Após confirmar o email aparece "👋 Seja bem-vindo(a) novamente, Marcio!" e o input de senha:

```
mcp__playwright__browser_type
  target: input[type="password"]
  text: <senha do Marcio — armazenada na memória reference_notro_painel_login.md>

mcp__playwright__browser_click
  target: button:has-text("Continuar"):not([disabled])
```

⚠️ **Credenciais**: usar sempre as do Marcio salvas em `~/.claude/projects/-Users-marciofmjr-dev-guinchox/memory/reference_notro_painel_login.md`. O usuário tem acesso ao tenant `guinchox` no Notro com permissões de operação.

Esperado: redirect pra `https://cliente.notro.io/painel/#/auto/assistencias` (módulo AUTO selecionado por padrão, listagem de assistências do tenant Guincho X).

### Passo 9 — Fechar modal de notificações + abrir assistência

Sempre que aparecer o dialog "Ativar notificações" (chato, aparece com frequência), clicar em **NÃO** pra fechar:

```
mcp__playwright__browser_click
  target: button:has-text("NÃO")
```

A listagem mostra as últimas assistências do tenant. A primeira linha (mais recente) deve ser a do teste que acabou de rodar — comparar `Aberto em` com a hora atual e `Cliente: Marcio` (ou o nome do CPF de teste). Anote o número da assistência (ex: `00000046`).

Para abrir os detalhes, clicar no botão de ações da linha (ícone na coluna `Ações`, à direita do `Status`):

```
mcp__playwright__browser_click
  target: tr:has(td:has-text("<numero da assistencia>")) td:last-child button
```

Se o seletor não pegar pelo ref do snapshot, alternativa: clicar via Playwright pelo `ref` capturado no snapshot.

### Passo 10 — Validar aba Geral

A tela de detalhes abre na aba **Geral** por padrão. URL fica `cliente.notro.io/painel/#/auto/assistencias/<uuid>`.

Validações **obrigatórias** (campos disabled — só leitura):

- **Apólice (combobox de busca)**: `Nome: <Marcio> - CPF: <CPF do teste> | Placa: <placa> | Modelo: <modelo da apólice no Notro> | Plano: Automóvel`
- **Número** + **Apólice**: mesmo número da listagem (ex: `00000046`)
- **Contratante**: `Guincho X`
- **Plano**: `automovel-guincho-x - Automóvel`
- **Data/Hora**: hora aproximada da solicitação no GuinchoX (ex: `29/04/2026, 11:39:10`)
- **Status**: `Aberto`
- **Solicitante**: nome do cadastro
- **Celular 1** / **Celular 2**: telefone do cadastro
- **Problema**: `Pane`
- **Serviço**: `Reboque Leve` (ou correspondente ao tipo solicitado)
- **Descrição**: `Criado via GuinchoX`
- **Endereço de origem**: deve corresponder ao digitado no GuinchoX (ex: `Avenida Paulista, 1000, São Paulo - São Paulo, 01310-100`)
- **Endereço de destino**: idem (ex: `Rua Domingos de Morais, 100, São Paulo - São Paulo, 04010-000`)
- **Horário do serviço**: `Imediato` (radio checked) — ou `Agendado` se foi feito o override de "Agendar" no Step 2

#### ⚠️ Comportamento esperado: veículo pode divergir do que foi digitado no GuinchoX

Os campos **Marca / Carro / Cor** podem mostrar valores **diferentes** dos que foram digitados no Step de Veículo do GuinchoX (ex: GuinchoX recebeu `Honda / Civic / Preto` mas Notro mostra `Nissan / March / Bege`).

**Causa**: quando o CPF + Placa enviados pelo GuinchoX já casam com uma apólice existente no Notro, o backend reaproveita a apólice cadastrada e ignora os dados de veículo enviados pela integração. Isso é comportamento de produto (apólice é fonte canônica), não regressão. Não falhar o smoke test por isso — apenas registrar.

**Como diferenciar bug de comportamento esperado**: se o CPF é novo (primeira vez integrando no Notro), o veículo deve refletir o que foi digitado. Se o CPF já tem apólice prévia, espera-se divergência.

### Passo 11 — Validar aba Serviços

Clicar na aba **Serviços** (mostra contador `(1)` se vier 1 serviço associado):

```
mcp__playwright__browser_click
  target: [role="tab"]:has-text("Serviços")
```

A tabela deve ter exatamente **1 linha**:

| Campo | Valor esperado |
|---|---|
| Número | `<numero-assistencia>/1` (ex: `00000046/1`) |
| Problema | `Pane` |
| Serviço | `Reboque Leve` |
| Cidade / Estado | corresponde ao destino |
| Prestador | _vazio_ (ainda não foi alocado prestador) |
| Status | `Fila Manual` |
| Criado por | **`Integração GuinchoX`** ← marca canônica do fluxo ponta-a-ponta |

Se o `Criado por` for `Integração GuinchoX`, é a confirmação definitiva de que a integração rodou: a public-api recebeu o payload e o server criou o `assistences_services` corretamente.

### Passo 12 — Validação de WhatsApp (manual, conduzida pelo usuário)

> 🔒 **Não automatizado**: o WhatsApp do Marcio é pessoal — **não tente abrir, navegar ou inspecionar o WhatsApp Web**. Essa validação é feita manualmente pelo usuário; você só pergunta e ele responde se chegou ou não.

Ao longo do ciclo de vida da assistência, o ecossistema Notro envia mensagens via WhatsApp (Twilio) pro celular cadastrado. As esperadas são:

| Evento | Quando dispara | O que valida |
|---|---|---|
| **Abertura** | Logo após `integrate-notro` criar a assistance no Notro (ainda em `Fila Manual`) | A integração GuinchoX → Notro disparou o evento de abertura corretamente |
| **Acionamento** | Quando a Regi (ou alocação automática) atribui um prestador e muda o status para `Acionado` | Notificator detectou o evento e roteou pro tenant correto |
| **Finalização (NPS)** | Quando o serviço é concluído (status `Finalizado`) | Pesquisa de NPS chega com link de avaliação |

#### Como validar

Quando precisar saber se uma mensagem chegou:

1. **Pergunte ao usuário** — algo como: _"acabei de rodar o teste, chegou WhatsApp de abertura no seu celular?"_
2. Aguarde a resposta. Não tente abrir WhatsApp Web nem qualquer interface de mensageria.
3. Se o usuário confirmar que chegou, registre como ✅ no histórico. Se não chegou, anote como possível regressão e trate como item de investigação (notificator pode estar com problema, Twilio pode estar com bloqueio do número, etc.).

A mensagem de **abertura** é a mais importante pro smoke test — confirma que o pipeline `Notro → notifications (notificator) → Twilio` rodou ponta-a-ponta. As outras dependem de fluxo manual posterior (alocar prestador, finalizar serviço) e geralmente não são exigidas no smoke test default.

### Passo 13 — Direcionar e acionar a assistência no painel Notro

Depois de validar a aba Geral e a aba Serviços (Passos 10 e 11), o smoke test pode opcionalmente continuar pra simular o **acionamento ponta-a-ponta** — atribuir a assistência a um prestador real, levá-la pelo HUB até a finalização. Esse passo só roda quando o usuário pedir explicitamente o teste estendido.

#### 13.1 — Capturar o número da assistência

> ⚠️ **Número é dinâmico**: cada execução do smoke gera um número novo (`00000046`, `00000047`, ...). Capture o número da assistência que **você acabou de criar** (cruze com a hora de abertura no painel) — não use número hardcoded da skill.

Estando em `cliente.notro.io/painel/#/auto/assistencias` (listagem), confirmar pela coluna `Aberto em` qual linha é a sua execução. Anotar o número (ex: `00000046`) e o `id` UUID da URL dos detalhes (`/auto/assistencias/<uuid>`).

#### 13.2 — Ir em Operações > Acionamentos

Navegar direto via URL:

```
mcp__playwright__browser_navigate
  url: https://cliente.notro.io/painel/#/auto/acionamentos
```

A listagem mostra todas as assistências do tenant Guincho X que estão em `Fila Manual`, ordenadas por mais antiga. A nossa provavelmente está nas últimas páginas — usar filtro pra encontrar.

#### 13.3 — Filtrar por número

Clicar no botão de filtro (ícone `filter_alt_outline` no canto superior direito):

```
mcp__playwright__browser_click
  target: button:has(img:text("filter_alt_outline"))
  # se o seletor falhar, capturar snapshot e usar o ref direto
```

Painel lateral abre. Preencher o campo `Filtrar por número` com o número da assistência e dar Enter:

```
mcp__playwright__browser_type
  target: input[aria-label="Filtrar por número"]
  text: <numero-da-assistencia>
  submit: true
```

A listagem fica com 1 linha só. **Antes de clicar no Acionar, fechar o painel de filtros** (ele fica sobreposto à tabela e bloqueia cliques):

```
mcp__playwright__browser_press_key
  key: Escape
```

#### 13.4 — Clicar em Acionar

```
mcp__playwright__browser_click
  target: button:has-text("Acionar")
```

URL muda pra `/auto/acionamentos/<uuid>` — entra no wizard de acionamento com 4 abas: **Dados da assistência → Prestador → Rotas → Valores e acionamento**.

#### 13.5 — Aba 1: Dados da assistência (read-only)

Conferir que os dados batem com a assistência aberta no GuinchoX (número, contratante Guincho X, plano Automóvel, problema Pane, serviço Reboque Leve, endereços, descrição "Criado via GuinchoX"). Todos os campos são read-only nesta etapa. Clicar em **Próximo**.

#### 13.6 — Aba 2: Prestador

Lista os prestadores disponíveis pro tenant. Cada linha tem 2 botões na coluna de ações:
- `close` (vermelho) — recusar
- `check` (azul) — selecionar este prestador

**Default**: clicar no `check` da linha do **Notro Assistance** (é o prestador padrão de teste, baseado em Barueri/SP).

```
mcp__playwright__browser_evaluate
  function: () => {
    const row = Array.from(document.querySelectorAll('table tr')).find(r => r.innerText.includes('Notro Assistance'));
    const btn = Array.from(row.querySelectorAll('button')).find(b => b.innerText.trim() === 'check');
    btn.click();
  }
```

Clicar em **Próximo**.

#### 13.7 — Aba 3: Rotas

Mostra o trajeto completo (base do prestador → origem → destino → volta pra base) num mapa Leaflet. **Não precisa interagir** — só conferir que o trajeto faz sentido. Clicar em **Próximo**.

#### 13.8 — Aba 4: Valores e acionamento

2 inputs obrigatórios:

| Campo | formcontrolname | Valor padrão |
|---|---|---|
| Previsão (min) | `estimatedTime` | `30` |
| Contato responsável | `responsibleContactName` | `Marcio` |

```
mcp__playwright__browser_type
  target: input[formcontrolname="estimatedTime"]
  text: 30

mcp__playwright__browser_type
  target: input[formcontrolname="responsibleContactName"]
  text: Marcio
```

Botão final é **Acionar** (não mais "Próximo").

```
mcp__playwright__browser_click
  target: button:has-text("Acionar")
```

#### 13.9 — Modal de modo de acionamento (Manual vs Semi-automático)

Aparecem 2 opções:

| Modo | Comportamento |
|---|---|
| **Manual** (default) | Inicia o serviço automaticamente para o prestador, sem ele precisar aceitar. Encurta o smoke test. |
| **Semi-automático** | Cai na fila do prestador; ele precisa aceitar manualmente no HUB antes de avançar. |

Clicar em **Manual** (opção default na maioria dos testes).

```
mcp__playwright__browser_click
  target: button:has-text("Manual"), [role="button"]:has-text("Manual")
```

Aparece popup de confirmação com `Cancelar` / `Acionar`. Clicar no `Acionar` do popup (cuidado: há 2 botões "Acionar" no DOM neste momento — o do wizard e o do popup; usar o do popup, que está dentro de um container que também tem `Cancelar`):

```
mcp__playwright__browser_evaluate
  function: () => {
    const acionarBtns = Array.from(document.querySelectorAll('button'))
      .filter(b => b.offsetParent !== null && b.innerText.trim() === 'Acionar');
    for (const btn of acionarBtns) {
      let parent = btn.parentElement;
      while (parent) {
        if ((parent.innerText || '').includes('Cancelar') && parent.innerText.length < 500) {
          btn.click();
          return;
        }
        parent = parent.parentElement;
      }
    }
  }
```

#### 13.10 — Validar o acionamento

Após confirmar, a URL volta pra `/auto/acionamentos` (listagem). A assistência **sai dessa listagem** porque mudou de status (`Fila Manual` → `Aguardando deslocamento` / atribuída ao prestador). Esse é o sinal de sucesso do acionamento.

Validações esperadas:
- ✅ URL retornou pra `cliente.notro.io/painel/#/auto/acionamentos`
- ✅ Listagem **não contém mais** a assistência (ela saiu da fila manual)
- ✅ Sem toast de erro

#### 13.11 — Notificações esperadas (validação manual via WhatsApp/SMS)

> 🔒 **Não automatizado**: pergunte ao usuário se chegaram. Não tente abrir WhatsApp/SMS.

Após o acionamento Manual, o pipeline `Notro → notifications → Twilio` dispara 2 mensagens pro celular cadastrado:

**WhatsApp**:
```
Olá, Marcio, o serviço de Reboque Leve foi acionado com previsão de chegada de 30 minutos. ✅

Guincho X
```

**SMS**:
```
Ola Marcio, o servico de Reboque Leve foi acionado com previsao de chegada de 30 minutos.
Guincho X
```

Pergunte ao usuário: _"acabei de acionar a assistência, chegou WhatsApp + SMS de acionamento?"_

### Passo 14 — Login no HUB e abrir Acompanhamento

> 🟢 Esse passo só roda quando o usuário pedir validação ponta-a-ponta no HUB (executar serviço como prestador). Caso contrário, parar no Passo 13.

```
mcp__playwright__browser_tabs
  action: new
  url: https://hub.notro.io/
```

Login (mesmo padrão de 2 sub-passos: email → senha) com as credenciais do prestador de teste salvas em `reference_notro_hub_login.md` (`prestador.re@notro.io` / senha em memória). Após confirmar a senha, aparece dialog "Ativar notificações" → clicar **Cancelar**.

URL final: `hub.notro.io/#/atividades`. Dashboard de KPIs em iframe + tabela "Acompanhamento" abaixo.

⚠️ **Erros de console esperados** ao logar no HUB (não bloqueiam o smoke):
- `403` em `/assets/logos/logo_.png` (asset de branding cosmético)
- `403` em `o4504532419543040.ingest.us.sentry.io/api/.../envelope` (telemetria Sentry, ad-blocker)
- `ApolloError: You must be signed in to view this resource` originando em `dashboards.notro.io` (widget de dashboards cross-origin sem auth — não impacta o fluxo)

### Passo 15 — Filtrar pela assistência no HUB

A tabela de "Acompanhamento" vem com filtros default que podem ocultar a assistência. Abrir o filtro pelo botão `filter_alt_outline` no canto superior direito da seção "Acompanhamento" (NÃO o botão "Filtros" do iframe de Dashboards acima).

No painel lateral de filtros, preencher **Filtrar pelo Número do serviço** com o número da assistência e dar Enter:

```
mcp__playwright__browser_type
  target: input[aria-label="Filtrar pelo Número do serviço"]
  # use ref do snapshot se o aria-label não pegar
  text: <numero-da-assistencia>
  submit: true
```

Fechar o painel:
```
mcp__playwright__browser_press_key
  key: Escape
```

A linha aparece na aba **Aguardando deslocamento** (status inicial logo após o acionamento Manual no Notro).

### Passo 16 — Vincular veículo com GPS (opcional, recomendado)

Abrir o menu "Mais ações" da linha → clicar **Detalhes**. Na modal, descer até a seção **VINCULAR VIATURA GPS, COLABORADOR OU APLICATIVO** e abrir o select **Veículo**.

> ⚠️ **Priorize veículos com `Último sinal: <data>`** — esses têm GPS ativo e disparam o link de tracking em tempo real pro WhatsApp do cliente. Veículos com `Sem sinal` não têm sinal GPS no momento e devem ser evitados (a menos que seja o único disponível ou o usuário pedir explicitamente).

Exemplo de opções:
- ✅ `Volkswagen - EFU1F90 - 24.13 km - Último sinal: 29/04/2026 14:24` (com GPS ativo)
- ❌ `City - VJD4302 - (Carro Desativado) Sem sinal`
- ❌ `ARGOS - 3721891 - Sem sinal`

Selecionar o veículo com sinal e clicar **SALVAR** no rodapé da modal. Modal fecha.

Caso não haja nenhum veículo com sinal, **pular esse passo** — o smoke ainda passa, só não envia link de tracking.

### Passo 17 — Iniciar deslocamento

Abrir Mais ações → clicar **Iniciar deslocamento**. Aparece dialog: _"Deseja iniciar o deslocamento? Se o serviço estiver vinculado a um prestador que possua o aplicativo ou um veículo com GPS, o cliente receberá a localização do prestador."_ → clicar **Iniciar**.

Status na tabela muda de **Aguardando deslocamento** → **A caminho do local**. A coluna Colaborador agora mostra a placa do veículo vinculado (ex: `EFU1F90`).

### Passo 18 — Confirmar Chegada

Abrir Mais ações novamente → agora a opção **Chegada** substitui "Iniciar deslocamento". Clicar.

Aparece **popup de segurança** "Confirmar código" mostrando os primeiros caracteres da placa mascarados (ex: `ABC-12`) e pedindo os **2 últimos dígitos** em campos separados (`firstDigit`, `secondDigit`):

```
mcp__playwright__browser_type
  target: input[formcontrolname="firstDigit"]
  text: <antepenúltimo dígito da placa>

mcp__playwright__browser_type
  target: input[formcontrolname="secondDigit"]
  text: <último dígito da placa>
```

Para placa `ABC-1234` → digitar `3` e `4`. Clicar **Confirmar**.

Status muda pra **Em serviço** (contador da aba "Em serviço" sobe).

### Passo 19 — Concluir o serviço

Abrir Mais ações → clicar **Concluir**. Modal "Concluir serviço" abre com:
- Select **Tipo de conclusão** (4 opções)
- Textarea **Descrição** (obrigatória)

Opções de tipo de conclusão:
| Opção | Quando usar |
|---|---|
| **Serviço realizado com sucesso** | ✅ Default do smoke test — fluxo feliz |
| Cliente ausente do local | Cliente não estava no endereço |
| Serviço não realizado com deslocamento iniciado | Deslocamento feito mas não foi possível executar |
| Recusado pela oficina/concessionária | Oficina/concessionária recusou |

Selecionar **Serviço realizado com sucesso**. Preencher Descrição com qualquer texto (ex: `Smoke test concluido com sucesso via skill /guincho-x-teste-prod`). Clicar **Concluir**.

A modal fecha e a linha **sai de todas as abas** (todos os contadores ficam em 0 dentro do filtro aplicado). Esse é o sinal final de sucesso.

### Critério de sucesso ponta-a-ponta (HUB)

- ✅ Status passa por: `Fila Manual` → `Aguardando deslocamento` → `A caminho do local` → `Em serviço` → finalizado
- ✅ Após Concluir, a assistência some das tabs do Acompanhamento (todos contadores zerados pra o filtro)
- ✅ (opcional) WhatsApp/SMS adicional pode ser enviado em cada transição (perguntar ao usuário se chegou)

### Critério final de sucesso

O smoke test passa quando **todas** estas condições são verdadeiras:

- ✅ Tela "Guincho solicitado!" + `R$ <valor> Pago` no GuinchoX
- ✅ Listagem de assistências do tenant Guincho X mostra a nova linha como mais recente
- ✅ Aba Geral com os campos canônicos preenchidos (status `Aberto`, contratante `Guincho X`, descrição `Criado via GuinchoX`, endereços corretos)
- ✅ Aba Serviços com 1 linha de `Reboque Leve` em status `Fila Manual` criada pela `Integração GuinchoX`
- ✅ (opcional/manual) WhatsApp de abertura chegou no celular cadastrado — confirmar com o usuário

Se qualquer um falhar, abrir investigação (verificar logs do guinchox-api no Elastic Beanstalk, status da public-api, tabela `assistances` do banco do GuinchoX, e logs do notificator no caso de falha de WhatsApp).

### Validações opcionais (apenas se houver suspeita)

- **Console** (ambas as abas): 0 errors (warnings do Google Maps deprecation são esperados)
- **Network tab**: `POST /public/assistances/draft` e `POST /public/assistances/:id/attach-payment` retornaram 200/201
- **Banco GuinchoX** (`apps/api/.env` ou Secrets Manager `prod/guinchox/postgresql`):
  ```sql
  SELECT id, status, integration_status, created_at
  FROM assistances
  ORDER BY created_at DESC LIMIT 1;
  -- Esperado: status='DONE' (ou 'PAYMENT_CONFIRMED' se integrate-notro estiver lento), integration_status indicando sucesso
  ```

## Cleanup / Rollback

> **Não é necessário fazer cleanup** dos testes rodados com bypass `pagar()`. As contas usadas (CPF do Marcio + tenant `Guincho X`) são de teste — pode rodar o smoke quantas vezes quiser sem se preocupar em limpar banco, cancelar assistência ou reembolsar nada.

Exceções que ainda exigem atenção:

- **Cenário 3 (PIX pago de verdade)**: cobrança real R$ 100. Cancelar assistance via UI e reembolsar via painel da Iugu se for engano. Use só quando explicitamente pedido.
- **Cenário com cartão real**: idem, cobrança real — só com pedido explícito.

