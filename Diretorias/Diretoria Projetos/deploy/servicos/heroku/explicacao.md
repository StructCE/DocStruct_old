---
order: 3
icon: question
label: "O que é?"
---

<!-- Ultima atualização: 22/09/2023 -->
<!-- Autor(es): Artur Padovesi e Pedro Augusto Ramalho Duarte -->

Heroku é uma **plataforma** de deployment de website, desenvolvidos em quase todos frameworks mais famosos utimamente, ela **automaticamente** analisa o repositório e verifica a **linguagem** que você está **utilizando**, e já **faz o setup completo**, facilitando uma das principais e mais demoradas atividades de um projeto de website.

O Heroku **roda** um **container docker** com seus **scripts** de inicialização. Isso significa que é **necessário** que o seu **projeto seja** um **servidor**, **ou seja**, que ele tenha uma porta para ser acessado, e que ele seja **capaz de responder a requisições**. Sendo assim, é ideal para api's, mas não para servir _assets estáticos_ (como os gerados numa aplicação React comum).

!!!
O Heroku, na ultima atualização dessa documentação, não possui plano gratuito, logo utilizamos self-hosting para o back-end na Struct.
!!!
