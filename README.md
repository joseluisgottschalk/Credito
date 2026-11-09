# Avaliação de clientes por pontualidade

Dashboard em um único arquivo HTML (`index.html`) que avalia clientes e redes de lojas pela pontualidade de pagamento, a partir do financeiro de clientes e dos pedidos.

## Uso rápido
1. Abra o `index.html` (direto no navegador ou publicado no GitHub Pages).
2. Em **Configurar endereços**, cole uma vez a URL do webhook n8n da base financeira e a dos pedidos. Ficam salvas no navegador.
3. Clique em **Atualizar tudo**. Em um clique a página:
   - busca a base financeira no n8n e recalcula as notas;
   - busca os pedidos, identifica o cliente de cada um e avalia só esses clientes (redes de lojas incluídas);
   - consulta a data de fundação dos clientes sem histórico.
4. Opcional: marque **Atualizar tudo ao abrir a página** para não precisar clicar.

Também dá para enviar um arquivo no cartão **Base de dados** (planilha `.xlsx`/`.csv` ou o JSON do n8n) e usar os botões **Atualizar só a base** e **Buscar pedidos e avaliar** separadamente.

## Usar em qualquer estação da rede
O `localhost` só existe na própria máquina; por isso a página precisa ser aberta a partir de um servidor da rede.
1. **Hospede o dashboard** num computador que fique ligado (de preferência o mesmo do n8n). Copie `index.html`, `servidor-dashboard.js` e `iniciar-dashboard.bat` para uma pasta nele e dê dois cliques em `iniciar-dashboard.bat` (precisa do [Node.js](https://nodejs.org), que o n8n instalado por npm já tem). A janela mostra os endereços, por exemplo `http://192.168.0.10:8080`. Deixe-a aberta. Se a 8080 estiver ocupada por outro programa, o script usa sozinho a próxima porta livre (8081, 8082...) e mostra qual é. Se o dashboard já estiver rodando, ele avisa em vez de dar erro. Para escolher a porta inicial: `node servidor-dashboard.js 9000`.
2. **Nas estações**, abra esse endereço no navegador. Os endereços `localhost` dos webhooks são trocados automaticamente pelo endereço do servidor (o n8n deve estar no mesmo computador que hospeda a página; se não estiver, troque o endereço em *Configurar endereços*).
3. **Libere as portas** no firewall do servidor (8080 do dashboard e 5678 do n8n). Se o n8n roda em Docker, publique a porta para a rede (`-p 5678:5678`, e não `-p 127.0.0.1:5678:5678`).
4. Cada estação guarda a sua própria base no navegador. Clique em **Atualizar tudo** (ou marque *Atualizar tudo ao abrir a página*).

Atenção: os webhooks do n8n não têm senha, então qualquer computador da rede que conheça o endereço consegue consultar os dados financeiros. Se isso for um problema, proteja o Webhook com autenticação ou restrinja por IP.

## Webhooks n8n
Base financeira e pedidos: Webhook (GET ou POST, resposta pelo nó **Respond to Webhook**) → ... → **Respond to Webhook** com *All Incoming Items* (uma lista JSON, um objeto por linha).
- Use a URL de **produção** (`/webhook/...`) com o workflow ativo. A URL de teste (`/webhook-test/...`) responde uma vez por clique em "Execute workflow".
- Base financeira, colunas: `Cd_empresa`, `Nome_completo`, `Nf`, `Complemento`, `Valor`, `Saldo`, `Dt_vencimento`, `Dt_ultima_liqui`; opcional `Cd_centralizado` (reúne clientes em rede de lojas).
- Pedidos: uma linha por pedido. A página reconhece sozinha os campos de cliente (`Cd_empresa`, `Cd_cliente`, `Cod_cliente`...), pedido, data, CNPJ, representante (fantasia), regional, valor, condição de pagamento (`Cond_pgto`, `Condicao_pagamento`, `Prazo`...) e observação. O que ela reconheceu aparece em **Campos do pedido** e dá para trocar qualquer campo ali (a escolha fica salva). Se o mesmo número de pedido vier em mais de uma linha, a primeira é usada.

## Aprovar pedidos e exportar em PDF
- Cada pedido aparece em **Pedidos para liberação**, com a nota e o parecer do cliente (ou da rede de lojas), o **valor do pedido** e a **condição de pagamento**. Os dois campos também aparecem na análise do cliente, junto com o total em aberto + pedido.
- Cada pedido tem os botões **Aprovar** e **Reprovar** (clicar de novo desfaz). Há filtros por Pendentes, Aprovados e Reprovados. A decisão também pode ser dada na análise do cliente.
- **Exportar aprovados (PDF)** gera um PDF A4 paisagem com: Pedido, Data, Cliente, CNPJ, Representante (campo Fantasia), Regional, Valor do pedido, Observação e, no final, as colunas em branco **Qtd. volumes** e **Nº NF** para preencher. O PDF é gerado no próprio navegador, sem biblioteca externa, e funciona offline.
- As decisões ficam salvas **neste navegador** (cada estação tem as suas) e voltam ao buscar os pedidos de novo. O PDF contém os pedidos aprovados que a consulta atual devolve. **Limpar decisões** zera tudo.

## Data de fundação dos clientes sem histórico
- **Padrão (campo em branco):** consulta direta na Receita Federal via [BrasilAPI](https://brasilapi.com.br), pelo **CNPJ do pedido** (o JSON de pedidos precisa trazer `cnpj`, `nr_cnpj` ou `cgc`). Sem token, 3 consultas simultâneas, repete automaticamente se houver limite de uso. Usa a data de início de atividade do CNPJ; para filiais, consulte pelo CNPJ da matriz para aproximar a fundação.
- **Opcional (campo preenchido):** webhook n8n próprio, por exemplo com Econodata. A página envia `POST` com `{"clientes":[{"codigo":"...","cnpj":"...","nome":"..."}]}` em lotes de 50 e espera uma lista com `codigo` (ou `cnpj`) e a data (`data_fundacao`, `dt_fundacao`, `data_abertura`, `data_inicio_atividade`...; formatos `AAAA-MM-DD`, ISO ou `DD/MM/AAAA`), e opcionalmente `situacao`. O token fica no n8n, nunca no navegador.
- Datas encontradas ficam em cache no navegador (não consulta duas vezes). Empresas com menos de 2 anos são marcadas como "empresa recente".

## Privacidade
- Os dados são lidos no navegador e **não passam por servidor de terceiros**, exceto a consulta de CNPJ à BrasilAPI (envia só o número do CNPJ) quando esse modo está ativo.
- A base carregada fica salva no navegador de quem usou (IndexedDB); **Limpar base salva** remove.
- **Não** coloque planilhas nem dados de clientes no repositório se ele for público.

## Dependências
- [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) 0.18.5 (cdnjs), para ler planilhas.


## Colunas do PDF de pedidos aprovados
Pedido, Data, Cliente, CNPJ, Representante, Regional, UF, Valor do pedido, Fat., Observação, Qtd. volumes e Nº NF (as duas últimas em branco para preenchimento).

- **UF**: lida do campo `UF` do pedido.
- **Fat.**: baseada em `Total_faturamento` — **B** = igual a zero, **N** = diferente de zero (em branco se o campo não vier). A legenda aparece no cabeçalho do PDF.
- Os campos são detectados automaticamente e podem ser ajustados em *Campos do pedido*.

## Aprovação de pedidos gravando no banco (opcional)
Preencha **Aprovação de pedidos** em *Configurar endereços* com a URL de produção (`/webhook/aprovacao-pedido`) do workflow importado de `n8n-aprovacao-pedido.json`.

- **Aprovar** grava `controle = '19'` e **Reprovar** grava `controle = '06'` em `fapedido` e `faitempe`.
- O dashboard envia `pedido`, `empresa` (campo mapeado em *Campos do pedido* > Empresa do pedido), `decisao` e `cliente`. O n8n define o valor do controle a partir da decisão e valida os dados antes de usar no SQL (parâmetros, sem concatenar texto).
- Só marca como aprovado/reprovado na tela se o banco confirmar. Se falhar, aparece o motivo e a decisão não é aplicada.
- Uma decisão já gravada só muda clicando na outra opção (não há "desfazer" que volte o banco ao estado anterior).
- Em branco, o comportamento é o antigo: decisão só local.
- Confira em `aprovacao-pedido.sql` os nomes das colunas (`nr_pedido`, `cd_empresa`) e o tipo do campo `controle`. Teste antes em homologação.
