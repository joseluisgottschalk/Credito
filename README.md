# Avaliação de clientes por pontualidade

Dashboard em um único arquivo HTML (`index.html`) que avalia clientes e redes de lojas pela pontualidade de pagamento, a partir da planilha de financeiro de clientes.

## Publicar no GitHub Pages
1. Crie um repositório no GitHub e envie o `index.html` (e este README).
2. Em **Settings → Pages**, escolha **Deploy from a branch**, branch `main`, pasta `/ (root)`.
3. Abra o endereço que o GitHub mostrar (`https://SEU-USUARIO.github.io/NOME-DO-REPO/`).

Também funciona abrindo o `index.html` direto no navegador.

## Como usar
1. No cartão **Base de dados**, envie a planilha (`.xlsx`, `.xls`, `.xlsm` ou `.csv`).
2. Colunas necessárias: `Cd_empresa`, `Nome_completo`, `Nf`, `Complemento`, `Valor`, `Saldo`, `Dt_vencimento`, `Dt_ultima_liqui`.
3. Coluna opcional: `Cd_centralizado`. Quando preenchida, os clientes com o mesmo código são avaliados juntos como uma rede de lojas.
4. Busque clientes por nome ou código, ou cole uma lista de vários de uma vez.

## Privacidade
- A planilha é lida no navegador e **não é enviada a nenhum servidor**.
- A base carregada fica salva no navegador de quem usou (IndexedDB). O botão **Limpar base salva** remove.
- **Não** coloque a planilha nem dados de clientes no repositório se ele for público.

## Dependências
- [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js) 0.18.5, carregado do cdnjs, para ler a planilha.
