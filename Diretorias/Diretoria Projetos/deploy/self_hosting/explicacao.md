---
order: 3
icon: rocket
label: "Explicando nosso self hosting"
---

<!-- Ultima atualização: 22/09/2023 -->
<!-- Autor(es): Artur Padovesi e Pedro Augusto Ramalho Duarte -->

## O que é self hosting?

Self-hosting, também conhecido como auto-hospedagem, é um conceito em tecnologia que se refere à prática de hospedar e gerenciar seus próprios serviços, aplicativos ou plataformas em infraestrutura própria, em vez de utilizar serviços e plataformas de terceiros ou fornecedores externos.

Em outras palavras, quando você opta por self-hosting, você é responsável por configurar, administrar e manter todos os aspectos do serviço ou aplicativo que está executando.

## Nossa estrutura

No momento, pagamos por um servidor no **DigitalOcean**, que é um serviço de cloud computing, ou seja, uma plataforma que usa a conectividade da internet para hospedar e prover recursos, programas e informações em nuvem, e usamos ssh para acessá-lo e fazê-lo rodar exatamente o que queremos.

Além disso, usamos Docker para poder isolar e rodar vários servidores na mesma máquina. Para levar o tráfego que chega em nossa máquina para o container correto, usamos o [Traefik v2](https://doc.traefik.io/traefik/) como [proxy reverso](https://pt.wikipedia.org/wiki/Proxy_reverso). Isso é necessáro pois temos vários domínios apontando para o nosso servidor (structej.unb.br, marks.com.br).
