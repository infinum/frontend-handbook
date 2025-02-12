## What is NextAuth?

[NextAuth](https://next-auth.js.org/) is an open-source authentication library designed specifically for Next.js applications. It streamlines adding authentication to your projects by providing a robust, flexible, and secure solution that integrates directly with Next.js API routes and pages. With built-in support for popular OAuth providers, email/passwordless login, and custom credential systems, NextAuth makes it easier to implement Single Sign-On (SSO) and other authentication patterns without reinventing the wheel.

It simplifies authentication by handling session management, token issuance, and secure communication between your client and server. It leverages Next.js’s file-based API routing to create endpoints (e.g., `/api/auth/[...nextauth]`) that manage sign-in, sign-out, and session verification automatically.

### How it works

* **Providers**: NextAuth supports a variety of authentication providers—OAuth (Google, GitHub, Facebook, etc.), email-based sign-ins, and custom credentials. This means you can easily integrate third-party SSO or build your own login flow.
* **Session Management**: Once authenticated, NextAuth creates a session that can be stored either in a JSON Web Token (JWT) or in a database-backed session store. The session object contains the user’s information and any additional claims you decide to include.
* **API Routes**: By utilizing Next.js API routes, NextAuth manages the authentication endpoints, handling tasks like token refresh, CSRF protection, and secure cookie management out-of-the-box.
* **Callbacks & Events**: NextAuth provides a variety of callbacks (e.g., for sign-in, JWT creation, session retrieval) that allow you to customize the authentication flow, such as adding custom claims or performing side effects like logging.

### Use cases

NextAuth can be used in various scenarios:

* **Single Page Applications**: Secure client-side rendered apps with minimal configuration.
* **Server-Side Rendering**: Integrate authentication in SSR workflows by accessing sessions on the server before rendering pages.
* **API Security**: Protect API routes by verifying sessions or JWT tokens.
* **Multi-Provider SSO**: Allow users to authenticate using their preferred identity provider (Google, GitHub, etc.) or via passwordless methods.

### Pros and Cons

**Pros**

* **Tailored for Next.js**: Seamless integration with Next.js features like API routes and SSR.
* **Out-of-the-box Security**: Comes with built-in protections (CSRF, secure cookie handling) that follow modern best practices.
* **Flexibility with Providers**: Supports many OAuth providers and custom credential systems.
* **Customizability**: Callbacks allow for tailoring the authentication flow, adding roles, or injecting additional user data.
* **Community and Documentation**: A vibrant community and extensive documentation make troubleshooting and customization straightforward.

**Cons**

* **Opinionated**: NextAuth makes certain assumptions about the authentication flow, which may not fit all use cases. Highly customized workflows might require additional work.
* **Next.js Centric**: While excellent for Next.js, its design does not lend itself well to non-Next.js projects.
* **Abstraction Overhead**: For very simple projects, using NextAuth might feel like overkill compared to implementing minimal custom logic.
* **Complexity in Advanced Scenarios**: Advanced use cases (like intricate multi-tenant setups) can lead to more complex configurations and potential gotchas.

## Basic Setup

### Installation

Install NextAuth and any necessary provider packages:

```bash
pnpm add -E next-auth
```

> If you’re using a specific provider (e.g., Google), make sure to install any required packages as outlined in the [NextAuth Providers documentation](https://next-auth.js.org/providers/).

### Secret Key

In order to automatically generate an example `AUTH_SECRET` in your `.env.local` file, you can run:

```bash
pnpx auth secret
```

> ⚠️ Warning! For Production environments make sure you're using ASYMMETRIC `RS256` 3072-bit key - you can read more about Security of JWT [here](https://cyberpolygon.com/materials/security-of-json-web-tokens-jwt/). If you want to hide the data inside token, use `JWE`.

### Configuration

Create configuration file at `/lib/auth.ts` - include which providers you want to use and any session settings.

```ts
import {
  getServerSession as getNextAuthServerSession,
  NextAuthOptions,
} from "next-auth";
import GitHubProvider from "next-auth/providers/github";
// ...import any other providers or config as needed

export const authOptions: NextAuthOptions = {
  providers: [
    GitHubProvider({
      clientId: process.env.GITHUB_ID || "",
      clientSecret: process.env.GITHUB_SECRET || "",
    }),
    // ...add more providers here
  ],
  // You can choose between JWT or DB sessions
  session: { strategy: "jwt" }, // (default strategy)
  
  // callbacks let you customize or enrich the session
  callbacks: {
    async session({ session, token, user }) {
      // e.g. attach a "role" to the session
    //   session.user.role = user.role;
      return session;
    },
  },
  // ...more NextAuth configuration
};

// For better reusability, encapsulate the session logic in a separate hook
export const getServerSession = () => getNextAuthServerSession(authOptions);
```

### Authentication endpoint

Create a route handler at `/app/api/auth/[...nextauth]/route.ts`:

```ts
import NextAuth from "next-auth";
import { authOptions } from "@/lib/auth";

const handler = NextAuth(authOptions);

export { handler as GET, handler as POST };
```

## Using NextAuth in components

### Client components

You can consume the session in your client components using `next-auth/react` hooks.

```jsx
"use client";

import { useSession, signIn, signOut } from "next-auth/react";

export default function HomePage() {
  const { data: session, status } = useSession();

  if (status === "loading") {
    return <p>Loading...</p>;
  }

  if (session) {
    return (
      <div>
        <p>Signed in as {session.user?.email}</p>
        <button onClick={() => signOut()}>Sign out</button>
      </div>
    );
  }

  return (
    <div>
      <p>You are not signed in.</p>
      <button onClick={() => signIn("github")}>Sign in with GitHub</button>
    </div>
  );
}
```

### Server Components

In Server Components (or server-side logic), you can use `getServerSession` from `next-auth` to ensure data is fetched only for authenticated users.

```jsx
import { getServerSession } from "@/lib/auth";

export default async function DashboardPage() {
  const session = await getServerSession();

  if (!session) {
    // You can redirect or throw an error
    return <div>Please sign in to access your dashboard.</div>;
  }

  return <div>Welcome to the dashboard, {session.user?.name}!</div>;
}
```

## Best practices & gotchas

1. Use HTTPS Everywhere
   * Always serve your app over HTTPS to ensure secure cookie transmission.
2. Secure Cookies
   * By default, NextAuth sets httpOnly, sameSite cookies. Keep these settings to limit XSS/CSRF attack vectors.
3. Token Rotation
   * If you enable JWT sessions, consider token rotation or refresh tokens for better security.
4. Custom Callbacks
   * Enrich the session object with user roles or data.
   * Map external provider data to your custom user fields.
   * Handle token rotation or advanced encryption logic.
5. Stay Up to Date
   * NextAuth is actively developed. Watch the [changelog](https://github.com/nextauthjs/next-auth/releases) for new features and security updates.

### Gotchas

1. Route Handlers vs. Pages
   * If you used older patterns (e.g., `/pages/api/auth/\[...nextauth].ts`), switch to App Router route handlers.
2. Database Requirements
   * If you use a database session strategy, make sure the schema is set up (NextAuth can generate it for certain DBs).
3. Provider Rate Limits
   * Social providers might rate-limit logins if you do lots of short-interval sign-ins.
4. CSRF and Custom Forms
   * Credential-based sign-ins require anti-CSRF tokens, which NextAuth handles automatically, but be cautious if you implement fully custom forms.
5. Deploying on Serverless
   * NextAuth works on platforms like Vercel seamlessly, but if you use custom serverless hosts, check for any environment-specific limitations.

## NextAuth + CASL

[CASL](https://casl.js.org/v6/en/) is a popular library for Role/Permission-based Access Control. It defines “abilities” that specify what a user can or cannot do in your application. Typically, you’d combine your authentication solution (who is the user? are they logged in?) with an authorization layer (what is the user allowed to do?).

You can combine CASL and NextAuth together, because they serve different purposes:

* **NextAuth**: Authenticates a user, creates a session, provides user identity data.
* **CASL**: Defines abilities (permissions/roles) based on that user’s data or role.

### Configuring CASL with NextAuth

**Install dependencies**

```bash
pnpm add -E @casl/ability @casl/react
```

**Add roles to the session**

In NextAuth, use the `callbacks.session` function to add user roles or permissions into the session object. For example:

```ts
async session({ session, user }) {
  session.user.role = user.role; 
  return session;
},
```

**Define abilities with CASL**

```ts
// lib/casl.ts
import { AbilityBuilder, Ability } from "@casl/ability";

export default function defineAbilitiesFor(user) {
  const { can, cannot, build } = new AbilityBuilder(Ability);

  if (user.role === "admin") {
    can("manage", "all");
  } else {
    can("read", "Post");
    can("delete", "Post", { authorId: user.id }); // Only allow deleting own posts
    cannot("delete", "Post").unless({ authorId: user.id }); // Prevent deleting others' posts
    // etc.
  }

  return build();
}
```

**Use the session**

When a user logs in via NextAuth, call `defineAbilitiesFor(session.user)` to create a CASL “ability” instance. Then check permissions in your components, API routes, or server logic.

### Example real-world configuration:

> Remember to add the `[...nextauth]` route handler before the rest of configuration files!

**CASL config**:

```ts
// lib/casl.ts
import { AbilityBuilder, PureAbility } from "@casl/ability";

type Actions = "manage" | "create" | "read" | "update" | "delete";
type Subjects = "Users" | "Posts" | "all"; // Example domain models

export type AppAbility = PureAbility<[Actions, Subjects]>;

export function defineAbilitiesFor(params: { role: string; userId: string }) {
  const { can, cannot, build } = new AbilityBuilder<AppAbility>(PureAbility);

  const { role } = params;

  if (role === "admin") {
    // Admin can do everything
    can("manage", "all");
  } else if (role === "editor") {
    can("read", "Posts");
    can("create", "Posts");
    can("update", "Posts");
    cannot("delete", "Posts");
  } else {
    // role === "user"
    can("read", "Posts");
    cannot("create", "Posts");
    cannot("delete", "Posts");
    // etc.
  }

  return build();
}
```

**NextAuth config**:

```ts
// lib/auth.ts
import {
  getServerSession as getNextAuthServerSession,
  NextAuthOptions,
} from "next-auth";
import CredentialsProvider from "next-auth/providers/credentials";

const MOCK_USER = {
  id: "1",
  name: "John Doe",
  email: "john@example.com",
  hashedPassword: "hashedPassword",
  role: "user",
};

const findUserByEmail = async (_email: string) => {
  return MOCK_USER;
};
const verifyPassword = async (_password: string, _hashedPassword: string) => {
  return true;
};

export const authOptions: NextAuthOptions = {
  session: {
    strategy: "jwt",
  },
  providers: [
    CredentialsProvider({
      name: "Credentials",
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" },
      },
      async authorize(credentials, req) {
        if (!credentials?.email || !credentials?.password) {
          throw new Error("Missing username or password");
        }

        const user = await findUserByEmail(credentials.email);
        if (!user) {
          // Remember to never leak if there's actually a user with given email, always show the end-user "Invalid password" error!
          throw new Error("User not found");
        }

        const isValid = await verifyPassword(
          credentials.password,
          user.hashedPassword
        );
        if (!isValid) {
          throw new Error("Invalid password");
        }

        // Return a "safe" user object. NextAuth will store this in JWT token
        return {
          id: user.id,
          name: user.name,
          email: user.email,
          // User role used by CASL
          role: user.role,
        };
      },
    }),
  ],
  callbacks: {
    async jwt({ token, user }) {
      // If `user` is defined, it means we're in the process of the user signing in
      if (user) {
        token.user = user;
      }
      return token;
    },
    async session({ session, token }) {
      // Add the user role to the session for CASL
      if (session.user && token) {
        session.user = token.user;
      }
      return session;
    },
  },
  // Optionally, add pages if you want custom error or signIn pages
  // pages: {
  //   signIn: '/login',
  //   error: '/login?error=CredentialsSignin', // example
  // },
};

// For better reusability, encapsulate the session logic in a separate hook
export const getServerSession = () => getNextAuthServerSession(authOptions);
```

**Typescript [Module Augmentation](https://next-auth.js.org/getting-started/typescript#module-augmentation)**:

Create `src/typings/next-auth.d.ts` file:

```ts
/// <reference types="next-auth" />

import type { DefaultSession, DefaultUser } from "next-auth";
import type { DefaultJWT } from "next-auth/jwt";

type AppUser = DefaultUser & {
  id: string;
  name: string;
  email: string;
  role: string;
};

declare module "next-auth" {
  interface User extends AppUser {
    // Strictly override the base type "string | null | undefined" with "string"
    name: string;
    email: string;
  }

  interface Session extends DefaultSession {
    user: AppUser;
  }
}

declare module "next-auth/jwt" {
  interface JWT extends DefaultJWT {
    user: AppUser;
  }
}
```

**Example server component**:

```jsx
import { getServerSession } from "@/lib/auth";
import { defineAbilitiesFor } from "@/lib/casl";

export default async function DashboardPage() {
  const session = await getServerSession();

  if (!session?.user) {
    return <div>Please sign in first.</div>;
  }

  const ability = defineAbilitiesFor({
    role: session.user.role,
    userId: session.user.id,
  });

  // Example usage
  const canCreatePost = ability.can("create", "Posts");

  return (
    <div>
      <h1>Welcome, {session.user.name}!</h1>
      <p>Your role: {session.user.role}</p>

      {canCreatePost ? (
        <div>Show "Create Post" button or form here.</div>
      ) : (
        <p>You do not have permission to create posts.</p>
      )}
    </div>
  );
}
```

**Example client component**:

```jsx
"use client";

import { useSession, signIn, signOut } from "next-auth/react";
import { useState } from "react";

export default function LoginPage() {
  const { data: session, status } = useSession();
  const [credentials, setCredentials] = useState({ email: "", password: "" });

  if (status === "loading") {
    return <p>Loading...</p>;
  }

  if (session) {
    return (
      <div>
        <p>Signed in as {session.user.email}</p>
        <p>Your role is: {session.user.role}</p>
        <button onClick={() => signOut()}>Sign out</button>
      </div>
    );
  }

  return (
    <form>
      <label>
        Email
        <input
          type="email"
          value={credentials.email}
          onChange={(e) =>
            setCredentials({ ...credentials, email: e.target.value })
          }
        />
      </label>
      <label>
        Password
        <input
          type="password"
          value={credentials.password}
          onChange={(e) =>
            setCredentials({ ...credentials, password: e.target.value })
          }
        />
      </label>
      <button
        type="submit"
        onClick={(e) => {
          e.preventDefault();
          signIn("credentials", {
            email: credentials.email,
            password: credentials.password,
          });
        }}
      >
        Sign in
      </button>
    </form>
  );
}
```

**Example route handler**:

```ts
// app/api/protected-resource/route.ts
import { NextResponse } from "next/server";
import { getServerSession } from "@/lib/auth";
import { defineAbilitiesFor } from "@/lib/casl";

export async function GET() {
  const session = await getServerSession();

  if (!session?.user) {
    return NextResponse.json({ error: "Not authenticated" }, { status: 401 });
  }

  const ability = defineAbilitiesFor({
    role: session.user.role,
    userId: session.user.id,
  });

  // For example, we check if the user can "read" a "Post"
  if (!ability.can("read", "Posts")) {
    return NextResponse.json({ error: "Forbidden" }, { status: 403 });
  }

  // Otherwise, proceed with returning the resource
  return NextResponse.json({ message: "Here is the protected data" });
}
```
