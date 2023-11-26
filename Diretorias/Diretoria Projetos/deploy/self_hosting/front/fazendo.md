---
order: 1
icon: rocket
label: "Fazendo deploy de frontend"
---

<!-- Ultima atualização: 23/09/2023 -->
<!-- Autor(es): Artur Padovesi -->

## Considerações

Considerando que nenhum framework, e nem SSR, está sendo usado, a aplicação React não inclui servidor. No ambiente de **desenvolvimento** são usadas **ferramentas** (como vite, create-react-app, etc) para servir a aplicação na porta determinada (na Struct determinamos como 3000), mas no deploy utilizamos o Express, um servidor web que pode ser usado para servir arquivos estáticos, como imagens, css, js, etc. Sendo assim, após fazer o build da aplicação React, o Express é que estará servindo os _assets_ estáticos na porta definida.

!!!danger Errado
Estamos usando Express para fazer o deploy!
!!!

## Configurando o projeto

!!!
Considere a [branch production do projeto jeira](https://github.com/StructCE/jeira-frontend/tree/production) como um exemplo.
!!!

## Criando a branch production

Crie a branch `production` a partir da `main` ou da `development`. **Esteja** localmente **na** branch **`production`**

1. Criar um arquivo `.dockerignore` para garantir que o container gere certos arquivos, ao invés de usar as suas locais. É importante retirar possíveis fontes de problemas. Colocar o seguinte conteudo dentro do arquivo criado:

```dockerignore
/node_modules
/dist
/build
```

2. Adicione o pacote `express` ao projeto. Crie um arquivo `sever.js` com o seguinte conteúdo:

```js
const express = require("express");
//chamamos o express para o arquivo

const { resolve } = require("path");
//para garantir que pegaremos o path exato

const app = express();
//criamos uma aplicação com o express

app.use("/static", express.static(resolve(__dirname, "./build/static")));

app.use(
  "/asset-manifest.json",
  express.static(resolve(__dirname, "./build/asset-manifest.json"))
);

app.use(
  "/favicon.ico",
  express.static(resolve(__dirname, "./build/favicon.ico"))
);

app.use(
  "/index.html",
  express.static(resolve(__dirname, "./build/index.html"))
);

app.use(
  "/manifest.json",
  express.static(resolve(__dirname, "./build/manifest.json"))
);

app.use(
  "/robots.txt",
  express.static(resolve(__dirname, "./build/robots.txt"))
);

app.use("*", express.static(resolve(__dirname, "./build")));
//serve para pegarmos os itens de forma estática e servirmos o build

app.listen(80, (err) => {
  //usaremos a porta que o heroku definir ou a 300
  if (err) {
    return console.log(err);
  }
  //caso tenha um erro vai retornar o erro na callback
  console.log("Deu bom!");
  //no mais, esse console.log aparecerá na tela
});
```

!!!warn
Eu sei que existe uma maneira melhor de fazer isso, mas desse jeito funciona, e como nossa aplicação é um SPA, então **não funciona** só fazer o seguinte:

```js
app.use("*", express.static(resolve(__dirname, "./build")));
```

Pois o `index.html` pede pelos arquivos `favicon.ico`, `static/*.js` e `static/*.css`. Nosso servidor serviria recursivamente o diretório inteiro da build para cada uma das requisições pedindo um único arquivo.

Além disso, ter um rota _catch all_ (`*`) é necessário pois, como temos um SPA, queremos que `/` e `/users/profile/1` retornem os mesmo arquivos (todos os da build).

!!!

3. Adicione o arquivo `start.sh` com o comando que deve ser rodado ao inicializar o container:

```bash
#!/bin/bash
bun server.js
```

4. Adicionar o Dockerfile de frontend:

```dockerfile
# Imagem de Origem
FROM oven/bun as build

RUN apt-get -y update

# # if using Create React App, uncomment:
# RUN apt-get -y install nodejs
# # needed so build using react-scripts works

WORKDIR /app

COPY bun.lockb .
COPY package.json .
RUN bun install

COPY . .
RUN bun run build


FROM oven/bun as start

WORKDIR /app

# copying only needed files to end image
COPY --from=build /app/build/. ./build
COPY --from=build /app/server.js ./server.js
COPY --from=build /app/start.sh ./start.sh

RUN chmod +x ./start.sh
CMD ["./start.sh"]
```

!!!
Estamos usando bun pra instalar e rodar as coisas porque, no momento, bun é mais rápido que npm, e npm era lento que incomodava fazer a build do projeto.
!!!

### Mudando as urls de localhost

!!!
A aplicação React usa urls locais para acessar a API, por exemplo, `http://localhost:3333/api/v1`. Essas urls devem ser alteradas para as urls de produção, por exemplo, `https://api.struct.com.br/api/v1`.É possível fazer isso usando variáveis de ambiente, mas no momento deve ser trocado manualmente, como no nosso repositório de exemplo (talvez esse gitbook esteja desatualizado em relação ao repositório, verifique).
!!!

### Mudando o `index.html`

1. Alterar o arquivo `index.html` para conter informações corretas sobre a aplicação, bem como os metadados.
2. Criar um `robots.txt`, para ajudar os mecanismos de busca, além indexar o site conforme necessario.
3. Colocar o título correto, colocar descrição, mudar o favicon e a linguagem para pt-BR.

## Criando a imagem

[Cr]

## Criando container

1. Atualize o repositório de docker_compose da Struct com o seguinte comando:

```bash Terminal
git pull
```

2. Crie uma pasta com o nome do projeto.
3. Modifique o template de `docker-compose.yml` do Traefik com os nomes que podem ser usados para identificar o projeto nos logs, caso ocorra algum erro.
4. Definir a imagem que será usada com o valor de `image`.
5. Alterar os valores de `environment`, `restart`, `volumes`, e `networks`.
6. Crie o container usando o comando:

```bash Terminal
`docker-compose up -d`
```
