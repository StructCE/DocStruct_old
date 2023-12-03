---
order: 1
icon: rocket
label: "Qual a ideia de qualquer deploy?"
---

<!-- Ultima atualização: 22/09/2023 -->
<!-- Autor(es): Artur Padovesi -->

## Passos

1. Apontar o domínio para o IP do nosso servidor:
   - Isso deve ser feito no site que configura o domínio;
   - Caso o cliente já tenha um domínio para uso interno de sua empresa, provavelmente não teremos acesso. Para gerar o certificado SSL deve ser usado nosso proxy interno (traefik), podendo tomar como exemplo o deploy do Jeira;
   - Caso o cliente não tenha um domínio, será necessário que o cliente compre o domínio, e é sugerido que ele seja configurado para usar Cloudflare como proxy reverso. Com o Cloudflare é mais fácil configurar o certificado SSL;
2. Configurar o projeto no github:
   - Crie uma branch `production` e faça as alterações nessa branch. Essas alterações podem ser trocar URL's, adicionar um start.sh, etc.;
3. Criar uma imagem docker com o projeto dentro.
   - Para isso, é feito um **Dockerfile** que faz a **_build_** do projeto, copia arquivos necessários, instala o mínimo necessário para depois o projeto poder rodar no servidor.
   - Uma imagem Docker é como uma configuração inicial para **depois** uma máquina virtual inicializada com **Docker Compose**. Sendo assim, tem sistema operacional, pacotes instalados, filesystem, etc. separados.
   - É comum gerar imagens a partir de outras imagens. Por exemplo, para rodar `NodeJS` é comum se basear na imagem oficial do Node ao invés de instalar tudo na mão (Sistema Operacional, dependências do Node, etc.).
4. Criar um `docker-compose.yml` para o projeto;
   - Boa parte das seções de um `docker-compose.yml` são específicas do projeto, mas a seção de labels está sendo usada para configurar o container dentro do **traefik**;
   - Esse arquivo pode ter mais de um `service`, por exemplo uma aplicação Next e um banco de dados MySQL.

## Glossário

### Dockerfile

Dockerfile constrói uma imagem. Uma imagem é uma "foto" de uma computador em um estado. Ela inclui arquivos, sistema operacional (geralmente é usado Alpine ou um SO extremamente leve), dependências instaladas e, finalmente, o **comando de inicialização**.

Ao construir imagens, geralmente são usadas imagens intermediárias.

!!!warning
Cuidado para não copiar coisas demais para a Imagem Docker. Por exemplo, `node_modules/` não deve ser copiado do local para a imagem, mas sim o `package.json` para rodar o `pnpm install` ao construir a imagem (no Dockerfile).

Como o sistema operacional rodando dentro dele é diferente do seu, copiar arquivos instalados para a sua máquina pode criar problemas.
!!!

==- Exemplo de Dockerfile usando bun para buildar um projeto react, e rodar um servidor express para servi-lo

```Dockerfile
# Imagem de Origem
FROM oven/bun as build

RUN apt-get -y update

# # needed so react-scripts build works
# # Se seu projeto estiver usando Create React App (for legado) descomente:
# RUN apt-get -y install nodejs

# Separando a pasta com a aplicação do resto da imagem original
WORKDIR /app

# RUN git clone ${PROJETO_GITHUB} --depth=1 --branch production /app/
COPY bun.lockb .
COPY package.json .
RUN bun install

# coloque o node_modules e outras pastas a serem ignoradas no .dockerignore
# senão pode ser copiada sem querer aqui
COPY . .
RUN bun run build

# Iniciando construção da imagem final:
FROM oven/bun as start

WORKDIR /app

# Copiando somente os arquivos necessários, para diminuir ao máximo o tamanho final da imagem
# Pasta com o resultado da build:
COPY --from=build /app/build/. ./build
# Pasta com o servidor express
COPY --from=build /app/server.js ./server.js
COPY --from=build /app/start.sh ./start.sh

# Tornando o start.sh executável
RUN chmod +x ./start.sh
# declarando o comando de inicialização da imagem:
CMD ["./start.sh"]
# Como só pode existir um comando de inicialização, se queremos ter mais de um comando
# ao iniciar o container precisamos usar o start.sh
```

===

### docker-compose.yml

Esse arquivo pode ser usado com o comando `docker-compose up` ou `docker compose up`, e deve ser usada a flag `-d` quando no servidor. Ao fazer tal coisa, ele gera um **container com** o conteúdo da **imagem**, e roda o **comando de inicialização**.

Ele tem várias partes de configuração:

1. `networks`:
   - Declara as conexões que os containers tem. No servidor, temos uma conexão chamada `proxy` que conecta todo mundo, mas às vezes é necessário ter uma conexão interna também, para, por exemplo, conectar um app Next com seu banco de dados;
   - Caso queira rodar o container localmente, deve tirar o network proxy.
2. `services`:
   - É aqui que a magia acontece.
   - Declare um nome para os serviços que serão rodados, coloque o nome da imagem delas (a gerada anteriormente pelo Dockerfile, ou uma da internet);
   - Declare `volumes` caso queira sincronizar arquivo/diretório do container com o do computador. Isso é útil ao ter um banco de dados, ao guardar arquivos no container, etc.

Um exemplo do projeto Jeira:

```yml
version: "3.3"

# `dominio.qualquer.br`,`www.dominio.qualquer.br`
# my-react-app (uniq)
# my-react-app-certresolver (uniq)
# 80

networks:
  proxy:
    external: true

services:
  my-react-app:
    image: structej/projetos:my-react-app-1.0.9
    environment:
      - TZ=America/Sao_Paulo
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy

      # # http access
      - traefik.http.routers.my-react-app.rule=Host(`dominio.qualquer.br`,`www.dominio.qualquer.br`)
      - traefik.http.routers.my-react-app.entrypoints=web

      # # https access

      - traefik.http.routers.my-react-app-websecure.rule=Host(`dominio.qualquer.br`,`www.dominio.qualquer.br`)
      - traefik.http.routers.my-react-app-websecure.entrypoints=websecure
      - traefik.http.routers.my-react-app-websecure.tls=true

      # # Specify to container port
      - traefik.http.routers.my-react-app-websecure.service=my-react-app-service
      - traefik.http.routers.my-react-app.service=my-react-app-service
      - traefik.http.services.my-react-app-service.loadbalancer.server.port=80

      # # redirect http to https

      - traefik.http.middlewares.my-react-app-redirect-to-websecure.redirectscheme.scheme=https
      - traefik.http.middlewares.my-react-app-redirect-to-websecure.redirectscheme.permanent=true
      - traefik.http.routers.my-react-app.middlewares=my-react-app-redirect-to-websecure

      # # certResolver pra quando precisar gerar certificado por aqui:
      # # Exemplo do jeira. Ver traefik.yml para ver a configuracao do challenge:

      - traefik.http.routers.my-react-app-websecure.tls.certResolver=my-react-app-certresolver

    restart: always
    volumes:
      - project_data:/app/storage/
    networks:
      - proxy

volumes:
  project_data:
```

### IP

IP nada mais é do que o endereço da máquina.

Em uma metáfora: se você quer ir pra casa (website) de alguém, precisa saber qual o endereço.

### servidor

É um computador que **serve** clientes. Você pode fazer do seu laptop um servidor.

Nós pagamos por um computador da DigitalOcean, para que ele fique no ar 24/7.

### domínio

IP serve como endereço, mas é feio. Você, como usuário, não quer ir para http://104.131.141.67/, você quer ir para o https://struct.unb.br/.

Sendo assim, geralmente se tem/compra um domínio (que.eh.tipo.isso.com) com uma autoridade de domínio, que ela redirecionará o usuário para o IP. Se configura com a autoridade de domínio para qual IP o domínio deve redirecionar.

### reverse proxy

### traefik

### certificado SSL

É a diferença do http para o https (http secure). Para garantir que a conexão com o site é criptografada (e então não pode ser interceptada), se coloca um certificado no servidor. Quando o usuário acessar seu site, caso esse certificado não seja encontrado, a conexão é http e os navegadores avisam o usuário que eles não estão seguros.

Para ter um certificado no seu site, se faz uma configuração no container (quem está de fato servindo o website) ou nos reverse proxy que receberem a conexão antes dele. Sendo assim, isso pode ser feito usando o Cloudflare, o Traefik, ou dentro do container, instalando o certificado com alguma outra ferramenta.
