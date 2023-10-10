---
icon: pencil
label: "Páginas customizadas"
order: 2
---

# Custom App

Por padrão, o Next usa um componente próprio `App` para iniciar páginas. Porém, há a opção de criar seu próprio componente, criando um arquivo chamado `pages/_app.tsx`. Este arquivo possui prioridade em relação ao `App` padrão e, então, será o componente usado.

Um exemplo básico de como definir o seu próprio `App`:

```ts pages/_app.tsx
import type { AppProps } from 'next/app'
 
export default function MyApp({ Component, pageProps }: AppProps) {
  return <Component {...pageProps} />
}
```

Neste caso, `Component` será a página atual e `pageProps` será um objeto com parâmetros carregados por métodos de busca de dados como `getInitialProps`, caso nenhum seja usado, será um objeto vazio. 

Como usamos TypeScript, que é uma linguagem bastante tipada, e o componente `MyApp`será por onde todos layouts/componentes irão receber seus respectivos parâmetros (`Component` e `pageProps`), temos que declarar o tipo dos parâmetros passados que será o `AppProps` do próprio Next. Assim, nossa aplicação poderá apresentar erros de tipo ao longo de seu desenvolvimento.

# Custom Document

Um `Document` customizado serve para atualizar o `<html>` e o `<body>` padrões do Next. Assim como no Custom App, deve-se criar um arquivo `_document.tsx` para sobrepor o componente padrão.
Neste arquivo, pode-se definir algumas tags como :

```ts pages/_document.tsx
import { Html, Head, Main, NextScript } from 'next/document'
 
export default function Document() {
  return (
    <Html lang="en">
      <Head />
      <body>
        <Main />
        <NextScript />
      </body>
    </Html>
  )
}
```

Observações: 
- O componente `<Head />` aqui é comum a todas as páginas, caso contrário use o `next/head`.
- Neste arquivo não será inicializado nenhum componente react ou alguma lógica de aplicação fora de `<Main />`.

# Custom Error

Para criar uma página personalizada para um dado erro, basta criar um arquivo com o código de erro. Por exemplo, para o erro 404, basta criar um arquivo `pages/404.tsx` e, então, construir seu componente react que irá ser inicializado na ocorrência de tal erro.

Ou, pode-se fazer uma página genérica de erros de nome `pages/_error.tsx` e usar o `getInitialProps` para receber os parâmetros (resposta ou código de erro enviados pelo servidor). Assim, o componente irá ser inicializado na ocorrência de qualquer erro. Exemplo:

```ts pages/_error.tsx
function Error({ statusCode }) {
  return (
    <p>
      {statusCode? `Um erro ${statusCode} ocorreu com o servidor`
        : 'Um erro ocorreu com o cliente'}
    </p>
  )
}
 
Error.getInitialProps = ({ res, err }) => {
  const statusCode = res ? res.statusCode : err ? err.statusCode : 404
  return { statusCode }
}
 
export default Error
```

Do `getInitialProps`, dentro da própria função, pegamos os parâmetros `res`e `err` que são passados pela API ao componente e verificamos se há um `statusCode` na resposta recebida, caso não haja verificamos se é passado um erro e, se também não for passado, o `statusCode`é definido como 404, de `Not Found`.

No código html em si, é retornado o `statusCode` do erro que ocorreu com o servidor, se não houver, é alertado um erro no lado cliente.