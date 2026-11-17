# WMS Almoxarifado – Requisitos

Autor: Luan Vinicius

## Visão geral

Sistema web (PWA) para controlar o almoxarifado: saldo por endereço, entrada (OD), saída para a produção (OP), transferências, ressuprimento e inventário. O celular funciona como coletor, lendo QR Codes com a câmera.

Problemas reais que o sistema resolve:

- Saldo errado: o sistema mostra saldo, mas o item não está na prateleira (ou o contrário)
- Prateleira difícil de achar
- Falta de produto para a produção quando o navio atrasa

Tecnologias previstas: React, back-end com API, PostgreSQL.

## Glossário

| Termo | Significado |
| --- | --- |
| Endereço | Posição na prateleira, com QR Code (ex.: 001-015-05) |
| UC | Unidade de Carga: caixa ou pallet com etiqueta, produto e quantidade |
| OD | Ordem de Depósito: guardar o material recebido num endereço |
| OP | Ordem de Produção: pedido da produção com itens e quantidades |
| Ressuprimento | Pedido de reposição quando o saldo fica abaixo do mínimo |
| Inventário | Contagem física para comparar com o saldo do sistema |
| Ajuste | Correção negativa do saldo depois do inventário |

## Requisitos funcionais

| Código | Grupo | Requisito |
| --- | --- | --- |
| RF01 | Movimentação | Registrar entrada e saída de itens (quem fez, quando e quantidade) |
| RF02 | Inventário | Fazer contagem física e comparar com o saldo do sistema |
| RF03 | Inventário | Mostrar lista de divergências entre sistema e contagem |
| RF04 | Endereços | Cada produto fica num endereço (rua, prateleira, nível) |
| RF05 | Endereços | Pesquisar produto por nome ou código e ver onde ele está |
| RF06 | Ressuprimento | Definir estoque mínimo para cada produto |
| RF07 | Ressuprimento | Saldo abaixo do mínimo cria automaticamente uma solicitação de ressuprimento |
| RF08 | Ressuprimento | Cadastrar pedidos de compra com previsão de chegada e status (em trânsito, atrasado, recebido) |
| RF09 | Inventário | Prateleira vazia informada pelo operador cria uma tarefa de inventário |
| RF10 | Ressuprimento | Mostrar ressuprimentos e inventários pendentes |
| RF11 | Endereços | Cada endereço tem um QR Code com o código da posição |
| RF12 | Coletor | Ler o QR Code com a câmera do celular e mostrar produto e saldo do endereço |
| RF13 | Movimentação | Transferir saldo de um local para outro (ex.: almoxarifado → produção) |
| RF14 | Movimentação | Registrar cada transferência no histórico (quem, quando, quanto, origem, destino) |
| RF15 | UC e OD | Cadastrar UC com QR Code, produto e quantidade |
| RF16 | UC e OD | Criar OD para guardar o material recebido num endereço |
| RF17 | UC e OD | Confirmar a OD bipando a UC e o endereço (o saldo entra) |
| RF18 | OP | Criar OP com a lista de itens e quantidades |
| RF19 | OP | Separar a OP bipando endereço/UC, mostrando o que já foi separado e o que falta |
| RF20 | Ressuprimento | Confirmar a OD de um produto fecha o ressuprimento dele |
| RF21 | Inventário | Físico menor que o sistema gera ajuste negativo |
| RF22 | Inventário | Cada ajuste tem um código de motivo (01 perda, 02 erro de lançamento, 03 avaria) |
| RF23 | Inventário | Físico maior que o sistema: o excedente vira nova UC, na mesma posição ou numa nova |

## Regras de negócio

| Código | Regra |
| --- | --- |
| RN01 | Não pode dar saída maior que o saldo |
| RN02 | Toda movimentação precisa ter um responsável |
| RN03 | Não pode existir duas solicitações de ressuprimento abertas para o mesmo produto |
| RN04 | Não pode transferir mais do que o saldo da origem |
| RN05 | Só pode confirmar a OD se bipar a UC e o endereço certos |
| RN06 | A OP só fica concluída quando todos os itens forem separados |
| RN07 | Todo ajuste precisa de motivo e responsável |
| RN08 | O ajuste fica salvo no histórico e nunca é apagado |
| RN09 | Ajuste de inventário só pode ser negativo |

## Fluxos principais

**Entrada (material do fornecedor ou navio)**

1. O material chega e é cadastrado como UC
2. O sistema cria a OD
3. O operador bipa a UC e o endereço
4. O saldo entra na posição e o ressuprimento do produto é fechado

**Saída para a produção**

1. A produção cria a OP com os itens e quantidades
2. O operador bipa o endereço ou a UC e separa os itens
3. O saldo é transferido do almoxarifado para a produção
4. A OP fica concluída quando todos os itens forem separados

**Inventário**

1. A tarefa de inventário é criada (prateleira vazia ou divergência)
2. O operador conta o físico e informa a quantidade
3. Físico menor que o sistema: ajuste negativo com código de motivo (ex.: sistema 50, físico 20, ajuste de −30)
4. Físico maior que o sistema: o excedente vira nova UC, guardada na mesma posição ou numa nova (ex.: sistema 20, físico 50, nova UC com 30)
