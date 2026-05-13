# GW Dev Tools

Ferramentas Git para apoiar o fluxo de correcao de branches dos projetos GW.

## Comando disponivel

### `git bizu-fix`

Recria uma branch de story a partir da sua branch base atualizada e reaplica somente os commits da story.

Exemplo:

```bash
git bizu-fix gweb-saas-fix-5901
```

## Instalacao

Clone este repositorio:

```bash
git clone https://github.com/Danilo-tec-2003/git-bizu-fix.git
```

Garanta permissao de execucao:

```bash
chmod +x ~/gw-dev-tools/git-bizu-fix
```

Crie uma pasta local para comandos, caso ainda nao exista:

```bash
mkdir -p ~/bin
```

Crie o link do comando:

```bash
ln -sf ~/gw-dev-tools/git-bizu-fix ~/bin/git-bizu-fix
```

Garanta que `~/bin` esteja no `PATH`.

Bash:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

Zsh:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

Valide:

```bash
git bizu-fix
```

Se estiver instalado corretamente, o comando deve mostrar a mensagem de uso.

## Como usar

Entre no repositorio do projeto:

```bash
cd ~/Projetos/gweb
```

Garanta que a branch da story existe localmente:

```bash
git branch --list gweb-saas-fix-5901
```

Execute:

```bash
git bizu-fix gweb-saas-fix-5901
```

O script vai mostrar o projeto, a branch de trabalho, a branch base detectada e o numero da story.

Antes de alterar a branch, ele pede confirmacao digitando o nome completo da branch:

```text
Para confirmar, digite o nome da branch:
```

Exemplo:

```text
gweb-saas-fix-5901
```

Ao final, revise a branch e envie para o remoto:

```bash
git push origin gweb-saas-fix-5901 --force-with-lease
```

A branch que deve ir para o PR e sempre a branch original da story, nao a branch de backup.

## Projetos suportados

| Projeto | Padrao aceito | Base detectada |
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

Os commits da story precisam comecar com o numero da story entre parenteses:

```text
(5901)fix: ajusta validacao de frete
```

O script so reaplica commits encontrados no backup com esse padrao:

```text
^(5901)
```

## O que o script faz

1. Valida o projeto e o nome da branch.
2. Identifica o numero da story pelo final da branch.
3. Detecta a branch base removendo o numero final.
4. Bloqueia execucao se houver merge, rebase, cherry-pick ou revert em andamento.
5. Bloqueia execucao se houver alteracoes locais nao commitadas.
6. Verifica backups antigos e oferece limpeza assistida.
7. Pede confirmacao digitando o nome da branch.
8. Cria uma branch local de backup.
9. Atualiza a base local a partir do `origin`.
10. Reseta a branch da story para a base atualizada.
11. Reaplica os commits da story com `git cherry-pick`.
12. Orienta o push final com `--force-with-lease`.

## Branch de backup

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

A limpeza so acontece se o dev digitar exatamente:

```text
DELETE
```

Se pressionar Enter ou digitar qualquer outra coisa, os backups sao mantidos.

## Conflitos

Se algum `cherry-pick` gerar conflito, o script para imediatamente e deixa a resolucao para o dev.

Fluxo recomendado:

```bash
git status
# resolver os conflitos nos arquivos
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

Resolva a situacao indicada e execute novamente.

