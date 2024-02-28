---
order: 1
icon: file-code
label: "Utilizando NextAuth em projetos"
author:
  - name: João Santos
  - name: Willyan Marques
date: 2024-02-28
category: Implementação
tags:
  - NextAuth
  - instalação
  - Next.js
  - autenticação
  - usuário
---

# Guia de uso NextAuth com T3

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

## Providers

O Next Auth possibilita que o critério de autenticação (provedores) seja por meio de credenciais customizáveis (nome, senha, email, etc) ou por provedores externos (Google, GitHub, Discord, etc).

### OAuth (Externo)

OAuth (Open Authorization) é um protocolo de autorização que permite que um aplicativo obtenha acesso limitado a uma conta em um serviço de terceiros sem revelar credenciais de login.

Ele é o principal processo de autenticação utilizando por empresas na atualidade e é oferecido pelo NextAuth através de provedores de login externos preexistentes. A partir do OAuth, o usúario é capaz de realizar a autenticação por meio de outra plataforma como Google, Github, Discord, etc.

Por serem processos externos de várias fontes diferentes, cada autenticação escolhida terá uma documentação específica diferente. Para ter mais detalhes sobre provedores específicos oferecidos pelo NextAuth acesse suas [respectivas documentações](https://github.com/nextauthjs/next-auth/tree/main/packages/next-auth/src/providers).

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

!!!warning
É **necessário ter um serviço de autenticação externo, ou criar um do zero**, para poder usar o CredentialsProvider de forma realmente útil! O NextAuth nesse caso é muito menos útil! Considere recorrer a outras bibliotecas, como [Lucia Auth](https://lucia-auth.com/) ou [bcrypt](https://www.npmjs.com/package/bcrypt) para criptografar senhas.
!!!

O `credentials provider` permite lidar com o login usando credenciais arbitrárias, como nome de usuário e senha. A validação de sessão com base no banco de dados deve ser realizada de forma por meio da função `authorize()` na configuração dos providers no NextAuth.

```js src/app/api/auth/[...nextauth]/route.ts

import CredentialsProvider from "next-auth/providers/credentials";
import { prisma } from "~/server/db";
import { compare } from "bcrypt"
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
      // Verifica algum campo é vazio
      if (!credentials?.email || !credentials.password) {
        return null //não lança erro
      };

      // Procura o usuário na database
      const user = await prisma.user.findUnique({
        where: { email: credentials.email },
      });

      if (!user) {
        return null
      };

      // Compara a senha passada com a senha do bd
      // PERIGO!!! Senha do bd não está criptografada!
      if (credentials.password !== user.password) {
        return null
      };

      // Qualquer objeto retornado será salvo na propriedade user
      return {
        id: user.id + '',
        name: user.name,
        email: user.email,
        image: user.image,
        isAdmin: user.isAdmin,
      }
    }
  })
]
...

```

!!!
Você também pode rejeitar este retorno de chamada com um erro, assim o `client-side` pode lidar com o erro dependendo do status e da mensagem passada como um parâmetro de consulta. No tratamento desse erro o usuário pode, por exemplo, receber um aviso ou ser redirecionado para uma página de registro, ajuda, etc.
!!!

## Proteção de Rotas

O next oferece a possibilidade da criação de arquivos `layout.tsx` para configuração de páginas que estão dentro de determinadas rotas/pastas.

Além disso, também podemos criar grupos de rotas capazes de ocultar parte dos endereços de acesso para o usuário. Para isso, basta criarmos uma pasta de rota nomeada entre parênteses.
Por exemplo, podemos separar nossas rotas em dois tipos agrupando todas páginas que necessitam de autenticação em uma única rota `(auth)` e rotas públicas em `(public)`.

Utilizando esses dois recursos juntamente com o NextAuth, podemos criar rotas seguras que agrupam determinadas páginas protegidas por login. Para isso, podemos criar um arquivo `layout.tsx` como no exemplo abaixo:

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
      <SessionProvider>{children}</SessionProvider>;
    </>
  );
}
```

Nesse exemplo, temos um grupo de rotas chamado `(user)` que agrupa todas páginas relacionadas ao perfil de um usuário qualquer. O `SessionProvider` engloba o `children`, definindo que para acessar quaisquer uma dessas páginas, uma sessão precisa existir. Ou seja, o usuário deve estar estar logado, senão ele é redirecionado para a página de login.

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

## Route Handler

Se você precisa buscar dados do servidor em um `client component`, você pode chamar um `Route Handler` (manipulador de rota).
Os manipuladores de rota são executados no servidor e retornam os dados para o cliente. Eles devem ser utilizados quando você não quer expor informações confidenciais ao cliente, como tokens.

No caso do NextAuth, temos um handler padrão definido que é executado sempre que a API de autenticação é chamada na rota `api/auth`. Ele é necessário para que o funcionamento das funções `SignIn()`, `SignOut()` entre outras coisas.

```js src/app/api/auth/[...nextauth]/route.ts
import NextAuth from "next-auth/next";
import { authOptions } from "~/server/auth";

const handler = NextAuth(authOptions);

// você pode adicionar mais exports compatíveis
export { handler as GET, handler as POST };
```

!!!
Projetos criados a partir da stack do T3 separam o `authOptions` do `route handler` por questão de organização e modularidade. Para saber mais sobre a configuração NextAuth acesse a [documentação do authOptions](https://next-auth.js.org/configuration/options#options). Veja também um exemplo de uso em [Session Callback](#session-callback)
!!!

## Session

<!-- !!!warning **Atenção:**

Caso você tenha criado o projeto utilizando a **stack do T3** _não é necessário_ configurar o nextauth manualmente.

Caso contário, verifique a versão do Next.js utilizada no projeto e se o tipo de roteamento usado é o `Pages Router` ou o `App Router`, pois o modo de implementação é diferente.
!!! -->

### Controle

O NextAuth provê medidas de controle de sessão para que você consiga iniciar (SignIn) e encerrar (SignOut) sessões facilmente.

!!!warning
Não existe função SignUp para registro de usuários! Caso você esteja utilizando o `credentials provider` e queira registrar um usuário, você deve fazer isso manualmente
!!!

#### SignIn()

!!!
Execução:

- Client-Side: **YES**

- Server-Side: **NO**

!!!

Por padrão, ao chamar o método `signIn()` sem argumentos, você será redirecionado para a página de login do NextAuth dentro da rota `/api/auth/session`. Essa página possui uma interface básica que lista todos os provedores de autenticação inseridos no `authOptions`, mas não pode ser personalizada e expõe a rota api ao usuário.

##### Provedor externo

Você pode redirecionar o usuário diretamente para a página seu provedor de autenticação externo passando o `id` do provedor dentro de `signIn()`.
Além disso, você pode especificar para qual página o usuário irá retornar após o login passando uma Url no segundo argumento da função.

```js googleSignInButton.tsx
import { signIn } from "next-auth/react";

export default function GoogleSignInButton() {
  return (
    <button
      onClick={() => signIn("google", { callbackUrl: "/user/profile" });
      }
    >
      Sign in with Google
    </button>
  );
}
```

#### Provedor interno

No caso do `crendentials` ou `email` provider, além das opções acima, você tem a possibilidade lidar com a resposta do `SignIn()` na **sua própria página de login/cadastro** em que a função é chamada ao **desabilitar seu redirecionamento padrão**. Por exemplo, se ocorrer um erro (como credenciais incorretas fornecidas pelo usuário), você pode querer tratar o erro na mesma página. Para isso, você pode passar `redirect: false` dentro do segundo parâmetro da função.

!!!warning
A opção de `redirect` só funciona para `credentials` e `email` providers.

!!!

```js login/page.tsx
import { signIn, useSession } from "next-auth/react";
import { useState } from "react";
export default function LoginPage() {
  //hook para ter acesso ao resultado do SignIn
  const [signInResponse, setSignInResponse] = useState(null);
  const [password, setPassword] = useState(null);

  const handleSignIn = async () => {
    const response = await signIn("credentials", {
      redirect: false,
      // callbackUrl: "/user/profile",
      password: password,
    });
    setSignInResponse(response);
  };

  // Formulário de login
  // ...
  // ...
  // Botão de login:
  <button onClick={handleSignIn}>Sign in</button>;
}
```

#### SignOut()

!!!
Execução:

- Client-Side: **YES**

- Server-Side: **NO**

!!!

Quando chamado, o método `signOut()` encerra a sessão e por padrão redireciona o usuário à página inicial (`/`). Como na função de login você pode especificar o `callbackUrl` dentro do segundo parâmetro ou desativar o redirecionamento utilizando `redirect: false`.

```js signOutButton.tsx
import { signOut } from "next-auth/react";

export default function SignOutButton() {
  return (
    <button
      onClick={() => signOut({ redirect: true, callbackUrl: "/homepage" })}
    >
      Sign out
    </button>
  );
}
```

!!!
Se após o logout você precisar redirecionar para outra página sem recarregar a atual, você pode capturar a resposta da chamada da função: `const response = await signOut({redirect: false, callbackUrl: "/homepage"})`.

Nessa linha, `response.url` é a URL validada para a qual você pode redirecionar o usuário, usando `useRouter().push(response.url)` do Next.js.
!!!

### Session Fetching

#### useSession()

!!!
Execução:

- Client-Side: **YES**

- Server-Side: **NO**

!!!

O `useSession` é um importante hook do React que é utilizado nas aplicações Next Auth para recuperar informações da sessão de usuário. Para utilizá-lo primeiro é preciso expor o conteudo da sessão de usuário por meio do `<SessionProvider>`. Para saber como utilizar o `SessionProvider`, vá para a seção [Proteção de Rotas](#proteção-de-rotas).

##### Exemplo

O Hook do React `useSession` pode ser usado como uma maneira simples de verificar se alguém está autenticado ou não:

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

A ideia é armazenar os resultados do `useSession` em props para aferir se o usuário ja foi autenticado, assim você pode criar componentes que se comportam de maneiras diferentes caso o usuário esteja logado ou não. Um outro exemplo de uso poderia ser uma foto de perfil de uma foto de perfil de uma navbar que mostra um icone default ou a foto do usuário dependendo da sessão.

#### getServerSession()

!!!
Acesso:

- Client-Side: **NO**

- Server-Side: **YES**

!!!

Como o nome já diz, o `getServerSession` é utilizado para obter a sessão no `server-side`, fazendo os fetches e renderizações no servidor antes de serem passados para o usuário. Geralmente ele é utilizando em contextos de `Route Handlers`, `React Server Components`, `rotas API` ou no `getServerSideProps`

##### Exemplo (App Router)

Para pegar os dados no server side dentro das páginas no `App Router`, você precisa importar o `authOptions` e passar dentro da função.

```js
import { getServerSession } from "next-auth/next";
import { authOptions } from "~/server/auth";

export default async function Page() {
  const session = await getServerSession(authOptions);
  ...
}
```

#### getServerAuthSession() [_T3 stack_]

!!!
Execução:

- Client-Side: **NO**

- Server-Side: **YES**

!!!

A stack do T3 oferece a função auxiliar `getServerAuthSession` para fazer o pre-fetch da sessão no server-side. Essa função é parecida com a `getServerSession`, mas com a vantagem de que você não precisa importar o `authOptions` toda vez que você precisar realizar um pre-fetch da sessão. Assim como no caso da função do next-auth, os dados da sessão podem ser passados para o client-side por peio do `getServerSideProps`:

```js
import { permanentRedirect } from "next/navigation";

async function getData(ctx) {
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

## Callbacks

### Session Callback

O `session callback` é chamado sempre que uma sessão é verificada (`getSession()`, `getServerSession()`, `/api/auth/session`, etc).

**Por padrão, apenas uma parte do token é retornado como medida de segurança**, mas podemos facilmente personalizar quais dados de sessão são retornados. Para isso, a stack do T3 traz uma conexão pronta entre a sessão e a database a partir da model `User` definida no Prisma e com isso podemos passar parâmetros extras e acessá-los utilizando os `fetches` de sessão no `client-side`.

Podemos também passar parâmetros extras retirados da database e passá-los junto com os dados de sessão padrão (dependem do `Session Provider`, no caso do google são `email`, `name` e `image`)

```go
model User {
    id            String    @id @default(cuid())
    name          String?
    email         String?   @unique
    emailVerified DateTime?
    image         String?
    accounts      Account[]
    sessions      Session[]
}

```

Para fazer com que o session inclua esse novo parâmetro, basta ir no arquivo de configurações do NextAuth em `src/server/auth.ts` e adicionar esse parâmetro ao `session callback` do `authOptions`:

```js
export const authOptions: NextAuthOptions = {
  callbacks: {
    session: ({ session, user }) => ({
      ...session, // parâmetros default da session
      user: {     // usuário da session
        ...session.user, // parâmetros default
        id: user.id      // novo parâmetro para o objeto user da session
      },
    }),
  },
```

Quando você adicionar esse novo parâmetro ele já será passado dentro da sessão, mas o typescript apontará um erro de tipagem não segura. Para corrigir será necessário declarar uma nova interface para o user object da sessão no módulo `next-auth`:

```js
declare module "next-auth" {
  interface Session extends DefaultSession {
    user: {
      id: string; // novo parâmetro (session.user.isAdmin)
    } & DefaultSession["user"];
  }

  // ...novas propriedades
  interface User {
    id: string; // novo parâmetro
  }
}
```

!!!warning Somente atualizar a interface mais externa não atualiza as demais interfaces que a utilizam. No caso do exemplo acima, será necessário atualizar o `user` dentro de `Session` por conta da tipagem.
!!!

### SignIn Callback

O `SignIn Callback` é chamado sempre que uma requisição de login é feita a um provedor do NextAuth.

Quando você utiliza o NextAuth.js com uma database, o objeto `User` será um objeto da database (id, name, isAdmin, etc.) se o usuário já realizou login anteriormente, senão ele será um protótipo simples.
Por exemplo, quando o usuário realiza login pelo Google sem conexão com a database, o `session.user` possui os parâmetros _name_, _email_ e _image_.

Já no caso do `Credentials Provider` o objeto `user` será a resposta do `authorize callback` e o objeto `profile` será a resposta do HTTP POST.

Você pode utilizar o `signIn() callback` para verificar se um usuário tem permissão para fazer login:

```js src/app/api/auth/[...nextauth]/route.ts
// ...
callbacks: {
  async signIn({ user, account, profile, email, credentials }) {
    // caso em que o NextAuth está conectado com a database
    if (user.isAdmin) {
      return true
    };
    else {
      // Você pode retornar uma URL de redirecionamento
      // Ou pode retornar `false` para mostrar uma mensagem de erro padrão
      return "/unauthorized"
    }

  }
}
// ...
```
