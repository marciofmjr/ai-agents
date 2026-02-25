# AI Agents (Skills) - Setup para Time

Repositório oficial de skills/agentes para uso em ferramentas de IA no fluxo de desenvolvimento.

Objetivo: qualquer pessoa do time clonar este repositório, rodar o passo a passo e começar a usar os mesmos agentes no **Codex**, **Claude Code** e **Antigravity**.

## O que está versionado

- `skills/`: catálogo de skills (cada skill com seu `SKILL.md`)
- `skills/SKILLS_INVENTORY.md`: inventário resumido das skills disponíveis

Estrutura recomendada do repositório:

```text
ai-agents/
  README.md
  AGENTS.md            # instruções globais para Codex (opcional, mas recomendado)
  CLAUDE.md            # instruções globais para Claude/Antigravity (opcional)
  skills/
    <skill-name>/SKILL.md
```

## Pré-requisitos

- macOS ou Linux
- `git` instalado
- acesso ao repositório no GitHub
- ferramenta já instalada (Codex CLI, Claude Code e/ou Antigravity)

## 1) Clonar o repositório

```bash
git clone git@github.com:SEU_USUARIO/ai-agents.git ~/dev/ai-agents
cd ~/dev/ai-agents
```

Se preferir HTTPS:

```bash
git clone https://github.com/SEU_USUARIO/ai-agents.git ~/dev/ai-agents
cd ~/dev/ai-agents
```

## 2) Configurar no Codex

O Codex carrega skills em `~/.codex/skills`. A forma recomendada é usar **symlink** para este repo.

### 2.1 Backup (se já existir configuração local)

```bash
mkdir -p ~/.codex
[ -d ~/.codex/skills ] && mv ~/.codex/skills ~/.codex/skills.backup.$(date +%Y%m%d-%H%M%S)
```

### 2.2 Apontar para o repositório versionado

```bash
ln -sfn ~/dev/ai-agents/skills ~/.codex/skills
```

Se você versionar também um `AGENTS.md` no repo:

```bash
ln -sfn ~/dev/ai-agents/AGENTS.md ~/.codex/AGENTS.md
```

### 2.3 Validar

```bash
readlink ~/.codex/skills
ls -la ~/.codex/skills | head
```

Saída esperada no `readlink`: `~/dev/ai-agents/skills`.

## 3) Configurar no Claude Code

O Claude Code suporta skills em `~/.claude/skills` (escopo usuário) e também em `.claude/skills` (escopo de projeto).

### Opção recomendada (usuário global)

```bash
mkdir -p ~/.claude
[ -d ~/.claude/skills ] && mv ~/.claude/skills ~/.claude/skills.backup.$(date +%Y%m%d-%H%M%S)
ln -sfn ~/dev/ai-agents/skills ~/.claude/skills
```

### Validação

1. Abra uma sessão do Claude Code.
2. Rode `/skills` na conversa.
3. Confirme que as skills do repositório aparecem na lista.

### Opção por projeto (alternativa)

No repositório do projeto em que você está trabalhando:

```bash
mkdir -p .claude
ln -sfn ~/dev/ai-agents/skills .claude/skills
```

## 4) Configurar no Antigravity

No Antigravity (com extensão do Claude Code), o caminho de skills segue a mesma convenção do Claude (`~/.claude/skills` ou `.claude/skills` por projeto).

### 4.1 Confirmar extensão

- Abra o Antigravity.
- Verifique se a extensão **Claude Code** está instalada/ativa.

### 4.2 Configuração de skills (global)

```bash
mkdir -p ~/.claude
[ -d ~/.claude/skills ] && mv ~/.claude/skills ~/.claude/skills.backup.$(date +%Y%m%d-%H%M%S)
ln -sfn ~/dev/ai-agents/skills ~/.claude/skills
```

Se você versionar um `CLAUDE.md` no repo:

```bash
ln -sfn ~/dev/ai-agents/CLAUDE.md ~/.claude/CLAUDE.md
```

### 4.3 Validar carregamento

1. Abra um workspace de código.
2. Inicie uma conversa no Claude Code dentro do Antigravity.
3. Rode `/skills` e confirme a presença das skills.

## 5) Atualização quando houver merge de PR

Como as ferramentas apontam por symlink para este repo, atualizar é simples:

```bash
cd ~/dev/ai-agents
git pull
```

As skills novas/alteradas passam a valer imediatamente (pode ser necessário reiniciar a sessão da ferramenta).

## 6) Fluxo de contribuição

```bash
cd ~/dev/ai-agents
git checkout -b feat/minha-skill
# editar arquivos em skills/
git add .
git commit -m "feat: adiciona skill xyz"
git push -u origin feat/minha-skill
```

Depois abra PR no GitHub.

## 7) Exemplos de uso dos agentes

Use os nomes de skills diretamente no prompt.

### Exemplo A: revisão de PR

```text
Use o code-reviewer para revisar este diff com foco em regressão, segurança e testes faltantes.
```

### Exemplo B: planejamento + execução

```text
Primeiro use task-planner para decompor a mudança em etapas, depois use backend-specialist para implementar a API.
```

### Exemplo C: debugging

```text
Use debugger-specialist para reproduzir este bug, isolar causa raiz e propor correção com teste.
```

### Exemplo D: frontend Angular

```text
Use angular-specialist para refatorar este componente e melhorar legibilidade/performance sem quebrar contratos.
```

## 8) Troubleshooting

### `readlink ~/.codex/skills` não aponta para o repo

Recrie o link:

```bash
rm -f ~/.codex/skills
ln -sfn ~/dev/ai-agents/skills ~/.codex/skills
```

### Skill não aparece

- Confirme que a pasta existe: `ls ~/dev/ai-agents/skills`
- Confirme nome da skill e `SKILL.md`
- Reinicie a sessão da ferramenta (Codex/Claude/Antigravity)
- No Claude/Antigravity, rode `/skills` para verificar discovery

### Mudou de máquina

Repita as seções 1 a 4.

## 9) Checklist rápido para novos devs

1. Clonar `ai-agents` em `~/dev/ai-agents`
2. Criar symlink `~/.codex/skills -> ~/dev/ai-agents/skills`
3. Criar symlink `~/.claude/skills -> ~/dev/ai-agents/skills`
4. Abrir Codex/Claude/Antigravity e validar com listagem de skills
5. Sempre atualizar com `git pull` no repo de agentes

---

Se quiser padronizar ainda mais, adicione um script `scripts/bootstrap.sh` neste repositório para automatizar tudo em um único comando.
