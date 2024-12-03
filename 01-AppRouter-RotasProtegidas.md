# Exemplo Rotas Protegidas com Autenticação

## [0] Criando novo projeto

```
npx create-next-app@latest teste-login --typescript --tailwind --app
cd teste-login
```

## [1] Criando o Contexto de Autenticação

```src/contexts/AuthContext.tsx```

```javascript
import { createContext, useContext, useState, ReactNode } from "react"
import Cookies from "js-cookie"

interface AuthContextType {
  isAuthenticated: boolean
  error: string | null
  login: (email: string, password: string) => void
  logout: () => void
}

const AuthContext = createContext<AuthContextType | null>(null)

export function AuthProvider({ children }: { children: ReactNode }) {
  const [isAuthenticated, setIsAuthenticated] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const login = (email: string, password: string) => {
    if (email === "teste@teste.com" && password === "123456") {
      setIsAuthenticated(true)
      setError(null)
      Cookies.set("auth", "true")
    } else {
      setError("Email ou senha inválidos")
    }
  }

  const logout = () => {
    setIsAuthenticated(false)
    setError(null)
    Cookies.remove("auth")
  }

  return (
    <AuthContext.Provider value={{ isAuthenticated, error, login, logout }}>
      {children}
    </AuthContext.Provider>
  )
}

export const useAuth = () => {
  const context = useContext(AuthContext)
  if (!context) throw new Error("useAuth deve ser usado dentro de AuthProvider")
  return context
}
```

## [2] Criando o Middleware

```src/middleware.ts```

```javascript
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

const publicRoutes = ['/', '/login'];

export function middleware(request: NextRequest) {
  const isAuthenticated = request.cookies.has('auth'); // Simples verificação
  const isPublicRoute = publicRoutes.includes(request.nextUrl.pathname);

  if (!isAuthenticated && !isPublicRoute) {
    return NextResponse.redirect(new URL('/login', request.url));
  }

  if (isAuthenticated && request.nextUrl.pathname === '/login') {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }

  return NextResponse.next();
}

export const config = {
  matcher: ['/((?!api|_next/static|favicon.ico).*)'],
};
```

## [3] Página de Login

```src/app/login/page.tsx```

```javascript
"use client"
import { useState } from "react"
import { useAuth } from "@/contexts/AuthContext"
import { useRouter } from "next/navigation"

export default function Login() {
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const { login, error } = useAuth()
  const router = useRouter()

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault()
    login(email, password)
    router.push("/dashboard")
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100">
      <form
        onSubmit={handleSubmit}
        className="bg-white p-8 rounded-lg shadow-md w-96"
      >
        <h2 className="text-2xl font-bold mb-6 text-center">Login</h2>
        <div className="space-y-4">
          <input
            type="email"
            placeholder="Email"
            value={email}
            onChange={(e) => setEmail(e.target.value)}
            className="w-full p-2 border rounded"
          />
          <input
            type="password"
            placeholder="Senha"
            value={password}
            onChange={(e) => setPassword(e.target.value)}
            className="w-full p-2 border rounded"
          />

          {error && (
            <div className="mt-4 p-2 bg-red-100 text-red-600 rounded">
              {error}
            </div>
          )}
          <button
            type="submit"
            className="w-full bg-blue-500 text-white p-2 rounded hover:bg-blue-600"
          >
            Entrar
          </button>
        </div>
      </form>
    </div>
  )
}
```

## [4] Layout Principal

```src/app/layout.tsx```

```javascript
"use client"
import { AuthProvider } from "@/contexts/AuthContext"
import "./globals.css"

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="pt-BR">
      <body>
        <AuthProvider>{children}</AuthProvider>
      </body>
    </html>
  )
}
```

### [5] Dashboard (página só permitida com autenticação)

```src/app/dashboard/page.tsx```

```javascript
// src/app/dashboard/page.tsx
export default function Dashboard() {
  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold">Dashboard</h1>
      <p>Página protegida</p>
    </div>
  );
}
```

## Testando

Credenciais de teste:

Email: teste@teste.com
Senha: 123456


Rotas protegidas acessíveis apenas após login:

/dashboard


Rotas públicas:

/
/login


## Como funciona o Middleware

O middleware.ts no Next.js é um interceptador de requisições que atua antes das rotas serem acessadas. Vamos analisar cada parte:

```typescript
const publicRoutes = ['/', '/login'];
```
Define rotas que não precisam de autenticação.

```typescript
export function middleware(request: NextRequest) {
  const isAuthenticated = request.cookies.has('auth');
  const isPublicRoute = publicRoutes.includes(request.nextUrl.pathname);
```
- Verifica se existe cookie 'auth'
- Checa se a rota atual é pública

```typescript
  if (!isAuthenticated && !isPublicRoute) {
    return NextResponse.redirect(new URL('/login', request.url));
  }
```
Redireciona para login se:
- Usuário não autenticado
- Tentando acessar rota protegida

```typescript
  if (isAuthenticated && request.nextUrl.pathname === '/login') {
    return NextResponse.redirect(new URL('/dashboard', request.url));
  }
```
Redireciona para dashboard se:
- Usuário já autenticado
- Tentando acessar página de login

```typescript
export const config = {
  matcher: ['/((?!api|_next/static|favicon.ico).*)'],
};
```
Configura quais rotas o middleware deve interceptar, excluindo arquivos estáticos e API.
