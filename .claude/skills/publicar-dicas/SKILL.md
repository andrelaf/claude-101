---
name: publicar-dicas
description: This skill should be used when the user explicitly asks to commit and ship the study page 101-dicas-claude-code.html. Manual invocation only, because it writes to git history and republishes a live URL.
disable-model-invocation: true
allowed-tools: Bash(git add *), Bash(git status *), Bash(git diff *), Bash(git log *), Bash(git branch *), Bash(git commit *), Bash(git push *)
---

# Publicar página de dicas

## Estado do repositório neste instante

- Branch: !`git branch --show-current`
- Arquivos alterados: !`git status --short`
- Tamanho da mudança: !`git diff --stat`
- Últimos commits: !`git log --oneline -5`

Os quatro comandos rodaram antes de o Claude ler esta linha. Decidir a mensagem de
commit a partir do que está acima, no estilo dos commits recentes, sem chamar o Bash
para redescobrir nada disso.

## Purpose

Commitar e enviar a página de estudo depois que ela foi revisada. Roda apenas quando
o usuário digita `/publicar-dicas`: a skill escreve no histórico do git e republica
uma URL viva, ou seja, muda estado externo.

`disable-model-invocation: true` faz duas coisas aqui: impede o Claude de disparar a
skill sozinho no meio de uma conversa, e tira a descrição do contexto até a invocação.
O `allowed-tools` pré-aprova os comandos de git do fluxo, inclusive os quatro injetados
acima — sem ele, cada injeção pararia para pedir permissão. Ele concede, não restringe:
o que não está na lista continua possível, só volta a perguntar.

## Workflow

1. **Exigir revisão prévia.** Precondição: `/revisar-pagina-dicas` rodou nesta sessão
   com veredito "pronto para publicar". Se não rodou, parar e pedir.
2. **Ler o bloco de estado acima.** Se o branch não for `main`, parar e perguntar.
   Se houver arquivo alterado que não seja a página ou uma skill, parar e perguntar.
3. **Confirmar com o usuário.** Mostrar a mensagem de commit proposta e esperar um
   "pode publicar" explícito.
4. **Commitar.** Formato `dicas: adiciona dica NN (<título curto>)`. Adicionar os
   arquivos por nome; nunca `git add -A`.
5. **Republicar o artifact.** Mesmo caminho de arquivo, mesma URL. Nunca criar um
   artifact novo para a mesma página.
6. **Reportar.** Devolver o SHA do commit e a URL publicada.

## Rules

- Nunca rodar `git push --force`, nunca `git add -A`.
- Se a etapa 1 não puder ser comprovada, não publicar: o custo de republicar algo
  quebrado é maior que o de uma revisão a mais.
- Não alterar o conteúdo da página durante a publicação. Esta skill move bits,
  não escreve texto.

## Lessons Learned

Primeira execução em 2026-09-22, publicando as dicas 31 e 32.

- **O formato de commit do passo 4 não serve para a primeira execução.** `dicas:
  adiciona dica NN (<título curto>)` pressupõe que a página já esteja versionada e que
  cada publicação acrescente uma dica. Na estreia só o `README.md` estava no git, e o
  commit levou a página com 32 dicas, o `.gitignore` e as quatro skills de uma vez. O
  formato foi abandonado de propósito. Ele volta a valer da segunda publicação em
  diante — mas a skill não deve tratá-lo como obrigatório sem antes olhar
  `git ls-files`.
- **O passo 2 pegou o que devia, e o motivo era maior que o esperado.** O
  `9781808650314.pdf` apareceu no status. A regra "parar se houver arquivo que não seja
  a página ou uma skill" funcionou — mas o risco real não era ruído no commit: o remote
  é `github.com/andrelaf/claude-101`, e subir o PDF publicaria um livro comercial da
  Zenva. Resolvido acrescentando o arquivo ao `.gitignore`. Ao checar o status, olhar
  também `git remote -v`: o que o passo 2 protege depende de para onde o push vai.
- **Republicar o artifact custa duas leituras — mas só na primeira vez da sessão.**
  `action: "read"` com `path: "index.html"` baixa o arquivo e **não** conta como ter
  visto a versão; é preciso o `read` sem `path` e depois `Read` em todas as linhas do
  arquivo salvo que ele indica (735 linhas na estreia). Da segunda publicação da mesma
  sessão em diante isso não se repete: a sessão já publicou o artifact, e o
  `publish` com `url` passa direto. Orçar o custo uma vez por sessão, não uma vez por
  execução da skill.
  Na publicação da dica 35 o arquivo salvo tinha 773 linhas e passou do limite de
  ~25 mil tokens de uma `Read` só, que falhou. Ler em faixas com `offset`/`limit`
  (1–290, 291–540, 541–fim funcionou) e conferir que a última faixa chega à linha
  que o `read` informou. O arquivo cresce a cada dica, então a divisão só piora.
- **A manutenção sempre cai depois do commit, e suja o repositório de novo.** O passo 4
  commita, e só então esta seção é atualizada — então toda execução termina com o
  SKILL.md alterado e fora do commit que acabou de subir. Aconteceu duas vezes
  seguidas. O conserto é fazer a revisão desta seção **antes** do passo 4, para as
  lições da execução entrarem no mesmo commit. Na terceira execução o conserto foi
  aplicado à mão e funcionou: a pendência fechou. O preço é que as lições precisam ser
  decididas antes de o passo 5 rodar, então uma falha na republicação ainda cai fora do
  commit — e essa, sim, vira `PENDÊNCIAS MANUAIS`. O Workflow segue sem a reordenação,
  que é mudança de comportamento e cabe ao usuário.
- **Dois commits são melhores que um quando as mudanças não têm relação.** Na segunda
  execução havia a dica 33 e a seção de lições pendente. Um commit só, com o formato
  `dicas: adiciona dica NN`, descreveria mal metade do conteúdo. Separar em
  `dicas: ...` e `skills: ...` custou nada e manteve o histórico honesto. O formato do
  passo 4 governa o commit da dica, não a execução inteira.
- **O usuário pode aprovar reinvocando o comando.** O passo 3 pede um "pode publicar"
  explícito. Na estreia o usuário digitou `/publicar-dicas` de novo em vez da frase.
  Tratado como aprovação, e dito em voz alta antes de commitar. Reinvocar o comando de
  publicação depois de ver a mensagem proposta é intenção suficiente; pedir a frase
  literal uma terceira vez seria burocracia.

**Manutenção desta seção.** Ao terminar uma execução, revisar as seções relevantes
desta skill para refletir o que foi aprendido. Toda linha de `PENDÊNCIAS MANUAIS` no
relatório final é candidata a virar entrada aqui.

## Output format

```
COMMIT: <sha curto> <mensagem>
ARQUIVOS: <lista>
URL: <url do artifact>
PENDÊNCIAS MANUAIS: <nenhuma | lista>
```
