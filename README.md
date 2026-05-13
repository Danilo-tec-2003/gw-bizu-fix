# GW Dev Tools

Ferramentas Git para apoiar o fluxo de correcao de branches dos projetos GW.

## Visao geral

Este repositorio disponibiliza dois comandos:

```bash
git bizu-fix <branch>
```

Recria uma branch de story a partir da branch base atualizada e reaplica somente os commits da story.

```bash
git bizu-fix install-hook
```

Instala o validador de mensagem de commit no repositorio atual.

Use o `git bizu-fix` quando uma branch de story precisar ser refeita em cima da base mais recente.

O script faz backup local da branch atual, atualiza a base pelo `origin`, reseta a branch da story para essa base e reaplica os commits da story usando `git cherry-pick`.

## Requisitos

Antes de instalar, confirme que sua maquina possui:

- Git instalado;
- acesso ao GitHub;
- terminal Bash ou Zsh;
- permissao para criar arquivos em `~/bin`.

## Instalacao da ferramenta

### 1. Clone este repositorio

Entre na pasta onde voce costuma guardar seus projetos:

```bash
cd ~/Documentos/Projetos
```

Clone o repositorio:

```bash
git clone https://github.com/Danilo-tec-2003/gw-dev-tools.git
```

Entre na pasta criada:

```bash
cd ~/Documentos/Projetos/gw-dev-tools
```

### 2. De permissao de execucao aos scripts

```bash
chmod +x git-bizu-fix git-bizu-commit-msg
```

### 3. Crie a pasta local de comandos

```bash
mkdir -p ~/bin
```

### 4. Crie os links dos comandos

```bash
ln -sf ~/Documentos/Projetos/gw-dev-tools/git-bizu-fix ~/bin/git-bizu-fix
ln -sf ~/Documentos/Projetos/gw-dev-tools/git-bizu-commit-msg ~/bin/git-bizu-commit-msg
```

Esses links permitem chamar a ferramenta como comandos Git.

O Git reconhece executaveis chamados `git-alguma-coisa` como comandos no formato `git alguma-coisa`, desde que estejam no `PATH`.

### 5. Adicione `~/bin` ao PATH

Primeiro, veja qual shell voce usa:

```bash
echo $SHELL
```

Se o resultado for `/bin/bash`, rode:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Se o resultado for `/bin/zsh`, rode:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### 6. Valide a instalacao

Rode:

```bash
git bizu-fix
```

Se estiver instalado corretamente, o terminal deve mostrar a mensagem de uso:

```text
Uso:
  git bizu-fix <branch>
  git bizu-fix install-hook
```

Exemplo de instalacao validada:

![Instalacao do git bizu-fix](docs/images/sigaGit.png)

## Instalacao do validador de commits

Para bloquear commits fora do padrao, instale o hook dentro de cada repositorio GW em que voce usa o fluxo.

Exemplo para o `webtrans`:

```bash
cd ~/Documentos/Projetos/webtrans
git bizu-fix install-hook
```

Exemplo para o `gweb`:

```bash
cd ~/Documentos/Projetos/gweb
git bizu-fix install-hook
```

Exemplo para o `gw-base-webtrans`:

```bash
cd ~/Documentos/Projetos/gw-base-webtrans
git bizu-fix install-hook
```

Exemplo para o `gw-lib`:

```bash
cd ~/Documentos/Projetos/gw-lib
git bizu-fix install-hook
```

Depois disso, o Git passa a validar a mensagem em todo `git commit` feito naquele repositorio.

Exemplo bloqueado na branch `webtrans-saas-elastic-2045`:

```bash
git commit -m "testando"
```

Exemplo aceito:

```bash
git commit -m "(2045)fix: ajusta validacao"
```

## Uso do `git bizu-fix`

### 1. Entre no repositorio do projeto

Exemplo para o projeto `gweb`:

```bash
cd ~/Documentos/Projetos/gweb
```

### 2. Confira se a branch da story existe localmente

```bash
git branch --list gweb-saas-fix-5901
```

Se o comando nao mostrar nada, traga a branch do remoto antes de continuar:

```bash
git fetch origin
git checkout gweb-saas-fix-5901
```

### 3. Confirme que nao existem alteracoes locais pendentes

```bash
git status
```

O script nao executa se houver arquivos alterados, arquivos staged, merge, rebase, cherry-pick ou revert em andamento.

### 4. Execute o comando

```bash
git bizu-fix gweb-saas-fix-5901
```

O script vai mostrar:

- repositorio detectado;
- branch de trabalho;
- branch base detectada;
- numero da story.

Antes de alterar a branch, ele pede confirmacao:

```text
Para confirmar, digite o nome da branch:
```

Digite o nome completo da branch:

```text
gweb-saas-fix-5901
```

### 5. Revise e envie a branch para o remoto

Depois que o script terminar, revise o resultado:

```bash
git status
git log --oneline --decorate -10
```

Se estiver tudo certo, envie a branch:

```bash
git push origin gweb-saas-fix-5901 --force-with-lease
```

A branch que deve ir para o PR e sempre a branch original da story, nunca a branch de backup.

## Projetos suportados

| Projeto | Padrao da branch de story | Branch base detectada |
| --- | --- | --- |
| `gweb` | `gweb-qualquer-coisa-5901` | `gweb-qualquer-coisa` |
| `webtrans` | `webtrans-qualquer-coisa-5901` | `webtrans-qualquer-coisa` |
| `gw-base-webtrans` | `master-5901` | `master` |
| `gw-lib` | `gw-lib-qualquer-coisa-5901` | `gw-lib-qualquer-coisa` |

Exemplos validos:

```bash
git bizu-fix gweb-saas-fix-5901
git bizu-fix gweb-saas-elastic-5901
git bizu-fix webtrans-saas-fix-5901
git bizu-fix master-5901
git bizu-fix gw-lib-fix-5901
```

## Regra dos commits

Os commits da story precisam comecar com o numero da story entre parenteses.

Exemplo correto:

```text
(5901)fix: ajusta validacao de frete
```

O script procura commits no backup usando o numero final da branch.

Para a branch:

```text
gweb-saas-fix-5901
```

ele reaplica somente commits que comecam com:

```text
(5901)
```

Se nenhum commit seguir esse padrao, o script nao consegue identificar o que deve ser reaplicado.

Quando o hook estiver instalado com `git bizu-fix install-hook`, o proprio `git commit` bloqueia mensagens fora desse padrao.

## O que o script faz

1. Valida se o diretorio atual esta dentro de um repositorio Git.
2. Valida se o repositorio e suportado.
3. Valida o nome da branch informada.
4. Identifica o numero da story pelo final da branch.
5. Detecta a branch base removendo o numero final.
6. Bloqueia a execucao se houver merge, rebase, cherry-pick ou revert em andamento.
7. Bloqueia a execucao se houver alteracoes locais nao commitadas.
8. Verifica backups antigos e oferece limpeza assistida.
9. Pede confirmacao digitando o nome completo da branch.
10. Cria uma branch local de backup.
11. Atualiza a branch base local a partir do `origin`.
12. Reseta a branch da story para a base atualizada.
13. Reaplica os commits da story com `git cherry-pick`.
14. Orienta o push final com `--force-with-lease`.

## Backups

Antes de resetar a branch original, o script cria uma branch local de backup.

Exemplo:

```text
gweb-saas-fix-5901_back_20260513_103000
```

Essa branch serve apenas para recuperacao local em caso de erro, falha ou necessidade de consulta.

A branch que deve ir para o PR continua sendo:

```text
gweb-saas-fix-5901
```

O script nao envia a branch de backup para o remoto.

## Limpeza de backups

Para evitar acumulo, o script verifica backups locais com mais de 30 dias a cada execucao.

Ele remove apenas branches locais no padrao:

```text
*_back_YYYYMMDD_HHMMSS
```

Quando encontrar backups antigos, mostra a lista e pergunta:

```text
Para remover esses backups, digite DELETE.
Para manter, pressione Enter:
```

A limpeza so acontece se voce digitar exatamente:

```text
DELETE
```

Se pressionar Enter ou digitar qualquer outra coisa, os backups sao mantidos.

## Conflitos durante o cherry-pick

Se algum `cherry-pick` gerar conflito, o script para imediatamente e deixa a resolucao para o desenvolvedor.

Fluxo recomendado:

```bash
git status
# resolva os conflitos nos arquivos
git add <arquivos-resolvidos>
git cherry-pick --continue
```

Se quiser cancelar o cherry-pick em conflito:

```bash
git cherry-pick --abort
```

Se existirem commits pendentes depois do conflito, o script mostra quais ainda faltam e imprime os comandos para aplicar manualmente.

## Situacoes bloqueadas

O script nao executa se encontrar:

- alteracoes locais nao commitadas;
- merge em andamento;
- rebase em andamento;
- cherry-pick em andamento;
- revert em andamento;
- branch fora do padrao do projeto;
- repositorio nao suportado.

Resolva a situacao indicada no terminal e execute o comando novamente.

## Solucao rapida de problemas

### `git: 'bizu-fix' is not a git command`

O comando nao esta no `PATH`.

Confira se o link existe:

```bash
ls -l ~/bin/git-bizu-fix
```

Confira se `~/bin` esta no `PATH`:

```bash
echo $PATH
```

Depois rode novamente o passo de configuracao do shell, conforme Bash ou Zsh.

### `git bizu-fix` funciona, mas o commit fora do padrao ainda passa

O validador de commit provavelmente ainda nao foi instalado no repositorio atual.

Entre no repositorio do projeto e rode:

```bash
git bizu-fix install-hook
```

Depois confira se o hook existe:

```bash
ls -l .git/hooks/commit-msg
```

Se o repositorio usar `core.hooksPath`, confira o caminho configurado:

```bash
git config --get core.hooksPath
```

### `[erro] Informe o nome completo da branch.`

O comando foi chamado sem informar a branch.

Use:

```bash
git bizu-fix nome-da-branch
```

Exemplo:

```bash
git bizu-fix gweb-saas-fix-5901
```

### `[erro] A branch '...' nao existe localmente.`

Traga a branch do remoto:

```bash
git fetch origin
git checkout nome-da-branch
```

Depois execute novamente:

```bash
git bizu-fix nome-da-branch
```

### `[erro] Nenhum commit encontrado com o padrao (...)`

Os commits da story nao comecam com o numero da story entre parenteses.

Exemplo esperado:

```text
(5901)fix: ajusta validacao de frete
```
