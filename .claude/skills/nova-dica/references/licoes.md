# Lessons Learned — /nova-dica

Escritas depois de execuções reais. Entrada nova aqui só depois de um erro que
de fato aconteceu, nunca preventivamente.

- **`grep '^  { n:'` conta os capítulos junto.** O array `CHAPTERS` usa a mesma
  abertura que `TIPS`, com um espaço depois dos dois pontos. A âncora que separa é
  `, ch:`. Uma contagem errada aqui é pior que nenhuma: ela entra no bloco de estado
  como se fosse fato.
- **Editar o fim do array leva o `];` embora.** Um `old_string` que inclui o
  fechamento do array e um `new_string` que esquece de devolvê-lo apagam a página
  inteira, sem erro visível. Rodar `node --check` no script extraído depois de toda
  inserção, antes de publicar.
- **Seção nova cai onde a edição encostou, não onde ela pertence.** O Portão A
  nasceu depois da Fase 4 porque substituiu a seção Retomada. Depois de inserir uma
  seção, conferir a ordem com `grep -n '^## '`.
- **O corpo da skill pode chegar numa versão antiga.** Aconteceu três vezes, sempre
  logo depois de uma edição: o disco tinha a versão nova e a invocação entregou a
  anterior. Na terceira, o corpo injetado ainda não tinha o Portão A nem esta seção.
  Antes de seguir um corpo injetado, conferir `grep -c '^## Portão A\|^## Lessons'`
  no disco. O disco é a verdade; o corpo injetado é uma cópia que pode estar velha.
- **Bloco multilinha não casa com `Bash(grep *)`.** Injeção cujo comando começa em
  `T=$(mktemp -d)` não é pré-aprovada e para para pedir permissão. Não é falha — é
  casamento por prefixo funcionando. Injeção que precisa ser automática começa com o
  comando que está no `allowed-tools`.
- **`paths:` escondia a skill da lista, e isso não era o que se queria.** `paths:`
  carrega a skill quando o Claude **lê** um arquivo que casa, não quando o assunto é
  o arquivo — `/nova-dica` rodava, mas a skill nunca aparecia para ser escolhida
  sozinha. Como o gatilho real é uma frase do usuário ("acrescenta a dica 31"), e não
  um arquivo já tocado, o campo foi removido do frontmatter em 2026-09-21. Efeito
  colateral: a skill agora ocupa lugar na lista de toda sessão. Se um dia o custo de
  contexto pesar, o caminho não é voltar o `paths:` — é `when_to_use`.
- **O PDF do livro não abre com o Read.** A fase 1 tentou `Read` em
  `9781808650314.pdf` e levou `pdftoppm is not installed` — a renderização de página
  do Read depende de poppler-utils, que não está neste Windows. Mas `pdftotext` está,
  em `/mingw64/bin`. Extrair uma vez para o scratchpad com `pdftotext -layout` e
  trabalhar sobre o `.txt`: o sumário e as dicas ficam a um `grep -n "^Tip NN"` de
  distância, e o PDF não é reaberto.
- **Exemplo inventado passa pelo portão, não por baixo dele.** Na dica 26 o bloco de
  código usou nomes de skill que não estão no livro. O caminho certo foi declarar isso
  no portão B e registrar a ressalva no `aprovacao.md`, não decidir sozinho.
- **"Leitura: dica N de 101" não existe literal no HTML.** Na dica 35, um
  `grep 'Leitura: dica'` voltou vazio, porque a linha real é
  `Leitura: <b>dica 34 de 101</b>`. O mesmo vale para "Capítulos cobertos: <b>…</b>".
  Para achar os contadores, procurar só o rótulo (`Leitura:`, `Capítulos cobertos`)
  e não o texto que a skill cita.

**Manutenção desta seção.** Ao terminar uma execução, revisar as seções relevantes
desta skill para refletir o que foi aprendido. Toda linha de `PENDÊNCIAS` no relatório
final é candidata a virar entrada aqui. Sem essa revisão, a seção congela na versão um
e para de importar.
