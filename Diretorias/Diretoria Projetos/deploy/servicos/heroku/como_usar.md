---
order: 1
icon: plug
label: "Fazendo deploy (Rails | assets estáticos)"
---

<!-- Ultima atualização: 22/09/2023 -->
<!-- Autor(es): Artur Padovesi e Pedro Augusto Ramalho Duarte -->

## Configurando o projeto

!!!
Esse deploy esta sendo feito com base em um projeto Rails
!!!

1. Crie uma branch chamada `heroku`, caso ainda não tenha, e faça as seguintes mudanças;
2. Crie um arquivo chamado `start.sh` na raíz do projeto com o seguinte conteúdo:

```bash
#!/bin/sh
rails db:migrate
bundle exec puma -C config/puma.rb
```

- `#!/bin/sh`: essa linha tem, por principal objetivo, informar ao shell qual intérprete deverá ser usado para a execução do script. No nosso caso, o intérprete usado será o sh, ou seja o Bourne shell.

- `rails db:migrate`: a linha de código padrão para executar as migrations no rails e garantir que todos os bancos de dados estejam atualizados e com suas informações certas

- `bundle exec puma -C config/puma.rb`: esse código funciona como um rails s, ou seja, serve para executar o puma (servidor padrão do rails) com alguns recursos a mais que o rails s por si só, não oferece.

!!!danger Caso seu projeto tenha banco de dados
É necessário ter uma instância separada de banco de dados. Se não me engano, o Heroku possui memória volátil, ou seja, alterações e criações de arquivos após o deploy podem ser perdidas.
!!!

!!!danger Caso seu projeto utilize Active Storage
Ou guarde alguma imagem enviada por upload, é necessário mudar o path no qual os arquivos ficam armazenados, escolhendo entre [cloudinary](https://cloudinary.com/documentation/ruby_rails_quickstart) e [amazon s3](https://devcenter.heroku.com/articles/active-storage-on-heroku). Execute o comando **Certo** para configurar os arquivos.
!!!

!!!
Não colocar as chaves de acesso no repositório, apenas no heroku. Na branch de produção, requisitar as variáveis de ambiente no heroku, e colocar no código com `ENV['NOME_DA_VARIAVEL']`.
!!!

```rb Errado
Cloudinary.config_from_url("cloudinary://API_KEY:API_SECRET@CLOUD_NAME")
```

```rb Certo
Cloudinary.config_from_url("cloudinary://#{ENV['CLOUDINARY_API_KEY']}:#{ENV['CLOUDINARY_API_SECRET']}@#{ENV['CLOUDINARY_CLOUD_NAME']")
```

4. Colocar as variáveis de ambiente no heroku

- Instalar a SDK do serviço escolhido;
- Executar o comando:

```rb Terminal
bundle
```

- `config/environments/production.rb`;
- `config/storage.yml`;
- `config/initializers/active_storage.rb`;
- Colocar as chaves de acesso do serviço escolhido no Heroku;

## Executando deploy do projeto

### Por CLI

!!!
Tutorial para instalar o CLI do heroku https://devcenter.heroku.com/articles/heroku-cli#download-and-install
!!!

1. Logar no Heroku e criar uma nova aplicação;
2. Na pasta do repositório do projeto, executar o comando:

```bash Terminal
heroku git:remote -a {nome_da_aplicação}
```

3. Configure o banco de dados para PostgreSQL;
4. E por fim execute o comando:

```bash Terminal
git push heroku {nome_da_aplicação}
```

### Por Dashboard

1. Acesse a dashboard da conta de projetos da Struct;
2. Crie uma nova aplicação;
3. Navegue até a aba Deploy e selecione a opção GitHub;
4. Selecione o repositório do projeto;
5. Selecione a branch `Heroku`;
6. Selecione a opção **Deploy Branch**.

### Deploy front-end com heroku

É possível usar do Heroku para fazer deploy de front-end. Basta criar um único endpoint para enviar os assets estáticos do front. Exemplo:

```js
// /package.json

{
  "scripts": {
    "build": "um comando de build qualquer", // comando para gerar os assets estáticos
    "start": "node server.js" // comando para rodar o servidor que servirá os assets
  }
}


```

```js
// /server.js

const express = require("express");
//chamamos o express para o arquivo

const { resolve } = require("path");
//para garantir que pegaremos o path exato

const app = express();
//criamos uma aplicação com o express

app.use(
  "*",
  express.static(
    resolve(
      __dirname,
      "./build" // Onde os assets estáticos do front-end estão depois de rodar build
    )
  )
);
//serve para pegarmos os itens de forma estática e servirmos o build

app.listen(process.env.PORT || 3000, (err) => {
  //usaremos a porta que o heroku definir ou a 300
  if (err) {
    return console.log(err);
  }
  //caso tenha um erro vai retornar o erro na callback
  console.log("Deu bom!");
  //no mais, esse console.log aparecerá na tela
});
```

Agora temos uma api, e então o deploy pode ser feito no Heroku.
