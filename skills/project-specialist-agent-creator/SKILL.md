---
name: project-specialist-agent-creator
description: Cria ou atualiza a skill `project-specialist-in-{projeto}` para projetos em `/Users/marciofmjr/dev`. Use quando precisar gerar um especialista profundo de uma codebase (arquitetura, stack, features, endpoints, padroes, testes e deploy) ou manter esse especialista sincronizado com mudancas recentes no projeto.
---

# Project Specialist Agent Creator

## Objetivo

Atuar como meta-agente para criar ou atualizar especialistas de projeto no formato de skill do Codex.

Gerar artefatos uteis para consulta tecnica recorrente, evitando analise repetida da mesma codebase.

## Entrada Obrigatoria

Receber o nome da pasta do projeto em `/Users/marciofmjr/dev`.

- Exemplo: `guinchox`
- Caminho esperado: `/Users/marciofmjr/dev/guinchox`

Se o nome nao for informado, solicitar ao usuario antes de continuar.

Se o caminho do projeto nao existir, interromper e informar erro de projeto nao encontrado.

## Nomenclatura e Saida

Usar sempre o nome de skill `project-specialist-in-<projeto>`.

Criar ou atualizar:

- `/Users/marciofmjr/dev/ai-agents/skills/project-specialist-in-<projeto>/SKILL.md`
- `/Users/marciofmjr/dev/ai-agents/skills/project-specialist-in-<projeto>/agents/openai.yaml`
- `/Users/marciofmjr/dev/ai-agents/skills/SKILLS_INVENTORY.md`

Atualizar o inventario em ordem alfabetica e ajustar o contador `Total`.

## Workflow Obrigatorio de Exploracao

1. Estrutura geral:
- Listar arvore de diretorios (1-2 niveis).
- Ler arquivos raiz de configuracao (`package.json`, `nx.json`, `tsconfig*`, `angular.json`, `Dockerfile`, `docker-compose*`, etc.).
- Identificar stack completa.

2. Arquitetura e modulos:
- Mapear apps/libs e dominios principais.
- Identificar padrao arquitetural e camadas.
- Mapear fronteiras de modulo e contratos entre camadas.

3. Features e fluxos:
- Ler arquivos centrais de cada dominio.
- Documentar fluxo de dados end-to-end.
- Mapear endpoints de API e rotas frontend, quando houver.
- Identificar integracoes externas, jobs, filas e crons.

4. Dados e modelo:
- Ler schema e migrations.
- Mapear entidades, relacionamentos, enums e tipos relevantes.

5. Padroes e convencoes:
- Mapear padroes de codigo recorrentes.
- Mapear estrategia de testes.
- Mapear convencoes de nomenclatura, lint, formatacao e CI/CD.

6. Infra e deploy:
- Mapear ambientes, pipelines e estrategia de deploy.
- Listar variaveis de ambiente essenciais.

Usar subagentes de exploracao em paralelo quando a base for grande.

## Conteudo Minimo do Especialista Gerado

Escrever `project-specialist-in-<projeto>/SKILL.md` com:

- Visao geral do projeto e proposito.
- Stack tecnologico.
- Estrutura de pastas relevante.
- Arquitetura e fluxo de dados.
- Entidades e banco de dados.
- Features e dominios.
- Endpoints da API.
- Rotas do frontend (quando aplicavel).
- Integracoes externas.
- Padroes e convencoes.
- Testes.
- Deploy e infraestrutura.
- Comandos uteis.

No frontmatter usar:

```yaml
---
name: project-specialist-in-<projeto>
description: "Especialista no projeto <projeto>. Conhece toda a arquitetura, features, padroes e convencoes. Use para consultar qualquer aspecto do projeto."
---
```

No `agents/openai.yaml` usar:

```yaml
interface:
  display_name: "Project Specialist: <projeto>"
  short_description: "Help with <projeto> project tasks"
  default_prompt: "Use $project-specialist-in-<projeto> to handle this task with the skill workflow."
```

## Modo Atualizacao

Se a skill do projeto ja existir:

- Ler o estado atual da skill antes de editar.
- Atualizar apenas o que mudou na codebase.
- Manter dados ainda validos.
- Remover informacoes obsoletas.

## Regras de Qualidade

- Nao inventar informacoes.
- Basear tudo em leitura real do codigo.
- Priorizar utilidade pratica para engenharia.
- Escrever de forma estruturada e consultavel.
- Nao copiar blocos grandes de codigo; sintetizar com precisao.

## Checklist Final

1. Confirmar existencia de `/Users/marciofmjr/dev/<projeto>`.
2. Explorar codebase seguindo as 6 fases.
3. Criar ou atualizar `project-specialist-in-<projeto>`.
4. Atualizar `SKILLS_INVENTORY.md`.
5. Validar a skill com `quick_validate.py`.
6. Reportar ao usuario o que foi criado ou atualizado.
