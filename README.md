# Avaliação de clientes por pontualidade

Dashboard em um único arquivo HTML (`index.html`) que avalia clientes e redes de lojas pela pontualidade de pagamento, a partir da planilha de financeiro de clientes.

## Publicar no GitHub Pages
1. Crie um repositório no GitHub e envie o `index.html` (e este README).
2. Em **Settings → Pages**, escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`.
3. Abra o endereço que o GitHub mostrar (`https://SEU-USUARIO.github.io/NOME-DO-REPO/`).

Também funciona abrindo o `index.html` direto no navegador.

## Como usar
1. Clique em **Atualizar dados** para executar a rotina no n8n e trazer o resultado (veja a configuração abaixo), **ou** envie um arquivo no cartão **Base de dados**: planilha (`.xlsx`, `.xls`, `.xlsm`, `.csv`) ou o JSON gerado pelo n8n (`.json` / `.txt`).
2. Colunas necessárias: `Cd_empresa`, `Nome_completo`, `Nf`, `Complemento`, `Valor`, `Saldo`, `Dt_vencimento`, `Dt_ultima_liqui`.
3. Coluna opcional: `Cd_centralizado`. Quando preenchida, os clientes com o mesmo código são avaliados juntos como uma rede de lojas.
4. Busque clientes por nome ou código, ou cole uma lista de vários de uma vez.

## Configurar o botão "Atualizar dados" (n8n)
No workflow que gera a base:
1. Troque o gatilho (ou acrescente um) por um nó **Webhook**. Método `GET` (ou `POST`), campo **Respond** = *Using 'Respond to Webhook' Node*. Em **Options**, deixe **Allowed Origins (CORS)** como `*` ou coloque o endereço de onde a página roda.
2. No final do fluxo, acrescente um nó **Respond to Webhook** com **Respond With** = *All Incoming Items* (devolve a lista de parcelas, no mesmo formato do JSON atual: `Cd_empresa`, `Nome_completo`, `Nf`, `Dt_vencimento`, `Valor`, `Saldo`, `Dt_ultima_liqui`, `Cd_centralizado`...).
3. **Ative** o workflow e copie a **Production URL** do Webhook (`http://localhost:5678/webhook/...`). A *Test URL* (`/webhook-test/`) só funciona enquanto você clica em "Listen for test event".
4. Cole a URL no campo do dashboard e clique em **Atualizar dados**. A URL fica salva no navegador.

Observações:
- O n8n precisa estar rodando no mesmo computador do navegador (ou numa URL acessível a ele). Se a página for servida por HTTPS (GitHub Pages) e o n8n estiver em `http://localhost`, o Chrome pode pedir permissão de acesso à rede local.
- A resposta é grande (cerca de 90 MB); o download e o processamento levam alguns segundos.

## Avaliar clientes dos pedidos (n8n)
1. Monte no n8n uma rotina que consulte os pedidos no banco e termine com **Respond to Webhook** (*All Incoming Items*), no mesmo formato da base financeira: uma lista JSON com um objeto por pedido.
2. Cole a URL do webhook no cartão **Pedidos** e clique em **Buscar pedidos e avaliar**. Antes, carregue a base financeira (botão **Atualizar dados** ou envio de arquivo).
3. A página identifica o campo do cliente (`Cd_empresa`, `Cd_cliente`, `Cod_cliente`, `Cliente`...). Se não reconhecer, mostra os campos recebidos para você escolher, e lembra da escolha.
4. Resultado: tabela de pedidos por cliente (quantidade, valor, nota, classe, vencido, parecer), do pior para o melhor, com o detalhe completo ao clicar. Clientes que fazem parte de uma rede de lojas são avaliados pela rede. Clientes sem histórico na base aparecem numa lista à parte.
5. Campos opcionais que a página aproveita: valor do pedido (`Valor_pedido`, `Valor_total`, `Valor`...), número do pedido e nome do cliente.
6. A URL de teste (`/webhook-test/...`) só responde uma vez por clique em "Execute workflow". Para uso contínuo, ative o workflow e use a URL de produção (`/webhook/...`).

## Privacidade
- A planilha é lida no navegador e **não é enviada a nenhum servidor**.
- A base carregada fica salva no navegador de quem usou (IndexedDB). O botão **Limpar base salva** remove.
- **Não** coloque a planilha nem dados de clientes no repositório se ele for público.

## Dependências
- [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) 0.18.5, carregado do cdnjs, para ler a planilha.
