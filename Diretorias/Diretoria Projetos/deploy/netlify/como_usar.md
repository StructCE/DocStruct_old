---
order: 1
icon: plug
label: "Fazendo deploy de assets estáticos (talvez mais?)"
---

!!!
**Só foi testado** o deploy de assets estáticos gerados por build de apps **React**. Provavelmente é **possível** hospedar uma **API**, porém **sem o Banco de Dados**. Isso porque o armazenamento do Netlify é "volátil" (não garante permanência de alterações pós deploy, pode resetar pro estado inicial a qualquer momento)
!!!

<!-- Ultima atualização: 23/09/2023 -->
<!-- Autor(es): Artur Padovesi -->

## Configurando o projeto

1. Na própria main, mude o `/public/favicon.ico` para o real ícone do projeto.
2. Mude o `index.html`, trocando os conteúdos das tags `<meta>`, `<title>` e a `lang` da tag `<html>`.
3. Pense na possibilidade de adicionar um `/public/robots.txt`.
4. Crie uma branch chamada `netlify`
5. Mude todas as referências a localhost por suas respectivas urls de produção

!!!
Se você tem uma instância axios com a url `http://localhost:3333/api/v1`, mude para `https://seu-projeto.railway.app/api/v1` ou equivalente (a url de _staging_). Se a url das imagens é `http://localhost:3333/`, mude para `https://seu-projeto.railway.app` ou equivalente.
!!!

## Roteamento Client Side

!!!
Geralmente fazemos _deploy_ de React como [_single page app_](https://en.wikipedia.org/wiki/Single-page_application), com um roteador _client side_, como react-router-dom. Sendo assim, rotas não são [_endpoints_](https://www.cloudflare.com/pt-br/learning/security/api/what-is-api-endpoint/), e sempre devem ser retornados os mesmos arquivos pra requisições, independente da rota.
!!!

1. Adicione o arquivo `_redirects` no do projeto com o seguinte conteúdo:

```
/*    /index.html   200
```

## Fazendo deploy manual

1. Vá para a branch `netlify`;
2. Rode o comando `yarn build` para gerar uma pasta `build` com os arquivos estáticos;
3. Faça login na conta de projetos da Struct;
4. Selecione a opção de _deploy_ manual (\__deploy_ manually\_);
5. Arraste e solte os arquivos da pasta `build` na área de upload;
