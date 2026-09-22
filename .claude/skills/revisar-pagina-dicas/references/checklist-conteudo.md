# Checklist de conteúdo (camada 3)

Carregado sob demanda, só quando a revisão for de conteúdo e não apenas estrutural.

## Por dica

- O `title` é a tradução da dica, não uma paráfrase livre do livro.
- O `orig` reproduz o título em inglês exatamente como aparece no sumário.
- O `body` responde a três perguntas: o que fazer, por que funciona, e qual armadilha evita.
- O `code`, quando existe, é copiável e roda como está — sem placeholders do tipo `<seu-valor>`
  que não estejam explicados na frase anterior.
- O `lang` do snippet descreve o destino real do trecho (`SKILL.md`, `CLAUDE.md`, `terminal`),
  não a linguagem de sintaxe.
- O `note` carrega um "Claude Code Insight" do livro ou um cruzamento com outra dica,
  nunca uma repetição do `body`.
- O `docs` aponta para a página citada no fim da dica no livro, quando o livro citou uma.

## Entre dicas

- Dicas que se referenciam (11 e 18, por exemplo) fazem isso pelo `note`, e o número citado existe.
- Nenhuma dica em português usa segunda pessoa para descrever o comportamento do Claude
  quando o livro descreve o comportamento do agente em terceira.
