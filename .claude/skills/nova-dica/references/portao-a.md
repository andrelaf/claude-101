# Portão A — Reset ou Resume

Carregado quando a varredura do portão A encontra execução anterior desta dica.

Este portão vem **antes** da fase 0, e é a primeira coisa que a skill faz.

Procurar execuções anteriores desta mesma dica com `ls -d output/nova-dica/dica-NN-*`.
Quando não houver nenhuma, seguir direto para a fase 0 sem perguntar nada.

Quando houver, **STOP no portão A.** Pegar a mais recente pelo carimbo, listar o que
há dentro, dizer em qual fase ela parou, e perguntar:

```
A pasta output/nova-dica/dica-NN-YYYYMMDD-HHMMSS/ já tem artefatos de uma
execução anterior, parada na fase N.

Escolha:
- **Começar de novo.** Abro uma pasta nova, com carimbo de agora. A antiga
  fica onde está, intocada.
- **Retomar.** Mantenho tudo no lugar e sigo da fase N, pulando o que já
  tem saída.

Saída velha não é, por si só, prova de que você quer descartar trabalho em
andamento. Por isso eu pergunto em vez de decidir.
```

Esperar a escolha do usuário. Nunca resetar sozinho, nunca retomar sozinho, nem
quando a pasta anterior parecer obviamente abandonada. Perguntar é barato; comer o
trabalho do usuário é caro.

**Saídas:** `<job>/escolha.md`, com a opção escolhida, a frase do usuário copiada
literalmente e a data — no mesmo formato de bloco datado do portão B.
