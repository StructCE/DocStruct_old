---
order: 4
icon: apps
label: "Guia App routing"
author:
    name: "Pedro Amorim de Gregori"
category: Explicação
date: 2024-02-20
---

# Guia de desenvolvimento Next utilizando App Router

## Introdução

O `App Router` foi introduzido na versão 13 do next.js e foi construído utilizando o `React Server Components`.

O `App Router` trabalha com o diretório app para a criação de rotas que podem ou não serem de acesso público, por padrão componentes deste diretório são `RSCs`. As páginas são criadas nas rotas e irão herdar certas características definidas por arquivos especiais como `navbar` e `footer` definidos no arquivo `layout.tsx` da rota.

!!!
Para mais informações sobre [React Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components).
!!!

!!!
O App Router não trabalha exclusivamente com `server components`, se for necessário o uso de `client components` adicione a linha `"use client"` no topo do arquivo da página.
!!!

## Roteamento

### Definindo rotas

O next utiliza um sistema de roteamento baseado no sistema de arquivos. Desse modo, os diretórios são utilizados para criar as rotas que levarão a outros diretórios ou arquivos filhos e os arquivos são utilizados para definir a página da rota correspondente.
{.list-icon}

==- Exemplo de rota

-   :icon-file-directory: pai
    -   :icon-file-directory: exemplo
        -   :icon-file-code: page.tsx

URL será: `site.com/pai/exemplo`

===

#### Rotas dinâmicas

As rotas dinâmicas são utilizadas quando é necessário a utilização de segmentos de dados que variam, mas são importantes para a implementação da página. Elas podem ser "declaradas" utilizando os `[]` e utilizadas no código ao serem passadas como parâmetros da função da página, layout, etc.

==- Exemplo de rotas dinâmicas

-   :icon-file-directory: [slug]
    -   :icon-file-directory: exemplo
        -   :icon-file-code: page.tsx

```page.tsx
export default function page({ params }: { params: { slug: string}}) {
  return <h1> Olá, {params.slug}! </h1>
}

```

URL poderá ser: `site.com/pedro/exemplo` resultando em uma página com um \<h1> `Olá, pedro!`.

===

#### Grupo de rotas

É possível criar um diretório para agrupar certas rotas sem que o nome agregue na URL, só é necessário utilizar o `()` no nome do diretório.

==- Exemplo de grupo de rotas

-   :icon-file-directory: (pai)
    -   :icon-file-directory: exemplo
        -   :icon-file-code: page.tsx

URL será: `site.com/exemplo`

===

!!! Diretórios privados

Diretórios podem ser criados para organizar arquivos sem serem considerados rotas. Para isso, o nome do diretório deve começar com uma `_`.
!!!

### Convenção de arquivos

Os arquivos apresentam uma certa nomenclatura a ser usada para a criação da UI. Sendo as principais:

-   [Page](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#pages): Esse arquivo define a UI da rota e a deixa o acesso público.
-   [Layout](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#layouts): Define uma estrutura padrão para as páginas pertencentes a rota. Por exemplo, configurações de metadados e componentes que são utilizados em múltiplas páginas dessa rota.
-   [Loading](https://nextjs.org/docs/app/building-your-application/routing/loading-ui-and-streaming#instant-loading-states): Define a UI da rota, enquanto a página principal não estiver completa. Dependendo do design pode ser utilizada uma página esqueleto, uma barra de loading ou o que vier de sua imaginação, desde que a página de loading não fique muito pesada, desse modo, causando problemas de performance.

!!!
Esses arquivos deverão ser da extensão `.js`, `.jsx` ou .tsx. Dê preferência ao uso da extensão `.tsx `(TypeScript).

Para mais informações procure na [documentação do next](https://nextjs.org/docs/app/building-your-application/routing#file-conventions).
!!!

### Criando páginas e layouts

As páginas são arquivos com o nome `page.tsx` que apresentam a estrutura do React e vão desempenhar o papel de criar UIs especificas a determinada rota. Elas tornam a rota acessível para visitantes e por padrão são ´Server Components´.

```page.tsx
export default function Page() {
  return <h1>Página de exemplo.</h1>
}
```

!!!
As páginas não são exclusivamente `Server Components`, elas podem ser `Client Components` ou podem ser híbridas. Para transformar um componente em `Client Component` é necessário adicionar `"use client"` no início do arquivo. Os arquivos híbridos são arquivos server components que englobam client components.

==- Client Component

```page.tsx
"use client"
import { useRouter } from 'next/navigation'

export default function page() {
  const router = useRouter()

  return (
    <button
      type='button'
      onClick= {() => router.push("/")}>
      Home
    </button>)
}
```

===
!!!

Os layouts são arquivos com o nome `layout.tsx` que apresentam estrutura do React e vão agregar nas UIs de múltiplas rotas. Elas não criam rotas públicas e acumulam seguindo a hierarquia de rotas. Componentes de layout devem receber o `react prop` `children` que será a página do arquivo `page.tsx` da rota.

```layout.tsx
export default function Layout({
  children
}: {
  children: React.ReactNode
}) {
  return {
    <div>Eu sou uma navbar diferenciada, confia!</div>

    { children }

    <div>E eu sou um footer bem interessante.</div>
    }
}
```

!!! Metadados
Os metadados de uma página devem ser alterados utilizando a [APIs de metadados do next.](https://nextjs.org/docs/app/building-your-application/optimizing/metadata) Por questões de performance, não utilize `<head>`.

```page.tsx
import { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Hack do dinheiro infinito',
}

export default function Page() {
  return '...'
}

```

!!!

### Hierarquia dos componentes

O next renderiza os arquivos especiais, vistos anteriormente, seguindo uma ordem especifica. Sendo essa, para os arquivos principais:

=== Ordem de hierarquia
Layout -> Loading -> Page
===

Se a rota for aninhada, a hierarquia ocorrerá juntando componentes da rota pai com as do filho, como o exemplo a seguir:

=== Ordem de hierarquia com aninhamento

-   :icon-file-directory: pai
    -   :icon-file-directory: exemplo
        -   :icon-file-code: page.tsx

A hierarquia será:
Layout (Pai) -> Loading (Pai) -> Layout (Exemplo) -> Loading (Exemplo)-> Page
===

!!!
Uma rota é publicamente acessível, somente se existir um arquivo do tipo `page.tsx` ou `route.tsx`.
!!!

!!!
Para padrões avançados de roteamento, como Rotas em paralelo ou interceptação de rotas, [veja](https://nextjs.org/docs/app/building-your-application/routing#advanced-routing-patterns)
!!!

### Navegação entre rotas

Por motivos de performance, ao utilizar o next sempre utilize o componente built-in do next `<Link>` ao invés do `<a>` quando quiser criar navegação entre rotas no "HTML".

!!!
Para utilizar o `<Link>` é necessário o preenchimento da propriedade `href`.
!!!

==- Exemplo de Link

```page.tsx
import Link from 'next/link'

export default function page() {
  return (
    <div>
      <Link href="/profile">Perfil</Link>
      <Link href="/">Página inicial</Link>
    </div>
  )
}

```

===

Dependendo do contexto, nem sempre é possível utilizar um componente para realizar essa navegação. Desse modo,
a utilização do [hook](https://nextjs.org/docs/app/api-reference/functions/use-router) `useRouter` para `client components` ou da [função](https://nextjs.org/docs/app/api-reference/functions/redirect) `redirect` para `server components` pode solucionar esse problema.

==- Exemplo de useRouter

```page.tsx
"use client"
import { useRouter } from 'next/navigation'

export default function page() {
  const router = useRouter()

  return (
    <button
      type='button'
      onClick= {() => router.push("/")}>
      Home
    </button>
  )
}

```

===

==- Exemplo de redirect

```page.tsx
import { redirect } from 'next/navigation'

export default function page() {
  return (
    <button
      type='button'
      onClick={() => redirect("/")}>
      Home
    </button>
  )
}

```

!!!
Para mais informações sobre [redirecionamento](https://nextjs.org/docs/app/building-your-application/routing/redirecting).
!!!
