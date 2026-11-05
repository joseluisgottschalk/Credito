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
1. **Hospede o dashboard** num computador que fique ligado (de preferência o mesmo do n8n). Copie `index.html`, `servidor-dashboard.js` e `iniciar-dashboard.bat` para uma pasta nele e dê dois cliques em `iniciar-dashboard.bat` (precisa do [Node.js](https://nodejs.org), que o n8n instalado por npm já tem). A janela mostra os endereços, por exemplo `http://192.168.0.10:8080`. Deixe-a aberta. Outra porta: `node servidor-dashboard.js 8081`.
2. **Nas estações**, abra esse endereço no navegador. Os endereços `localhost` dos webhooks são trocados automaticamente pelo endereço do servidor (o n8n deve estar no mesmo computador que hospeda a página; se não estiver, troque o endereço em *Configurar endereços*).
3. **Libere as portas** no firewall do servidor (8080 do dashboard e 5678 do n8n). Se o n8n roda em Docker, publique a porta para a rede (`-p 5678:5678`, e não `-p 127.0.0.1:5678:5678`).
4. Cada estação guarda a sua própria base no navegador. Clique em **Atualizar tudo** (ou marque *Atualizar tudo ao abrir a página*).

Atenção: os webhooks do n8n não têm senha, então qualquer computador da rede que conheça o endereço consegue consultar os dados financeiros. Se isso for um problema, proteja o Webhook com autenticação ou restrinja por IP.

## Webhooks n8n
Base financeira e pedidos: Webhook (GET ou POST, resposta pelo nó **Respond to Webhook**) → ... → **Respond to Webhook** com *All Incoming Items* (uma lista JSON, um objeto por linha).
- Use a URL de **produção** (`/webhook/...`) com o workflow ativo. A URL de teste (`/webhook-test/...`) responde uma vez por clique em "Execute workflow".
- Base financeira, colunas: `Cd_empresa`, `Nome_completo`, `Nf`, `Complemento`, `Valor`, `Saldo`, `Dt_vencimento`, `Dt_ultima_liqui`; opcional `Cd_centralizado` (reúne clientes em rede de lojas).
- Pedidos: o campo do cliente é reconhecido automaticamente (`Cd_empresa`, `Cd_cliente`, `Cod_cliente`, `Cliente`...); se não for, a página mostra os campos e você escolhe. Aproveita também valor do pedido, número, nome e `cnpj`.

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
