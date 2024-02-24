---
order: 1
icon: file-code
label: "Utilizando NextAuth em projetos"
author:
  name: João Santos
  # avatar: ../../Imagens DocStruct/Logos/logo_struct.png
author:
  name: Willyan Marques
date: 2023-10-27
category: Implementação
tags:
  - NextAuth
  - instalação
  - Next.js
  - autenticação
  - usuário
---

!!!warning **Atenção!! A documentação a seguir assume que você está utilizando o _App Router_ do Next.js**

!!!

## Instalação

Para adicionar o `NextAuth` a um projeto `Next.js` basta acessar a raiz do repositório pelo terminal e executar um dos comandos abaixo de acordo com o gerenciador de pacotes sendo utilizado:

+++ NPM

```bash
npm install next-auth
```

+++ PNPM

```bash
pnpm add next-auth
```

+++ YARN

```bash
yarn add next-auth
```

+++

## Session

### Configuração

!!!warning **Atenção:**

Caso você tenha criado o projeto utilizando a **stack do T3** _não é necessário_ configurar o nextauth manualmente.

Caso contário, verifique a versão do Next.js utilizada no projeto e se o tipo de roteamento usado é o `Pages Router` ou o `App Router`, pois o modo de implementação é diferente.
!!!

#### Pages Router (anterior ao nextjs 13.2)

Para adicionar o NextAuth.js a um projeto, crie um arquivo chamado `[...nextauth].js` na pasta `pages/api/auth/` e preencha-o com o código a baixo. Este arquivo contém o manipulador de rotas dinâmicas para o NextAuth.js, que também conterá todas as configurações globais do NextAuth.js e todas as solicitações para `/api/auth/*` (`signIn`, `callback`, `signOut`, etc.) serão automaticamente tratadas pelo NextAuth.js.

```js
import NextAuth from "next-auth";
import GoogleProvider from "next-auth/providers/google";

export const authOptions = {
  // Configure um ou mais provedores de autenticação
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_ID,
      clientSecret: process.env.GOOGLE_SECRET,
    }),
    // ...adicione mais provedores aqui
  ],
};

export default NextAuth(authOptions);
```

#### App Router (posterior ao nextjs 13.2)

Nesse caso, crie um arquivo chamado `route.js` na pasta `app/api/auth/[...nextauth]/` e preencha-o com o código a baixo.

```js
import NextAuth from "next-auth";

const handler = NextAuth({
  // ...adicione mais provedores aqui
});

export { handler as GET, handler as POST };
```

#### Exemplo de route.js com GoogleProvider:

```js
import NextAuth from "next-auth/next";
import GoogleProvider from "next-auth/providers/google";

const handler = NextAuth({
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    }),
  ],
});

export { handler as GET, handler as POST };
```

### Fetch e PreFetch

#### useSession()

!!!
Acesso:

- Client-Side: **YES**

- Server-Side: **NO**

!!!

O `useSession` é um importante hook do React que é utilizado nas aplicações Next Auth para recuperar informações da sessão de usuário. Para utilizá-lo primeiro é preciso expor o conteudo da sessão de usuário por meio do `<SessionProvider>`.

Você pode incluir todas rotas do seu projeto dentro de um contexto de `<SessionProvider>` global, mas também é possível definir contextos específicos apenas para aquelas rotas que precisam ter acesso aos dados da sessão de usuário.

Para o primeiro caso, podemos criar um arquivo `layout.tsx` na raiz do projeto (app) para englobar todas rotas em um contexto adicionando as tags personalizadas `<AuthProvider>` por fora do elemento `{children}` como no exemplo:

```js src/app/layout.tsx
import { AuthProvider } from "~/components/authProvider";

export default function RootLayout({
  children,
}: {
  children: React.ReactNode,
}) {
  return (
    <>
      <AuthProvider>{children}</AuthProvider>
    </>
  );
}
```

!!!
No exemplo acima, criamos um componente AuthProvider para empacotar as páginas (children) em um `SessionProvider`, permitindo o acesso aos dados de sessão pelo `client-side` a partir da função `getSession()`. Isso poderia ter sido feito diretamente no arquivo layout, mas assim conseguimos aproveitar o componente em outros layouts e modularizar melhor quais partes da página precisam ser `client-side` ou `server-side`.

```js src/components/auth/authProvider.tsx
"use client";
import { SessionProvider } from "next-auth/react";

const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  return <SessionProvider>{children}</SessionProvider>;
};

export default AuthProvider;
```

!!!

Dessa forma, o `useSession` terá acesso aos dados e status da sessão.

!!!warning AVISO!!

A desestruturação dos `pageProps` em session e no resto das pageProps significa uma alteração no comportamento normal de `getServerSideProps` e `getStaticProps`.

Caso pegue a session no servidor e tente passar para a página como suas pageProps, a session será interceptada aqui no App e nunca chegará no seu componente da página. Ao invés disso será usada para alimentar o hook `useSession`, portanto use-o.
Exemplo:

```js
import { getServerSession } from "next-auth/next";
import { authOptions } from "~/server/auth";
import { permanentRedirect } from "next/navigation";

export async function getServerSideProps(ctx) {
  const session = await getServerSession(ctx.req, ctx.res, authOptions);

  if (!session || !session.user) {
    permanentRedirect("/login");
  }

  return {
    props: {
      session,
    },
  };
}

// Dá erro, e a tipagem é mentirosa:
export default function HomePage({ session }: InferGetServerSidePropsType<typeof getServerSideProps>) {
    return <pre>{JSON.stringify(session.user)}</pre>
}

// Dá certo:
export default function HomePage() {
  const { data: session } = useSession();
  return <pre>{JSON.stringify(session.user)}</pre>;
}
```

!!!

##### Exemplo

O Hook do React `useSession` pode ser usado no NextAuth.js como uma maneira simples de verificar se alguém está autenticado. Como no exemplo de botão de SignIn:

```js src/components/auth/signInButton.tsx
"use client";
import React from "react";
import { signIn, signOut, useSession } from "next-auth/react";

const SigninButton = () => {
  const { data: session } = useSession(); // hook que pega os dados da sessão

  //se não existir a sessão ou o usuário, mostra o botão de login
  if (!session || !session.user) {
    return <button onClick={() => signIn()}>Sign In</button>;
  } else {
    //caso contrário, mostra os dados e a opção de SignOut
    return (
      <div>
        <p>{session.user.name}</p>
        <button onClick={() => signOut()}>Sign Out</button>
      </div>
    );
  }
};

export default SigninButton;
```

A ideia é armazenar os resultados do `useSession` em props para aferir se o usuário ja foi autenticado. Com isso você pode criar componentes que se comportam de maneiras diferentes caso o usuário esteja logado ou não,como no botão de login acima ou por exemplo em uma foto de perfil de uma navbar.

#### getSession()

!!!
Acesso:

- Client-Side: **YES**

- Server-Side: **NO**

!!!

O `getSession` assim como o `useSession` é uma função utilizada para obter os dados de sessão de um usuário no `client-side` do site. A difença entre ese é que o `useSession` é um hook do react, o que significa que ele está sujeito as regras dos hooks, enquanto o `getSession` é imune a elas.

!!!warning
Recomenda-se usar essa função apenas do `client-side`, pois no `server-side` ocorrerão `fetches` extras que podem impactar a performance e funcionamento da página.
!!!

```js exemplo
export async function HomePage() {
  const session = await getSession();
  /* ... */
}
```

#### getServerSession()

!!!
Acesso:

- Client-Side: **NO**

- Server-Side: **YES**

!!!

Como o nome já diz, o `getServerSession` é utilizado para obter a sessão no `server-side`, fazendo os fetches e renderizações no servidor antes de serem passados para o usuário. Geralmente ele é utilizando em contextos de `Route Handlers`, `React Server Components`, `rotas API` ou no `getServerSideProps`

##### Exemplo 1 (App Router)

Para pegar os dados no server side dentro das páginas no `App Router`, você precisa importar o `authOptions` e passar dentro da função.

```js
import { getServerSession } from "next-auth/next";
import { authOptions } from "~/server/auth";

export default async function Page() {
  const session = await getServerSession(authOptions);
  ...
}
```

##### Exemplo 2 (getServerSideProps)

```js
import { getServerSession } from "next-auth/next";
import { authOptions } from "~/server/auth";
import { permanentRedirect } from "next/navigation";

export async function getServerSideProps(ctx) {
  const session = await getServerSession(ctx.req, ctx.res, authOptions);

  if (!session || !session.user) {
    permanentRedirect("/login");
  }

  return {
    props: {
      session,
    },
  };
}
```

#### getServerAuthSession() [_T3 stack_]

!!!
Acesso:

- Client-Side: **NO**

- Server-Side: **YES**

!!!

A stack do T3 oferece a função auxiliar `getServerAuthSession` para fazer o pre-fetch da sessão no server-side. Essa função é parecida com a `getServerSession`, mas com a vantagem de que você não precisa importar o `authOptions` toda vez que você precisar realizar um pre-fetch da sessão. Assim como no caso da função do next-auth, os dados da sessão podem ser passados para o client-side por peio do `getServerSideProps`:

```js
import { permanentRedirect } from "next/navigation";

export async function getServerSideProps(ctx) {
  const session = await getServerAuthSession(ctx);

  if (!session || !session.user) {
    permanentRedirect("/login");
  }
  return {
    props: {
      session,
    },
  };
}

export default function HomePage() {
  const { data: session } = useSession(); // session já foi pré-processado
  // NOTE: `session` não possui o status `loading` porque ele é processado no server

  return <pre>{JSON.stringify(session.user)}</pre>;
}
```

### Personalização

A stack do T3 traz uma conexão pronta entre a sessão e a model `User` definida no Prisma, de forma que podemos passar parâmetros extras e acessá-los utilizando o comando `getSession()` no `client side` do nosso site.

Dessa forma, por exemplo, podemos criar um parâmetro para o `User` no schema que indique se o usuário é um administrador e passar essa informação nos dados de sessão para as páginas, o que possibilitará criar **renderizações condicionais** de componentes ou **proteger determinadas rotas** que precisam ser restritas.

```go
model User {
    id            String    @id @default(cuid())
    name          String?
    email         String?   @unique
    emailVerified DateTime?
    image         String?
    accounts      Account[]
    sessions      Session[]

    isAdmin       Boolean //novo parâmetro
}

```

Para fazer com que o session inclua esse novo parâmetro, basta ir no arquivo de configurações do nextath em `src/server/auth.ts` e adicionar esse parâmetro ao `session callback` do `authOptions`:

```js
export const authOptions: NextAuthOptions = {
  callbacks: {
    session: ({ session, user }) => ({
      ...session, // parâmetros default da session
      user: {     // usuário da session
        ...session.user,
        isAdmin: user.isAdmin // novo parâmetro para o user da session
      },
    }),
  },
```

Quando você adicionar esse novo parâmetro ele já será passado dentro da sessão, mas o typescript apontará um erro de tipagem não segura. Para corrigir será necessário declarar uma nova interface para o user object da sessão no next-auth:

```js
declare module "next-auth" {
  interface Session extends DefaultSession {
    user: {
      isAdmin: boolean; // novo parâmetro (session.user.isAdmin)
    } & DefaultSession["user"];
  }

  // ...novas propriedades
  interface User {
    isAdmin: boolean ; // novo parâmetro
  }
}

```

!!!warning Somente atualizar a interface mais externa não atualiza as demais interfaces que a utilizam. No caso do exemplo acima, será necessário atualizar o `user` dentro de `Session` por conta da tipagem.
!!!

<!-- ## Autenticação

### SignIn

### SignOut -->

## Provedores

O Next Auth possibilita que o critério de autenticação (provedores) seja por meio de credenciais customizáveis (nome, senha, email, etc) ou por provedores externos (Google, GitHub, Discord, etc).

### OAuth (Externo)

OAuth é o processo de autenticação do NextAuth que utiliza de provedores de login externos preexistentes que dão ao usúario a opção de realizar a autenticação por meio de outra plataforma como Google, Github, Discord, etc.

Por serem processos externos de várias fontes diferentes, cada autenticação escolhida terá uma documentação específica diferente. Para acessar todos os provedores de autenticação externa suportados pelo NextAuth e suas respectivas documentações clique [Aqui](https://github.com/nextauthjs/next-auth/tree/main/packages/next-auth/src/providers).

Segue um exemplo demonstrativo do Google como provedor OAuth:

```js
import GoogleProvider from "next-auth/providers/google";

GoogleProvider({
  clientId: process.env.GOOGLE_CLIENT_ID,
  clientSecret: process.env.GOOGLE_CLIENT_SECRET,
  profile(profile) {
    return {
      // Retorne todas as informações de perfil de que você precisa.
      // O único campo obrigatório é o 'id'
      // para ser capaz de identificar a conta quando adicionada a um banco de dados.
    };
  },
});
```

#### Como conseguir as credenciais para a autenticação externa Next auth (Google)

Utilizarei o exemplo do Google como autenticador externo, pela grande gama de [documentação](https://developers.google.com/identity/protocols/oauth2?hl=pt-br) disponível.

Para configurar a autenticação com o Next Auth usando o Google como provedor, você precisará obter as credenciais apropriadas do Google. Siga estas etapas:

1. Acesse o [Console de Desenvolvedores do Google](https://console.developers.google.com/apis/credentials).

2. Com o projeto criado e/ou adicionado na plataforma, na barra de opções lateral clique no item `Credenciais` e em seguida, no botão `Criar credenciais`, escolha `ID do cliente OAuth` como tipo de credencial, e configure as informações do OAuth de acordo com as necessidades do projeto e seguindo as intruções.

3. Preencha o campo `Tipo de aplicativo` com a opção `Aplicativo da Web`, nas seções `Origens JavaScript autorizadas` e `URLs de redirecionamento autorizados`, adicione URLs de acordo com a natureza do aplicativo, por exemplo, vale colocar `http://localhost:3000` e `http://localhost:3000/api/auth/callback/google` respectivamente, para uma aplicação que está rodando localmente. E , por fim ,clique em `Criar`.
   OBS:A segunda URL depende de como foram implementadas as rotas no projeto!!

4. Após a criação do cliente OAuth, serão exibidos O `ID do cliente` e a `Chave secreta do cliente`.

5. Por fim, crie um arquivo `.env` para guardar essas informações.Como no exemplo:

```bash
# Next Auth
# You can generate a new secret on the command line with:
# openssl rand -base64 32
# https://next-auth.js.org/configuration/options#secret
NEXTAUTH_SECRET="tefsdfadagdsdfagdf123413afadf"

NEXTAUTH_URL="http://localhost:3000"

# Next Auth GOOGLE Provider
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
```

Com as credenciais armazenadas de forma segura em seu arquivo .env, você pode configurar a autenticação com o provedor do Google de maneira segura e eficaz em seu projeto Next Auth.

### Credentials (Interno)

A autenticação por Credenciais permite lidar com o login usando credenciais arbitrárias, como nome de usuário e senha, ele vem com a restrição de que os usuários autenticados dessa maneira não se mantém no banco de dados e, consequentemente, é necessário o uso de tokens(id) para implementar a validação. Por exemplo:

!!!warning
É **necessário ter um serviço de autenticação externo, ou criar um do zero**, para poder usar o CredentialsProvider de forma realmente útil! O NextAuth nesse caso é muito menos útil! Considere recorrer a outras bibliotecas, como [Lucia Auth](https://lucia-auth.com/).
!!!

```js

import CredentialsProvider from "next-auth/providers/credentials";
...
providers: [
  CredentialsProvider({
    // O nome exibido no formulário de login (por exemplo, 'Entrar com...')
    name: "Credentials",

    // 'credentials' é usado para gerar um formulário na página de login.
    // Você pode especificar quais campos devem ser enviados, adicionando chaves ao objeto 'credentials'
    // por exemplo, domínio, nome de usuário, senha, token de autenticação de dois fatores, etc.
    // Você pode passar qualquer atributo HTML para a tag <input> por meio do objeto.
    credentials: {
      email: {
        label: "Email",
        type: "email",
        placeholder: "exemplo@gmail.com"
      },
      password: {
        label: "Password",
        type: "password",
        placeholder: `digite sua senha`,
      },
    },

    async authorize(credentials, req) {
      // verifica algum campo é vazio
      if (!credentials?.email || !credentials.password) return null;

      // Procura o usuário na database
      const user = await prisma.user.findUnique({
        where: { email: credentials.email },
      });

      if (user) {
        // Verifica se a senha passada é igual a senha da database
        // NENHUM POUCO SEGURO!!!
        if (credentials.password !== user.password)
          return null;
        else
          return user // Qualquer objeto retornado será salvo na propriedade user
      } else {
        // Se você retornar null, um erro será exibido, aconselhando o usuário a verificar seus dados.
        return null

        // Você também pode rejeitar este retorno de chamada com um erro, assim o usuário será
        // redirecionado para a página de erro com a mensagem de erro como um parâmetro de consulta
      }
    }
  })
]
...

```

## Proteção de Rotas

O next oferece a possibilidade da criação de arquivos `layout.tsx` para configuração de páginas que estão dentro de determinadas rotas/pastas.

Além disso, também podemos criar grupos de rotas capazes de ocultar parte dos endereços de acesso para o usuário. Para isso, basta criarmos uma pasta de rota nomeada entre parênteses.
Por exemplo, podemos separar nossas rotas em dois tipos agrupando todas páginas que necessitam de autenticação em uma única rota `(auth)` e rotas públicas em `(public)`.

Utilizando esses dois recursos juntamente com o nextauth, podemos criar rotas seguras que agrupam determinadas páginas protegidas por login. Para isso, podemos criar um arquivo `layout.tsx` como no exemplo abaixo:

```js src/app/(user)/layout.tsx
import { getServerAuthSession } from "~/server/auth";
import { permanentRedirect } from "next/navigation";
import { AuthProvider } from "~/components/authProvider";

export default async function AuthLayout({
  children,
}: {
  children: React.ReactNode,
}) {
  const session = await getServerAuthSession();

  if (session?.user) {
    //permanentRedirect não deixa retornar
    permanentRedirect("/login");
  }

  return (
    <>
      <AuthProvider>{children}</AuthProvider>;
    </>
  );
}
```

Nesse exemplo, temos um grupo de rotas chamado `(user)` que agrupa todas páginas relacionadas ao perfil de um usuário qualquer. O arquivo layout define que para acessar quaisquer uma dessas páginas, uma sessão precisa existir, ou seja, o usuário deve estar estar logado, senão ele é redirecionado para a página de login.

Dentro do grupo `(user)` podemos ter uma página com rota dinâmica para o perfil do usuário e uma página de login genérica. Nessa página de login, de maneira análoga ao layout, direcionamos o usuário para seu perfil caso ele já esteja logado.

```js src/app/(user)/login/page.tsx
import { permanentRedirect } from "next/navigation";
import SignInButton from "~/components/user/signIn";
import { getServerAuthSession } from "~/server/auth";

export default async function LoginPage() {
  const session = await getServerAuthSession();

  if (session?.user) {
    permanentRedirect(`/profile/${session.user.email}`);
  }

  return (
    <>
      <h1>Página de Login Foda</h1>
      <SignInButton />
    </>
  );
}
```

```js src/app/(user)/profile/[email]/page.tsx
import { getServerAuthSession } from "~/server/auth";

export default async function ProfilePage() {
  const session = await getServerAuthSession();
  return (
    <>
      <h1>Página do Usuário</h1>

      <span>Meu Nome: {session?.user.name}</span>
      <span>Meu Email: {session?.user.email}</span>
      <span>Minha Senha: LOL</span>
    </>
  );
}
```

!!!
A página `profile` é acessada a partir de uma rota dinâmica criada a partir do email do usuário na sessão, mas poderia ser feita utilizando outros dados personalizados como um `username`, por exemplo.
!!!
