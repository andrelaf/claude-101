---
name: revisar-pagina-dicas
description: This skill should be used when the user asks to review or audit the study page 101-dicas-claude-code.html before publishing it, checking tip data integrity, theme tokens and responsive rules. For review only, never for adding new tips or publishing.
paths:
  - "**/*.html"
allowed-tools: Bash(grep *)
---

# Revisar página de dicas

Frases-gatilho que o usuário costuma digitar:

- "revisa a página antes de publicar"
- "confere se a dica nova ficou consistente"
- "auditar o HTML das dicas"

## Estado no momento da invocação

Estes valores já chegam prontos, sem chamada de ferramenta:

- Objetos no array `TIPS`: !`grep -c '^  { n:[0-9]*, ch:' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`
- Números presentes: !`grep -o "{ n:[0-9]*, ch:" "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html" | grep -o "[0-9][0-9]*" | xargs`
- Dica marcada como atual: !`grep -o 'n:[0-9]*, ch:[0-9]*, here:true' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`
- Contador do cabeçalho: !`grep -o 'gaugeCount">[^<]*' "${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"`

Tokens de cor declarados fora do `:root` puro, que é o erro mais caro da página:

```!
T=$(mktemp -d)
PAGE="${CLAUDE_PROJECT_DIR}/101-dicas-claude-code.html"
grep -o 'var(--[a-z0-9-]*)' "$PAGE" | sort -u | sed 's/var(--\(.*\))/\1/' > "$T/usados"
sed -n '/^  :root{/,/^  }/p' "$PAGE" | grep -o '\--[a-z0-9-]*:' | sed 's/^--//; s/:$//' | sort -u > "$T/declarados"
comm -23 "$T/usados" "$T/declarados"
```

Se a última lista vier vazia, o passo 4 do workflow já está aprovado.

## Purpose

Auditar `101-dicas-claude-code.html` antes de uma republicação: garantir que o array
`TIPS` está íntegro, que os tokens de cor existem nos três estados de tema e que a
página não quebra em largura de celular.

Não usar esta skill para escrever dicas novas nem para publicar. Redigir conteúdo é
trabalho manual da sessão; publicar é `/publicar-dicas`.

## Workflow

1. **Conferir o bloco de estado acima.** Precondição: os comandos rodaram e trouxeram
   números. Se vieram vazios, o arquivo mudou de nome ou de lugar: parar e avisar.
2. **Checar o array `TIPS`.** A numeração é contígua a partir de 1, sem repetição.
   No máximo um objeto carrega `here: true`, e ele é o de maior `n`. Comparar com a
   lista de números injetada, sem reler o arquivo inteiro.
3. **Checar os capítulos.** Todo `ch` referenciado em `TIPS` existe em `CHAPTERS`,
   e todo capítulo em `CHAPTERS` tem ao menos uma dica.
4. **Checar tokens de tema.** Usar a saída do bloco multilinha. Todo token listado ali
   é um token usado mas não declarado no `:root` puro, o que apaga a cor num dos temas.
5. **Checar largura de celular.** Nenhum `min-width` maior que 400px, nenhum bloco
   largo fora de um container com `overflow-x: auto`, gutter lateral preservado no
   breakpoint de 900px.
6. **Checar se o script ainda é JavaScript válido.** Extrair o conteúdo entre
   `<script>` e `</script>` para um arquivo e rodar `node --check`. Precondição:
   node disponível. Saída: `JS OK`, ou o erro de sintaxe com a linha.
7. **Checar textos da interface.** O contador injetado (`0 / N`), a linha "Cobertura
   atual" e a linha "Leitura: dica N de 101" batem com a contagem de objetos.
   Consultar `references/checklist-conteudo.md` quando a revisão for de conteúdo
   e não só estrutural.

## Rules

- Não editar o arquivo. Esta skill relata; a correção é decidida pelo usuário.
- Não rodar navegador nem screenshot: a revisão é de código-fonte.
- Não repetir com o Bash o que o bloco de estado já trouxe. O ponto da injeção é
  economizar a ida e volta.
- Ao encontrar token órfão ou numeração quebrada, parar e reportar antes de seguir
  para as checagens de estilo: as duas falhas invalidam a página.
- Nunca inventar uma dica ausente: se um número faltar na sequência, relatar a lacuna.

## Lessons Learned

Entrada nova aqui só depois de um erro que de fato aconteceu.

- **A revisão passava com a página quebrada.** Todas as checagens desta skill leem o
  arquivo com `grep`, e `grep` não sabe se o JavaScript ainda roda. Numa inserção de
  dica o `];` que fecha o array `TIPS` foi apagado junto com a edição: numeração,
  tokens e contadores continuaram todos "ok", e a página não renderizava nada. Daí o
  passo 6. Ele é a única checagem que executa o script em vez de olhar para ele.
- **Duas injeções desta skill param para pedir permissão.** A que termina em `| xargs`
  e o bloco multilinha que começa em `mktemp` não casam com `Bash(grep *)` no
  `allowed-tools`. Não é falha: é casamento por prefixo funcionando. Bloco de shell
  arbitrário deve mesmo pedir passagem.

**Manutenção desta seção.** Ao terminar uma revisão que tenha deixado passar algo,
acrescentar aqui a checagem que faltava, e o passo correspondente no Workflow.

## Output format

```
ARQUIVO: <caminho>
SINTAXE: <JS OK | erro na linha N>
DICAS: <n> objetos, numeração <ok | lacuna em X>
CAPÍTULOS: <ok | problema>
TOKENS ÓRFÃOS: <nenhum | lista>
RESPONSIVO: <ok | lista de problemas>
TEXTOS FIXOS: <ok | lista de divergências>
VEREDITO: <pronto para publicar | corrigir antes>
```
