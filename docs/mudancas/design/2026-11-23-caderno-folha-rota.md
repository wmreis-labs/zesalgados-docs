# Design: folha do caderno da rota

**Status:** aceita
**Data:** 2026-11-23
**História:** [Folha da rota para imprimir e levar](https://github.com/wmreis-labs/zesalgados/issues/143)

## Objetivo

A cozinha imprime a rota já salva e o motorista leva essa folha. Uma linha por pedido: apelido, quantidade de salgados e valor. O corte 1 não marca pagamento nem registra gasto.

## Fora deste desenho

Acerto na parada (NP, PIX, crédito, dinheiro) e o reflexo no pagamento do pedido. Gastos do motorista, sobra e conferência da cozinha. Campo novo na rota. Atalho de quantidade do papel (`20x2E`, `80 ON`). Cupom térmico de 80 mm.

## Onde abre

A mesma folha tem duas entradas, porque o entregador não entra no Planejamento.

| Quem | Entra por | Caminho |
| --- | --- | --- |
| Cozinha | Planejamento, na rota salva | `/deliveries/{id}/caderno` |
| Motorista | Rota, na rota selecionada | `/driver-route/{id}/caderno` |

`{id}` é a rota já salva. Quem não vê a área da entrada não vê a folha.

## O que a pessoa vê

Cabeçalho: Caderno, o dia da rota, o nome do motorista e o botão Imprimir.

Tabela, na ordem da rota:

| Coluna | O que mostra |
| --- | --- |
| Ordem | A ordem da parada |
| Cliente | O apelido. Sem apelido, o nome. Sem os dois, "Cliente" |
| Salgados | A quantidade de salgados do pedido |
| Valor | O valor da parada |

A quantidade de salgados usa a mesma conta do detalhe do pedido e da comanda: item com "pct N" vale N por unidade vendida, "cento" vale 100, o resto vale 1. Bebida e o que a comanda não conta como salgado ficam de fora da coluna. O valor é o total da parada que a rota já gravou.

No fim, a soma dos salgados e a soma dos valores.

Rota sem parada: a folha diz que não há pedido para levar e não oferece impressão.

## Impressão

Imprimir abre a impressão do navegador, em folha, sem o menu e sem o botão. Não é o cupom de 80 mm.

## Contrato

A folha só lê a rota já salva. Não grava acerto e não recalcula o valor. A quantidade de salgados sai dos itens que a rota já devolve, com a conta descrita acima.
