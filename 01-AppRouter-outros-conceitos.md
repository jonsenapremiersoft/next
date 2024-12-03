# Router no Next.js

## 1. Configuração Básica

O Next.js usa um sistema de arquivos para roteamento:

```plaintext
app/
├── page.tsx            # Rota: /
├── about/
│   └── page.tsx       # Rota: /about
└── blog/
    └── [id]/          # Rota dinâmica
        └── page.tsx   # Rota: /blog/123
```

```typescript
// app/page.tsx
export default function Home() {
  // Página inicial - renderizada no servidor por padrão
  return (
    <div className="p-4">
      <h1 className="text-2xl font-bold">Bem-vindo</h1>
    </div>
  )
}

// app/blog/[id]/page.tsx
type BlogPostProps = {
  params: {
    id: string // O parâmetro dinâmico da URL
  }
}

export default function BlogPost({ params }: BlogPostProps) {
  // params.id contém o valor da URL
  // Ex: /blog/123 → params.id = "123"
  return (
    <article className="prose">
      <h1>Post {params.id}</h1>
    </article>
  )
}
```

## 2. Route Params e Query Strings

O Next.js oferece tipagem completa para parâmetros de rota e query strings:

```typescript
// app/products/[category]/[id]/page.tsx
type ProductPageProps = {
  params: {
    category: string  // /products/electronics/123
    id: string       // category="electronics", id="123"
  }
  searchParams: {     // ?sort=price&filter=new
    sort?: string    // searchParams.sort = "price"
    filter?: string  // searchParams.filter = "new"
  }
}

export default function ProductPage({ 
  params,
  searchParams 
}: ProductPageProps) {
  // Validação de tipos em tempo de compilação
  const { category, id } = params
  const { sort = 'default', filter } = searchParams

  return (
    <div className="container mx-auto p-4">
      <h1>Produto {id}</h1>
      <div className="mt-4">
        <p>Categoria: {category}</p>
        <p>Ordenação: {sort}</p>
        {filter && <p>Filtro: {filter}</p>}
      </div>
    </div>
  )
}
```

## 3. Protected Routes

Implementação robusta de autenticação usando middleware:

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { getToken } from './lib/auth'

export async function middleware(request: NextRequest) {
  const token = request.cookies.get('auth-token')
  const isAuthPage = request.nextUrl.pathname.startsWith('/login')

  try {
    // Verifica autenticação
    const isValidToken = token && await getToken(token.value)
    
    if (!isValidToken && !isAuthPage) {
      // Redireciona para login
      const loginUrl = new URL('/login', request.url)
      loginUrl.searchParams.set('from', request.nextUrl.pathname)
      return NextResponse.redirect(loginUrl)
    }

    if (isValidToken && isAuthPage) {
      // Usuário autenticado não deve acessar páginas de login
      return NextResponse.redirect(new URL('/dashboard', request.url))
    }

    return NextResponse.next()
  } catch (error) {
    // Em caso de erro, remove o token e redireciona
    const response = NextResponse.redirect(new URL('/login', request.url))
    response.cookies.delete('auth-token')
    return response
  }
}

export const config = {
  // Define quais rotas o middleware protege
  matcher: [
    '/dashboard/:path*',
    '/admin/:path*',
    '/login'
  ]
}
```

## 4. Nested Routes

O Next.js permite layouts aninhados sofisticados:

```typescript
// app/dashboard/layout.tsx
import { Sidebar } from '@/components/Sidebar'
import { Header } from '@/components/Header'

type DashboardLayoutProps = {
  children: React.ReactNode
  sidebar: React.ReactNode  // Slot para @sidebar
  header: React.ReactNode   // Slot para @header
}

export default function DashboardLayout({
  children,
  sidebar,
  header
}: DashboardLayoutProps) {
  return (
    <div className="min-h-screen flex flex-col">
      <div className="flex-none">
        {header}
      </div>
      <div className="flex-1 flex">
        <aside className="w-64 flex-none">
          {sidebar}
        </aside>
        <main className="flex-1 p-6">
          {children}
        </main>
      </div>
    </div>
  )
}

// app/dashboard/@sidebar/default.tsx
export default function DashboardSidebar() {
  return (
    <nav className="h-full bg-gray-50 p-4">
      <ul className="space-y-2">
        <li>Dashboard</li>
        <li>Configurações</li>
      </ul>
    </nav>
  )
}

// app/dashboard/page.tsx
export default function DashboardPage() {
  return (
    <div className="space-y-4">
      <h1 className="text-2xl font-bold">Dashboard</h1>
      <div className="grid grid-cols-3 gap-4">
        {/* Conteúdo do dashboard */}
      </div>
    </div>
  )
}
```

## 5. Navegação Programática

O Next.js oferece hooks poderosos para navegação:

```typescript
'use client'

import { useRouter, usePathname, useSearchParams } from 'next/navigation'
import { useState, useTransition } from 'react'

export default function NavigationExample() {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()
  const [isPending, startTransition] = useTransition()

  // Navegação com estado de loading
  const handleNavigation = () => {
    startTransition(() => {
      router.push('/new-page')
    })
  }

  // Atualização de query params
  const updateFilters = (newFilter: string) => {
    const params = new URLSearchParams(searchParams)
    params.set('filter', newFilter)
    router.replace(`${pathname}?${params.toString()}`)
  }

  // Navegação com dados
  const handleSubmit = async (formData: FormData) => {
    const result = await saveData(formData)
    
    if (result.success) {
      // Navegação após sucesso
      router.push('/success')
    } else {
      // Refresh da página atual
      router.refresh()
    }
  }

  return (
    <div className="space-y-4">
      <div className="flex gap-2">
        <button 
          onClick={() => router.back()}
          className="btn btn-secondary"
        >
          Voltar
        </button>
        
        <button 
          onClick={handleNavigation}
          className="btn btn-primary"
          disabled={isPending}
        >
          {isPending ? 'Carregando...' : 'Avançar'}
        </button>
      </div>

      <select 
        onChange={(e) => updateFilters(e.target.value)}
        className="form-select"
      >
        <option value="recent">Mais recentes</option>
        <option value="popular">Mais populares</option>
      </select>
    </div>
  )
}
```

## Conceitos Avançados

1. **Interceptação de Rotas**
   - Use `(..)` para interceptar rotas do nível acima
   - Use `(.)` para interceptar no mesmo nível
   - Útil para modais e overlays

2. **Grupos de Rotas**
   - Use `(grupo)` para organizar sem afetar a URL
   - Permite múltiplos layouts em uma mesma rota

3. **Layouts Paralelos**
   - Use `@nome` para renderizar componentes independentes
   - Cada slot pode ter sua própria navegação

4. **Carregamento e Erro**
   - `loading.tsx` para estados de carregamento
   - `error.tsx` para tratamento de erros
   - `not-found.tsx` para páginas 404

5. **Geração Estática**
   - Use `generateStaticParams` para rotas dinâmicas
   - Controle fino com `revalidate`

6. **Metadata**
   - Tipagem completa para SEO
   - Suporte a metadata dinâmica

