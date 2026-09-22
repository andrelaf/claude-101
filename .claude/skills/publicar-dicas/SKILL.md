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

Nenhuma ainda: esta skill nunca foi executada. A seção existe vazia de propósito, para
que a primeira execução tenha onde escrever em vez de virar comentário no chat.

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
