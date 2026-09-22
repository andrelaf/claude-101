---
name: estado
description: This skill should be used when the user asks for the current state of this project — date, branch, uncommitted files and how many tips the study page has. Read-only, never edits or publishes anything.
allowed-tools: Bash(date *), Bash(git branch *), Bash(git status *), Bash(grep *)
---

# Estado do projeto

Demonstração da dica 20. Tudo abaixo já chegou pronto: os comandos rodaram antes
de o Claude ler esta linha, e no lugar de cada um está a resposta.

- Data e hora: !`date`
- Branch: !`git branch --show-current`
- Arquivos modificados: !`git status --short`
- Dicas na página: !`grep -c '^  { n:[0-9]*, ch:' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`

## O que fazer

Apresentar os quatro valores acima ao usuário, em quatro linhas, sem rodeios.

Não chamar o Bash para reconferir nenhum deles: se o valor está aí, ele já é o valor
de agora. Se algum vier vazio, dizer qual veio vazio em vez de tentar descobrir o porquê.

## Lessons Learned

- **O `date` do Git Bash no Windows vem com espaços no meio.** A saída real foi
  `Wed Sep 16 12:38:04     2026`, com um bloco de espaços antes do ano. Apresentar a
  data normalizada ao usuário, sem reproduzir o espaçamento bruto, e nunca tratar a
  string como parseável por posição de caractere.

**Manutenção desta seção.** Ao terminar uma execução em que algum valor injetado veio
num formato inesperado, registrar aqui o formato real observado.
