# GW Dev Tools

Ferramentas Git para apoiar o fluxo de correcao de branches dos projetos GW.

## Comando disponivel

### `git bizu-fix`

Recria uma branch de story a partir da sua branch base atualizada e reaplica somente os commits da story.

Exemplo:

```bash
git bizu-fix gweb-saas-fix-5901
```

O script:

1. valida o projeto e o nome da branch;
2. identifica o numero da story pelo numero final da branch;
3. identifica a branch base removendo o numero final;
4. verifica se existem backups locais antigos para limpeza assistida;
5. cria uma branch de backup;
6. atualiza a base local a partir do `origin`;
7. reseta a branch da story para a base atualizada;
8. busca no backup os commits que comecam com `(numero-da-story)`;
9. reaplica esses commits com `git cherry-pick`;
10. orienta o push final com `--force-with-lease`.

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

## Regras importantes

Os commits da story precisam comecar com o numero entre parenteses:

```text
(5901)fix: ajusta validacao de frete
```

O script so reaplica commits encontrados no backup com esse padrao:

```text
^(5901)
```

## Seguranca

Antes de alterar a branch, o script bloqueia a execucao se existir:

- alteracao local nao commitada;
- merge em andamento;
- rebase em andamento;
- cherry-pick em andamento;
- revert em andamento.

Antes de continuar, o script tambem exige que o dev digite o nome completo da branch.

Exemplo:

```text
Para confirmar, digite o nome da branch: gweb-saas-fix-5901
```

Isso reduz o risco de resetar a branch errada por acidente.

## Limpeza de backups

O `bizu-fix` cria branches locais de backup antes de resetar a branch original.

Exemplo:

```text
gweb-saas-fix-5901_back_20260513_103000
```

Essas branches sao usadas apenas como seguranca em caso de falha ou recuperacao. A branch que deve ir para o PR continua sendo a branch original, por exemplo:

```text
gweb-saas-fix-5901
```

Para evitar acumulo, o script verifica backups locais com mais de 30 dias a cada execucao.

Quando encontrar backups antigos, mostra uma lista:

```text
[info] Foram encontrados backups com mais de 30 dias:

gweb-saas-fix-5600_back_20260401_091500
gweb-saas-elastic-5702_back_20260405_143000

Essas branches sao backups locais criados pelo bizu-fix.
Elas nao sao usadas no PR e normalmente servem apenas para recuperacao.

Para remover esses backups, digite DELETE.
Para manter, pressione Enter:
```

A limpeza so acontece se o dev digitar exatamente:

```text
DELETE
```

Se pressionar Enter ou digitar qualquer outra coisa, os backups sao mantidos.

A limpeza remove somente branches locais que seguem o padrao:

```text
*_back_YYYYMMDD_HHMMSS
```

O script nunca tenta remover a branch atual durante essa limpeza.

## Conflitos

Se algum `cherry-pick` gerar conflito, o script para imediatamente e deixa a resolucao para o dev.

O dev deve:

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

Quando houver commits restantes depois do commit que conflitou, o script mostra quais ainda faltam e imprime os comandos para aplicar manualmente.

## Instalacao local para teste

Durante a fase de teste, rode o script diretamente:

```bash
./gw-dev-tools/git-bizu-fix gweb-saas-fix-5901
```

Para testar no formato de subcomando Git, adicione a pasta ao `PATH` ou copie/symlink o arquivo para uma pasta que ja esteja no `PATH`.

Exemplo:

```bash
mkdir -p ~/bin
ln -s /caminho/para/gw-dev-tools/git-bizu-fix ~/bin/git-bizu-fix
```

Depois disso:

```bash
git bizu-fix gweb-saas-fix-5901
```

## Validacao tecnica

Antes de publicar alteracoes no script, rode:

```bash
bash -n git-bizu-fix
shellcheck git-bizu-fix
```

`bash -n` valida a sintaxe. `shellcheck` faz uma analise estatica mais completa e aponta problemas comuns em scripts Shell.
