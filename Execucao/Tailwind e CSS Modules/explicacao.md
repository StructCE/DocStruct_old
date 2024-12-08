---
order: 3
icon: question
label: "O que são TailwindCSS e CSS Modules?"
author:
    name: Eduardo P.P. Ferreira
date: 2024-02-28
category: Explicação
tags:
    - styles
    - explicacao
---

## Estilos em NextJS

NextJS permite utilizar várias formas de estilizar sua aplicação, como *inline CSS*, *Global CSS*, *CSS Modules*, *Tailwind CSS*, *Sass* e *CSS-in-JS*. Todos são explicados na [documentação oficial](https://nextjs.org/docs/app/building-your-application/styling), mas aqui vamos focar em **Tailwind** e **CSS Modules**.

## TailwindCSS

[TailwindCSS](https://tailwindcss.com/) é um framework CSS "*utility-first*", o que significa que ele disponibiliza várias classes utilitárias de propósito único para customizar elementos.

> "Tailwind CSS works by scanning all of your HTML files, JavaScript components, and any other templates for class names, generating the corresponding styles and then writing them to a static CSS file."
> \- [TailwindCSS](https://tailwindcss.com/docs/installation)

O propósito de Tailwind é baseado na ideia de *Atomic CSS*, que, de maneira simples, propõe a criação de classes com um propósito único bem definido. Por exemplo, ao invés de escrever a estilização de uma cor para um background de um elemento diretamente nele, criamos uma classe e a chamamos nele:

```CSS
.bg-blue {
  background-color: rgb(81, 191, 255);
}
```

```HTML
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Document</title>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <!-- Classe bg-blue que criamos antes -->
    <div><h1 class="bg-blue">Hello world!</h1></div>
  </body>
</html>
```

Tailwind tem essa idea como foco de seu funcionamento. De acordo com [sua própria documentação](https://tailwindcss.com/docs/utility-first),

> "With Tailwind, you style elements by applying pre-existing classes directly in your HTML.
> ...
>
> This approach allows us to implement a completely custom component design without writing a single line of custom CSS."

## CSS Modules

*CSS Modules* são estilos escritos em CSS que atuam em escopo local. São uma forma de importar arquivos CSS em um componente React.
Next possui suporte nativo a isso por meio de arquivos nomeados com o formato `[nome].module.css`.

A vantagem de usar CSS Modules é que fica mais simples evitar conflitos quando utilizamos o mesmo nome de classes em diferentes componentes da aplicação, já que o escopo de um módulo é restrito ao componente no qual foi chamado.

!!!info Como funciona isso?
Caso esteja se perguntando como CSS Modules resolve conflitos de nomenclatura, dê uma olhada [nessa documentação](https://github.com/css-modules/css-modules). Basicamente, os módulos são nomeados automaticamente utilizando um *hash* de acordo com o formato `[filename]_[classname]_[hash]`. Então, uma classe definida sem CSS Modules chamada `description` seria nomeada como `description`, enquanto uma classe de mesmo nome definida em um CSS Module nomeado `styles.module.css` seria chamada internamente de `styles_description_<hash>`.
!!!
