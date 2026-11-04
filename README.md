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

## Privacidade
- A planilha é lida no navegador e **não é enviada a nenhum servidor**.
- A base carregada fica salva no navegador de quem usou (IndexedDB). O botão **Limpar base salva** remove.
- **Não** coloque a planilha nem dados de clientes no repositório se ele for público.

## Dependências
- [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) 0.18.5, carregado do cdnjs, para ler a planilha.
