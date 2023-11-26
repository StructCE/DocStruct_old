---
order: 1
icon: rocket
label: "Qual a ideia de qualquer deploy?"
---

<!-- Ultima atualização: 22/09/2023 -->
<!-- Autor(es): Artur Padovesi e Pedro Augusto Ramalho Duarte -->

## Configurando o projeto para deploy

!!!
É mantido um repositório no GitBucket com os DockerFiles utilizados para construir as imagens do nosso servidor, bem como um repositório armazenado no servidor, com os arquivos do Docker Compose utilizados para rodar as imagens construídas.
!!!

1. Crie uma branch `production`;
2. Adicione `Dockerfile` ao `.gitignore` do projeto;

## Crie a imagem Docker

!!!
Muitos dos nossos Dockerfiles usam credenciais que **não devem** ser expostas. Se seu Dockerfile contém algo assim, dê commit apenas em um Dockerfile.example sem as credenciais.
!!!

1. Pegue o template de `Dockerfile` no GitBucket
2. Modifique conforme necessário, a fim de a imagem docker necessária.
3. Para construir a imagem, use o comando:

```bash Terminal
docker build -t structej/projetos:nome-do-projeto-vers.subv
```

!!!
Várias das nossas imagens são um pull do projeto, e para isso é necessário um token de acesso no github. Para gerar esse token, pode ser seguida a [Documentação do GitHub](https://docs.github.com/pt/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token#creating-a-personal-access-token-classic).
!!!

## Faça o Push da imagem Docker

1. Faça login no DockerHub, usando o comando:

```bash Terminal
docker login
```

2. Faça o push da imagem para o DockerHub, usando o comando:

```bash Terminal
docker push structej/projetos:nome-do-projeto-vers.subv
```

!!!
O `structej/projetos` é o `usuário/projeto` que enviamos a imagem de tag `nome-do-projeto-vers.subv`. É importante fazer assim, pois por padrão projetos no DockerHub são públicas, então enviamos ela para um repositório que sabemos ser privado.
!!!

## Faça o docker-compose no servidor

!!!
Os templates de `docker-compose.yml` ficam na pasta `templates` do repositório armazenado no servidor.
!!!

1. Fazer as atualizações localmente;
2. Fazer um push para o repositório;
3. Fazer um pull no servidor;
4. Faça o pull/clone do repositório do Docker Compose, usando o comando

```bash Terminal
git pull/clone
```

5. Crie um docker-compose.yml com o serviço do projeto, usando o template de `docker-compose.yml`.

## Atualize a branch `production`

1. Crie uma PR para a branch `production`, com as adições que devem ser feitas no deploy;

!!!
Procure por coisas que podem quebrar ou requerem passos adicionais no deploy, como mudanças de banco de dados, ou mudanças de configuração de serviços.
!!!

## Atualize a imagem docker

1. Faça o build da imagem docker, usando o comando:

```bash Terminal
docker build -t structej/projetos:nome-do-projeto-vers.subv
```

Por exemplo:

```bash Terminal
docker build -t structej/projetos:nome-do-projeto-1.0
```

!!!
Agora a versão da imagem docker é `versão`. Sempre incrementamos a `versão` da imagem docker, para que possamos fazer o rollback caso algo aconteça de errado.
!!!

2. Faça o push da imagem para o DockerHub, usando o comando:

```bash Terminal
docker push structej/projetos:nome-do-projeto-vers.subv
```

## Atualize o docker-compose no servidor

1. Esteja no repositório gitbucket do servidor;
2. Mude a propriedade `image` do serviço do projeto no `docker-compose.yml` para a versão atual da imagem docker;

```yml Exemplo
version: '3.7'

services:
    service-we-want-to-update:
        image: structej/projetos:nome-projeto-x.x
        ...
```

3. Faça o commit e push das alterações;
4. Entre no servidor com **ssh**;
5. Dê um `git pull` no diretório correto, com `docker-compose.yml` alterado, e depois rode `docker-compose up -d`
6. Verifique se deu certo e o serviço está funcionando normalmente;

### Deu errado, e agora?

1. Dê rollback de alguma maneira pra versão anterior. Pode usar git pra voltar pro último commit, reverter o commit anterior, alterar na mão a imagem pra versão anterior, etc.

2. Continuou dando errado? Talvez alguém tenha alterado esse arquivo anteriormente e não deu `docker-compose up -d`, e agora o problema sobrou pra você. Verifique se as variáveis de ambiente estão configuradas corretamente.

3. Tá difícil? Verifique os commits anterirores e as alterações feitas nesse `docker-compose.yml` e tente isolar a mudança que causou o problema.
