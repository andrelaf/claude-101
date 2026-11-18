---
name: nova-dica
description: This skill should be used when the user finishes reading a tip from the 101 Claude Code Tips book and asks to add it to the study page. Runs a six-phase pipeline that ends with one new entry in the TIPS array, committed, republished and pushed. For the study page only, never for other HTML files.
allowed-tools: Bash(grep *), Bash(ls *), Bash(mkdir *), Bash(cat *), Bash(date *), Bash(git add 101-dicas-claude-code.html), Bash(git commit *), Bash(git fetch *), Bash(git diff *), Bash(git push origin main)
---

# Nova dica na página

Frases-gatilho que o usuário costuma digitar:

- "acrescenta a dica 24"
- "li a dica nova, pode incluir"
- "vamos para a dica seguinte"

## Estado no momento da invocação

- Dicas já na página: !`grep -c '^  { n:[0-9]*, ch:' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`
- Marcador `here:true` está em: !`grep -o 'n:[0-9]*, ch:[0-9]*, here:true' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`
- Contador do cabeçalho: !`grep -o 'gaugeCount">[^<]*' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`
- Capítulos existentes: !`grep -o '{ n: [0-9]*, title:' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html" | grep -o '[0-9][0-9]*' | xargs`
- Execuções anteriores: !`ls -1 "${CLAUDE_PROJECT_DIR}/output/nova-dica" 2>/dev/null | tail -8 | xargs`
- Carimbo para esta execução: !`date +%Y%m%d-%H%M%S`

## Purpose

Acrescentar uma dica do livro à página de estudo: um objeto novo no array `TIPS`,
o marcador de posição atual movido, e os contadores do cabeçalho atualizados.
A dica entra em português, com o título original em inglês preservado.

Desde 2026-11-18, por decisão do usuário, **invocar `/nova-dica NN` é a única
aprovação do fluxo**: a skill vai da extração até o push sem parar, e o usuário pede
ajustes depois de publicado. As únicas paradas que restam são o portão A (execução
anterior da mesma dica) e um capítulo novo, porque os dois precisam de uma decisão ou de
um texto que só o usuário tem.

Não usar esta skill para reescrever dicas já publicadas nem para mudar o design da
página. Republicar sem dica nova continua sendo `/publicar-dicas`.

## Portão A: Reset ou Resume

Primeira coisa que a skill faz, antes da fase 0.

Procurar execuções anteriores desta mesma dica com `ls -d output/nova-dica/dica-NN-*`.
Sem nenhuma, seguir direto para a fase 0 sem perguntar nada.

**Com alguma, STOP no portão A.** Ler `references/portao-a.md` e conduzir o diálogo
descrito lá: dizer em qual fase a execução anterior parou e perguntar ao usuário entre
começar de novo e retomar. Nunca resetar sozinho, nunca retomar sozinho, nem quando a
pasta anterior parecer obviamente abandonada.

**Saídas:** `<job>/escolha.md`, com a opção escolhida, a frase do usuário copiada
literalmente e a data.

## Fase 0: Abrir a pasta da execução

**Idempotência.** O padrão de todas as fases: se a saída da fase já existe em `<job>/`
e não está vazia, a fase é pulada. "Existe" sozinho não basta — um arquivo de zero
byte é uma execução que morreu no meio da escrita, e vale como ausente. Saída grande
é escrita em `<arquivo>.parcial` e renomeada no fim, para nunca existir pela metade.

**Entradas:** o número da dica, dado pelo usuário no argumento da skill.
**Saídas:** `<job>/pedido.md` com o número, o capítulo a que a dica
pertence segundo o sumário do livro, e a data da execução.
**Pular se:** `<job>/pedido.md` já existe e não está vazio.

O `jobId` desta skill é `dica-NN-YYYYMMDD-HHMMSS`: o número da dica mais o carimbo
injetado no bloco de estado. O formato do caminho é constante, `output/<skill>/<jobId>/`,
e duas execuções da mesma dica nunca escrevem uma sobre a outra.

Daqui em diante, `<job>` neste documento significa
`output/nova-dica/dica-NN-YYYYMMDD-HHMMSS/`, a pasta criada aqui.

Criar a pasta com `mkdir -p`. O portão A já resolveu se esta é uma pasta nova
ou a retomada de uma antiga.

## Fase 1: Extrair o texto do livro

**Entradas:** `<job>/pedido.md`.
**Saídas:** `<job>/fonte.md` com o trecho da dica copiado de
`9781808650314.pdf`, incluindo o título em inglês, qualquer bloco de código e o
"Learn more" quando houver.
**Pular se:** `<job>/fonte.md` já existe e não está vazio. O PDF não é reaberto.

## Fase 2: Conferir as afirmações técnicas

**Entradas:** `<job>/fonte.md`.
**Saídas:** `<job>/verificacao.md`, com uma linha por afirmação: o que o livro
diz, o que a documentação atual diz, e se batem. Quando a dica não citar campo, flag
nem comando, o arquivo registra "sem afirmação técnica a verificar" e a fase termina.

**Pular se:** `<job>/verificacao.md` já existe e não está vazio. Vale inclusive quando
o conteúdo é "sem afirmação técnica a verificar": a fase rodou, a conclusão está escrita.

Consultar a documentação via Context7, não de memória.

## Portão B: Aprovação do texto

**Portão B automático.** Decidido pelo usuário em 2026-11-18 ("só dá o ok uma vez. se
precisar de ajuste peço depois de publicado"): o portão não para mais por divergência
nem por conteúdo fora do livro. A invocação da skill é a aprovação. Ele só para quando
a dica abre capítulo novo (ver Rules).

O portão grava o `aprovacao.md` com a linha
`Aprovação automática (critério do usuário, 2026-11-18): a invocação de /nova-dica é a
aprovação; ajustes são pedidos depois de publicado.` no lugar da frase do usuário. Logo
abaixo vem uma seção `### Para revisar`, que lista as divergências do `verificacao.md` e
tudo o que está fora do livro, um item por linha (`note`, links de `docs`, inferências,
negritos que o livro não tem). Sem nada a listar, a seção diz "nada". Depois vem o bloco
`## Approved YYYY-MM-DD` de sempre.

O portão continua existindo mesmo sem parar: ele é o ponto em que o texto fica decidido,
e a seção `Para revisar` é o que o usuário lê depois. Ela vai inteira para o relatório
final, junto com o objeto completo.

**Entradas:** o objeto redigido a partir de `fonte.md` e `verificacao.md`.
**Saídas:** `<job>/aprovacao.md`, contendo a frase de aprovação do usuário
copiada literalmente e, abaixo dela, um bloco `## Approved YYYY-MM-DD` com o
conteúdo final campo por campo.

**Pular se:** `<job>/aprovacao.md` já existe e contém um bloco `## Approved`. Só conta
a aprovação desta pasta: `aprovacao.md` de outro `jobId` nunca abre este portão, porque
foi dada para outro texto.

Esse bloco pós-portão é a fonte de verdade das fases seguintes. A fase 3 lê dele,
não do rascunho que foi mostrado no chat: divergência entre o bloco e o que entrou
na página é erro da fase 3, não revisão de escopo. A data vem do dia da aprovação,
nunca do dia da retomada.

## Fase 3: Redigir e inserir

**Entradas:** o bloco `## Approved YYYY-MM-DD` de `<job>/aprovacao.md`.
Sem esse arquivo, a fase 3 não começa; sem o bloco datado dentro dele, também não.
**Saídas:** o objeto novo ao final do array `TIPS`, o `here:true` movido da dica
anterior para a nova, os contadores do cabeçalho atualizados (`gaugeCount`,
"Cobertura atual", "Leitura: dica N de 101", "Capítulos cobertos"), e uma cópia do
objeto em `<job>/objeto.js`.

Campos: `n`, `ch`, `title` (português), `orig` (inglês, idêntico ao sumário), `body`
(HTML), `code` (um objeto ou uma lista deles), `note` e `docs`.

**Pular se:** a página já contém `{ n:NN, ch:` — a dica foi inserida numa execução
anterior. Inserir de novo duplicaria a dica no array, que é o erro mais caro que esta
fase pode cometer. Quando a página já tem a dica mas `<job>/objeto.js` falta, só a
cópia é refeita.

## Fase 4: Verificar

**Entradas:** a página alterada.
**Saídas:** `<job>/revisao.txt` com o relatório de `/revisar-pagina-dicas`,
e o veredito ecoado no bloco final.
**Pular se:** nunca. A verificação roda toda vez, mesmo em retomada: ela é barata, e é
ela que detecta o estrago de uma execução anterior interrompida.

## Fase 5: Publicar

**Entradas:** `<job>/revisao.txt` com veredito "pronto para publicar". Com qualquer
outro veredito, a fase 5 não começa: parar e reportar o que a revisão achou. Página
quebrada no ar custa mais que uma parada.
**Saídas:** `<job>/publicacao.txt` com o SHA do commit, a versão do artifact e a linha
do `git status -sb` depois do push.
**Pular se:** `<job>/publicacao.txt` já existe e não está vazio.

Os passos seguem o Workflow da `/publicar-dicas`, que não pode ser chamada daqui por
ter `disable-model-invocation: true`:

1. `git status --short`. Se houver arquivo alterado que não seja a página, um arquivo
   de skill ou `output/`, parar e perguntar.
2. Commitar só a página: `git add 101-dicas-claude-code.html`, mensagem
   `dicas: adiciona dica NN (<título curto em minúsculas>)` com as linhas de
   atribuição da sessão. Nunca `git add -A`.
3. Republicar o artifact `https://claude.ai/artifact/7AaiBkpATMMkncbLV6v9SU` a partir
   de `101-dicas-claude-code.html`. Nunca criar artifact novo. Se a sessão ainda não
   publicou nem leu esse artifact, fazer antes o `read` sem `path` e ler o arquivo
   salvo inteiro em faixas, como diz a lição da `/publicar-dicas`.
4. `git fetch origin` e `git diff --stat origin/main..main`. Se algum `.pdf` aparecer,
   parar. Senão, `git push origin main`. Nunca `--force`.

Se um passo falhar, parar nele e reportar. O que já rodou fica registrado em
`publicacao.txt`, para a retomada pular os passos feitos.

## Retomada

As saídas de cada fase são arquivos, então uma execução interrompida não recomeça
do zero. Ao retomar por decisão do portão A, pular para a primeira fase cuja saída
está faltando, na ordem: `pedido.md`, `fonte.md`, `verificacao.md`, `aprovacao.md`,
`objeto.js`, `revisao.txt`, `publicacao.txt`.

Retomar acontece dentro da pasta antiga, com o carimbo original. Carimbo é hora de
início da execução, não hora da retomada — a pasta nasce uma vez só.

Retomar numa fase posterior ao portão B exige que `aprovacao.md` exista. Uma execução
anterior que morreu no portão volta para o portão, nunca para depois dele.

Ao escolher "começar de novo", **a pasta antiga não é apagada**. Ela fica como estava,
e a execução nova ganha carimbo próprio. Reset aqui significa pasta nova, nunca
`rm -rf` na anterior.

## Rules

- Nunca inventar conteúdo que não esteja no livro. Se o livro não deu exemplo de
  código, a dica entra sem `code`.
- Nunca alterar uma dica já existente, exceto para remover o `here:true` dela.
- Sempre preservar `orig` em inglês, exatamente como no sumário do livro.
- Quando a documentação atual contradisser o livro, escrever o que a documentação
  diz e registrar a diferença na `note` — nunca escolher em silêncio.
- Parar e perguntar quando o capítulo da dica ainda não existir em `CHAPTERS`:
  um capítulo novo precisa de título e de nota introdutória escritos pelo usuário.
  O bloco de estado acima lista os capítulos que já existem.
- Nunca pular uma fase por achar que ela é rápida: a saída em disco é o que permite
  retomar. Fase sem arquivo é fase que não aconteceu.
- Nunca publicar sem que a fase 4 tenha passado com "pronto para publicar".
- Nunca apagar os intermediários de `<job>/` como parte de uma execução bem-sucedida.
  Eles são o que permite retomar e depurar. Limpeza só a pedido explícito do usuário,
  e só depois de verificar que a página contém a dica e que ela foi publicada.
- Escrever apenas em `<job>/` e em `101-dicas-claude-code.html`.
  Não escrever em nenhum outro lugar do projeto. Nenhuma fase cria arquivo fora
  desses dois destinos, nem "temporariamente". Os únicos efeitos fora deles são o
  commit, a republicação e o push da fase 5.
- Ajuste pedido depois de publicado é trabalho novo: editar a dica, rodar a fase 4 de
  novo e publicar com a mensagem `dicas: ajusta dica NN (<o que mudou>)`.

## Lessons Learned

As lições desta skill vivem em `references/licoes.md`. Ler esse arquivo quando uma
fase falhar, quando um valor injetado parecer estranho, ou antes de editar esta skill.

**Manutenção.** Ao terminar uma execução, revisar as seções relevantes desta skill
para refletir o que foi aprendido, e acrescentar a entrada nova em
`references/licoes.md`. Toda linha de `PENDÊNCIAS` no relatório final é candidata.
Entrada nova só depois de um erro que de fato aconteceu, nunca preventivamente.

## Output format

```
DICA: <n> — <título em português>
FASES: portão A <nova | retomada da fase N> | 0 <ok> | 1 <ok> | 2 <ok|n/a>
       | portão B <automático YYYY-MM-DD> | 3 <ok> | 4 <ok> | 5 <ok>
PASTA: output/nova-dica/dica-NN-YYYYMMDD-HHMMSS/ (<n> arquivos)
CAMPOS: body <ok> | code <n blocos> | note <sim/não> | docs <n>
MARCADOR: here movido de <n-1> para <n>
CONTADORES: <ok | divergência>
REVISÃO: <veredito de /revisar-pagina-dicas>
COMMIT: <sha curto> <mensagem>
URL: <url do artifact> (versão <n>)
PUSH: <main == origin/main | pendente e por quê>
PARA REVISAR: <nada | a lista do aprovacao.md>
PENDÊNCIAS: <nenhuma | lista>
```

Depois do bloco, o objeto da dica por inteiro (title, orig, body, code, note, docs),
para o usuário ler o que entrou sem abrir a página.
