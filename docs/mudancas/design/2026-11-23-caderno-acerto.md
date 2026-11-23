# Design: acerto e gastos no caderno

**Status:** aceita
**Data:** 2026-11-23
**História:** [Acerto do que o motorista recebeu na parada](https://github.com/wmreis-labs/zesalgados/issues/148)
**Contrato:** [Acerto da parada e fechamento da rota](https://wmreis-labs.github.io/airtestto-docs/mudancas/design/2026-11-23-caderno-acerto-rota/)

## Objetivo

Na mesma folha do caderno, o motorista marca o que recebeu em cada parada e os gastos da rota. A cozinha confirma a parada e confere a rota. A folha mostra o dinheiro, a sobra e o cartão que a API devolve.

## O que a pessoa vê

A tabela continua com ordem, apelido, salgados e valor. Quando a parada já tem marca, a linha do cliente também mostra a forma, o valor marcado e, se a cozinha confirmou, a palavra confirmada.

Embaixo da tabela, fora da impressão, cada parada tem:

- Recebeu: não pago, PIX, crédito ou dinheiro
- Valor recebido, escondido quando a marca é não pago
- Motivo, quando não for o valor todo
- Gravar
- Confirmar, só para a cozinha, e só depois que a marca existe

No fim, os gastos: comissão, marmita, combustível e outros. Gravar gastos. A cozinha vê Conferir rota.

Quando a API já devolveu o resumo, a folha mostra dinheiro arrecadado, cartão à parte e sobra. Rota conferida aparece quando a cozinha conferiu. Depois disso os campos ficam travados.

A impressão continua sem menu e sem os campos. A marca gravada sai na linha do cliente. O resumo sai no fim.

## Quem faz o quê

O motorista grava a marca e os gastos. A cozinha confirma a parada e confere a rota. O total do pedido não se edita nesta tela.
