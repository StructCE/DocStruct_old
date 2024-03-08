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

```tsx page.tsx
export default function page({ params }: { params: { slug: string } }) {
	return <h1> Olá, {params.slug}! </h1>;
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

```tsx page.tsx
export default function Page() {
	return <h1>Página de exemplo.</h1>;
}
```

!!!
As páginas não são exclusivamente `Server Components`, elas podem ser `Client Components` ou podem ser híbridas. Para transformar um componente em `Client Component` é necessário adicionar `"use client"` no início do arquivo. Os arquivos híbridos são arquivos server components que englobam client components.

==- Client Component

```tsx page.tsx
"use client";
import { useRouter } from "next/navigation";

export default function page() {
	const router = useRouter();

	return (
		<button type="button" onClick={() => router.push("/")}>
			Home
		</button>
	);
}
```

===
!!!

Os layouts são arquivos com o nome `layout.tsx` que apresentam estrutura do React e vão agregar nas UIs de múltiplas rotas. Elas não criam rotas públicas e acumulam seguindo a hierarquia de rotas. Componentes de layout devem receber o `react prop` `children` que será a página do arquivo `page.tsx` da rota.

```tsx layout.tsx
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

```tsx page.tsx
import { Metadata } from "next";

export const metadata: Metadata = {
	title: "Hack do dinheiro infinito",
};

export default function Page() {
	return "...";
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
Uma rota é publicamente acessível, somente se existir um arquivo do tipo `page.tsx` ou `route.ts`.
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

```tsx page.tsx
import Link from "next/link";

export default function page() {
	return (
		<div>
			<Link href="/profile">Perfil</Link>
			<Link href="/">Página inicial</Link>
		</div>
	);
}
```

===

Dependendo do contexto, nem sempre é possível utilizar um componente para realizar essa navegação. Desse modo,
a utilização do [hook](https://nextjs.org/docs/app/api-reference/functions/use-router) `useRouter` para `client components` ou da [função](https://nextjs.org/docs/app/api-reference/functions/redirect) `redirect` para `server components` pode solucionar esse problema.

==- Exemplo de useRouter

```tsx page.tsx
"use client";
import { useRouter } from "next/navigation";

export default function page() {
	const router = useRouter();

	return (
		<button type="button" onClick={() => router.push("/")}>
			Home
		</button>
	);
}
```

===

==- Exemplo de redirect

```tsx page.tsx
import { redirect } from "next/navigation";

export default function page() {
	return (
		<button type="button" onClick={() => redirect("/")}>
			Home
		</button>
	);
}
```

===

!!!
Para mais informações sobre [redirecionamento](https://nextjs.org/docs/app/building-your-application/routing/redirecting).
!!!

## API REST

!!!
Se estiver fazendo um projeto com a stack T3, utilize o tRPC para fazer a API.
!!!

As `APIs` são os meios que diferentes partes do site utilizarão para conversar entre si. No app router do next, é possivel a utilização do método `route handler` para a criação das `APIs`. As `route handlers` podem ser utilizadas para ler dados existentes `GET`, criar novos dados `POST`, atualizar dados existentes `PATCH` e deletar algum dado `DELETE`. Para criar uma API própria é interessante ter um banco de dados funcional, logo, é recomendado aprender a parte da `ORM` `Prisma` antes.

### Criando uma rota de API

Para criar uma rota API é necessário escrever um arquivo com o nome `route.ts`, nele será construída a API da rota.

Para criar os métodos recebidos pela API, iremos exportar funções assíncronas `export async function` com o nome `GET`, `POST`, `PATCH` e `DELETE`, dependendo do uso da função. Se sua função depender de algum parâmetro passado no `body` da requisição, nos parâmetros da função coloque um parâmetro request com a tipagem `Request`.

O corpo da função tem como objetivo a aquisição dos dados a serem transmitidos pela API ou a realização de mudanças no banco de dados, provavelmente vai entrar alguma lógica envolvendo o banco de dados nessa parte.

Por fim, as APIs `route handler` do next precisam do `return` de uma `Response`, nessa `Response` utilize o método `json` para a transmissão dos dados coletados ou código de status.

==- Exemplo de Route Handler

```ts route.ts
import prisma from "@/../prisma/index";

export async function GET() {
	let mensagens = await prisma.mensagem.findMany();
	if (mensagens) {
		let mensagem = mensagens[Math.floor(Math.random() * mensagens.length)];
		return Response.json({ mensagem });
	} else {
		return Response.json({ mensagem: "Nenhuma mensagem disponível." });
	}
}

export async function POST(req: Request) {
	const { texto } = await req.json();
	await prisma.mensagem.create({ data: { texto: texto } });
	return Response.json({ status: 200 });
}
```

```schema.prisma

// CÓDIGO ACIMA

model Mensagem {
  id    Int     @id @default(autoincrement())
  texto String?
}

// Model utilizado no exemplo anterior

```

!!!
A primeira linha deste arquivo está importando o cliente do `prisma` que é um singleton. Veja o arquivo do singleton na aba `como 'instalar' o prisma?` na documentação. O `path` pode ser diferente dependendo da organização do projeto.
!!!
===

!!!
Em uma rota não pode conter uma `page` e um `route handler` ao mesmo tempo, pois haverá conflito e o `route handler` prevalecerá. Por isso, recomendo criar um diretório `api` na rota para colocar o `route handler`.
!!!

## Interações com banco de dados

Podemos utilizar diferentes métodos para a busca de dados para formar a página do site, sendo eles `server-side` ou `client-side`, normalmente esses dados serão recebidos em formato `JSON`. Os dados podem vir de APIs existentes na internet ou de APIs internas alimentadas com o próprio banco de dados para `client components` e os dados podem vir de APIs externas ou diretamente do banco de dados para os `server components`.

!!!
Quando estiver fazendo essa busca utilize `async` e `await`, pois o programa tem que esperar a resolução da `Promise`.
!!!

### Server Side

Quando precisamos buscar dados para criar a página no lado do servidor pode-se utilizar `fetch` ou `axios` para buscar por dados externos e uma função com busca direta no banco de dados para dados internos.

!!!
O next extende o `fetch` tornando o processo de `caching` automático, por outro lado, quando utilizar o axios ou os dados internos é necessário utilizar a função `cache` do `react`.
!!!

!!!
Os desenvolvedores do next não recomedam a utilização de `route handlers` em `server side`.
!!!

==- Exemplo com fetch

```tsx page.tsx
async function getData() {
	const res = await fetch("https://pokeapi.co/api/v2/pokemon/ditto");

	if (!res.ok) {
		throw new Error("Ocorreu um erro.");
	}
	return res.json();
}

export default async function Page() {
	const data = await getData().catch((err) => {
		alert(err);
	});

	return <div>{data?.abilities[0].ability.name}</div>;
}
```

===

==- Exemplo com dados internos
!!!
Model sendo utilizada:

```schema.prisma
model Mensagem {
  id    Int     @id @default(autoincrement())
  texto String?
}
```

!!!

```ts utils/mensagem.ts
import prisma from "../prisma/index";
import { cache } from "react";

type mensagem =
	| void
	| {
			id: number;
			texto: string | null;
	  }[];

export const getMensagens = cache(async () => {
	let mensagens = await prisma.mensagem.findMany().catch(console.log);

	return mensagens;
});
```

!!!
Cache não é necessário, mas ajuda na performance do código.
!!!

```tsx page.tsx
import { getMensagens } from "@/../utils/mensagem";

export default async function Page() {
	const mensagens = await getMensagens();
	return (
		<div>
			<h2>Mensagens:</h2>
			{mensagens?.map((msg) => {
				return <p>{msg.texto}</p>;
			})}
		</div>
	);
}
```

===

### Client Side

Quando precisar de dados no lado do cliente, poderá ser utilizados `fetch` e `axios` para buscar dados no servidor via `route handlers`.

!!!
Dados externos podem sofrer `caching` no servidor, então, pode ser uma boa ideia buscá-los no lado do servidor.
!!!

Criar funções para fazerem as buscas no `client-side` pode ser interessante, pois evita a repetição de código e deixa a leitura mais simples.

==- Exemplo de função

```ts mensagem.ts
export type Mensagem = {
	id: number;
	texto: string;
};

export async function getMensagemFetch(): Promise<{
	mensagem: Mensagem;
} | null> {
	const res = await fetch("http://localhost:3000/api");
	if (!res.ok) {
		return null;
	}
	return res.json();
}

export async function postMensagemFetch(texto: string) {
	const res = await fetch("http://localhost:3000/api", {
		method: "POST",
		body: JSON.stringify(texto),
	});
}
```

===
Os dados coletados podem ser utilizados do jeito que for necessário, exemplo de uso abaixo:

==- Exemplo de client side

```tsx page.tsx
"use client"

import { getMensagem, postMensagem } from "../../clientApi/mensagem.ts"
import { useState } from "react";

export default function Page() {
  const [mensagem, setMensagem] = useState("");
  const [novaMensagem, setNovaMensagem] = useState("");

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
		setNovaMensagem(e.target.value);
	}

	function handleSubmit(e: FormEvent<HTMLFormElement>) {
		e.preventDefault();
		postMensagem(novaMensagem);
	}

  return (
  // Uso da api junto com useState
		<div>
			<p>Mensagem client-side: {mensagem}</p>
			<button
				type="button"
				onClick={async () => {
					await getMensagem().then((data) => {
						if (data) setMensagem(data.mensagem.texto);
					});
				}
			>
				Busque uma mensagem.
			</button>)

// Form para uso do POST
      <form onSubmit={handleSubmit}>
				<input
					onChange={handleChange}
					placeholder="Nova mensagem"
					name="mensagem"
				/>
				<button
					type="submit"
				>
					Criar mensagem
				</button>
			</form>
    </div>
}
```

===
