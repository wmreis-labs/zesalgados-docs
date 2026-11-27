# Design: NF-e no pedido

**Status:** rascunho
**Data:** 2026-11-22
**ADR:** [Emissor fiscal próprio](https://wmreis-labs.github.io/wmreis-docs/mudancas/adr/2026-11-20-emissor-fiscal-proprio/)
**PRD:** [Emitto, emissor fiscal](https://wmreis-labs.github.io/wmreis-docs/mudancas/prd/2026-11-20-emitto-emissor-fiscal/)
**Serviço:** [Emissor fiscal (Emitto) e NF-e do Zé](https://wmreis-labs.github.io/airtestto-docs/mudancas/design/2026-11-22-emissor-fiscal-emitto/)

## Objetivo

Quem fecha o pedido no PDV ou cuida dele em Pedidos emite a NF-e daquele pedido, vê o estado que a API devolve, abre o DANFE e, quando precisa, cancela ou corrige. A tela não calcula imposto, não escolhe CFOP nem NCM e não fala com o Emitto: chama as rotas do pedido no `sales` e mostra o bloco `fiscal` que o pedido devolve.

Cobre a tarefa [zesalgados#111](https://github.com/wmreis-labs/zesalgados/issues/111).

## Fora deste desenho

- NFC-e, que é a nota do cliente sem cadastro (só CPF ou nenhum documento). Fica para a fase 2.
- Cadastro fiscal (natureza, item fiscal, mapeamento): fica no console do Emitto.
- Emitir nota sem pedido, nota de devolução, nota de várias vendas juntas.
- Envio da nota por WhatsApp ou e-mail.

## Onde aparece

### No PDV, ao finalizar

O modal de finalizar ganha a opção **Emitir NF-e**, desligada por padrão. Ela só aparece quando o cliente do pedido tem cadastro (não é o cliente genérico). Para o cliente genérico continua o campo "CPF/CNPJ na nota (opcional)" de hoje, sem NF-e.

Com a opção ligada, o PDV finaliza o pedido como hoje e, quando o pedido fica pago, chama a emissão. O caixa segue para a próxima venda; o resultado aparece num aviso discreto no canto ("NF-e 25 autorizada" ou "NF-e do pedido 1234 rejeitada, ver pedido") e no pedido. O PDV não espera a SEFAZ.

### No detalhe do pedido

Ao lado do botão **Cupom**, um bloco **NF-e** mostra o estado:

| Estado do bloco `fiscal` | O que a tela mostra | Ações |
| --- | --- | --- |
| Sem nota | "Sem NF-e" | **Emitir NF-e** (só com pedido pago) |
| `processando` | "Enviando à SEFAZ…" com indicador | Nenhuma; a tela consulta o pedido a cada 3 s por até 1 min e depois a cada 30 s |
| `autorizado` | "NF-e nº 25, série 2" e data; chave com botão copiar | **DANFE**, **XML**, **Carta de correção**, **Cancelar** (enquanto a API disser que pode) |
| `rejeitado` | A explicação em português que a API devolve e o campo afetado; o motivo original da SEFAZ em texto menor | **Corrigir cadastro** (leva à tela do cliente ou do produto, conforme o campo) e **Emitir de novo** |
| `cancelado` | "NF-e nº 25 cancelada" e a data | **XML** e **DANFE** |

Quando o emitente está no modo simulado, o bloco tem uma etiqueta **Simulação** e o DANFE sai com a tarja da API.

### Na lista de Pedidos

Uma coluna estreita **NF** com um ícone por estado (sem nota, processando, autorizada, rejeitada, cancelada). O filtro ganha "NF-e rejeitada", para ninguém esquecer uma nota pendente.

## Emitir

1. **Emitir NF-e** chama `POST /sales/orders/{orderId}/fiscal/nfe` sem corpo.
2. Resposta 422 traz a lista do que falta no destinatário (CPF/CNPJ, endereço, município, indicador de IE) ou o item sem item fiscal. A tela abre um diálogo com a lista:
   - destinatário incompleto: botão **Completar cadastro**, que abre o formulário do cliente já no endereço; ao salvar, volta ao pedido e emite;
   - item sem item fiscal: mensagem para avisar o administrador, sem ação no Zé.
3. Resposta 409 mostra a mensagem da API (pedido não pago, nota já emitida).
4. Resposta 412 mostra "Emissão de NF-e não configurada para esta empresa" ou "Certificado vencido".
5. Sucesso mostra o bloco em `processando`.

O cadastro do cliente ganha, no endereço, o município com o código IBGE (escolhido numa lista pela UF e pelo nome, preenchido pelo CEP quando der) e o indicador de IE (contribuinte, isento, não contribuinte), com a IE quando for contribuinte. Esses campos vêm do `person` ([airtestto#307](https://github.com/wmreis-labs/airtestto/issues/307)).

## Cancelar e corrigir

- **Cancelar** abre um diálogo com a justificativa (mínimo 15 caracteres, contador) e o aviso de que o cancelamento só vale até 24 horas depois da autorização. Confirmar chama `.../fiscal/nfe/cancel`.
- **Carta de correção** abre um diálogo com o texto (15 a 1000 caracteres) e a lista do que ela não corrige: valor, imposto, quantidade, destinatário e data. Mostra a última carta enviada, se houver, porque a nova substitui a anterior. Confirmar chama `.../fiscal/nfe/correction`.

Os dois botões só aparecem para quem tem o papel GestorCozinha. O Pdv emite e consulta.

## Arquivos

**DANFE** e **XML** chamam `GET .../fiscal/nfe/files` na hora do clique e abrem a URL devolvida (o DANFE numa aba nova, o XML como download). A URL vence em 5 minutos; a tela nunca a guarda.

## Erros e carregamento

- Falha de rede ao emitir não repete sozinha: a tela mostra **Tentar de novo**. A API não duplica a nota se o pedido já tiver chegado.
- O bloco usa o mesmo carregamento do pedido; não há chamada própria para o estado.

## Critério de aceite

Contra a API de dev, com o emitente no modo simulado e depois em homologação: um pedido pago de cliente cadastrado emite NF-e pelo PDV e pelo detalhe; cliente sem endereço é levado ao cadastro e, ao voltar, a nota sai; a rejeição simulada aparece explicada e o reenvio funciona; DANFE e XML abrem; cancelamento e carta de correção funcionam para GestorCozinha e não aparecem para Pdv; a lista mostra o estado e filtra as rejeitadas.
