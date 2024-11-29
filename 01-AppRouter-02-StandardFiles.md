# Arquivos Especiais no App Router

## 1. `page.tsx`
É o arquivo que transforma um diretório em uma rota pública. Define a UI única para uma rota específica.

```typescript
// app/about/page.tsx
export default function AboutPage() {
  return <div>Sobre nós</div>
}
```

Particularidades:
- Deve ser exportado como default
- Pode ser um Server ou Client Component
- Pode ser assíncrono para fetch de dados
- É a única forma de criar rotas públicas

## 2. `layout.tsx`
Define uma UI compartilhada para múltiplos pages. Layouts são aninhados e persistem durante a navegação.

```typescript
// app/blog/layout.tsx
type BlogLayoutProps = {
  children: React.ReactNode
}

export default function BlogLayout({ children }: BlogLayoutProps) {
  return (
    <div>
      <nav>Menu do Blog</nav>
      {children}
      <footer>Footer do Blog</footer>
    </div>
  )
}
```

Particularidades:
- O children renderiza a page ou outro layout aninhado
- Preserva estado durante navegações
- Não é re-renderizado quando apenas o children muda
- Pode buscar dados
- Por padrão é Server Component
- Não pode aceitar parâmetros de rota

## 3. `template.tsx`
Similar ao layout, mas cria uma nova instância para cada navegação. Útil quando:
- Precisa de efeitos de montagem/desmontagem
- Quer usar transições de página
- Quer ver comportamentos específicos de framework que dependem do efeito de montagem

```typescript
// app/blog/template.tsx
'use client'

type TemplateProps = {
  children: React.ReactNode
}

export default function Template({ children }: TemplateProps) {
  useEffect(() => {
    // Este efeito roda em cada navegação
    console.log('Template montado')
    
    return () => console.log('Template desmontado')
  }, [])

  return (
    <div className="animate-fade-in">
      {children}
    </div>
  )
}
```

Diferenças principais entre `template.tsx` e `layout.tsx`:
- Template cria nova instância em cada navegação
- Estado não é preservado em templates
- Efeitos são re-executados
- DOM é re-criado

## 4. `loading.tsx`
Cria uma UI de loading usando Suspense por baixo dos panos.

```typescript
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="animate-spin rounded-full h-32 w-32 border-t-2 border-b-2 border-gray-900">
        Carregando dashboard...
      </div>
    </div>
  )
}
```

Particularidades:
- Mostrado automaticamente quando a page está carregando
- Encapsulado em Suspense automaticamente
- Pode ser usado com streaming SSR
- Aplica-se a todos os children da rota

## 5. `error.tsx`
Manipula erros em runtime para um segmento de rota.

```typescript
'use client'

type ErrorBoundaryProps = {
  error: Error & { digest?: string }
  reset: () => void
}

export default function ErrorBoundary({ error, reset }: ErrorBoundaryProps) {
  useEffect(() => {
    // Log do erro para um serviço de monitoramento
    console.error(error)
  }, [error])

  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2>Algo deu errado!</h2>
      <p>{error.message}</p>
      <button 
        onClick={reset}
        className="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
      >
        Tentar novamente
      </button>
    </div>
  )
}
```

Particularidades:
- Deve ser Client Component
- Recebe error e reset como props
- Aninhado: erros borbulham até encontrar o próximo error boundary
- Pode ser usado com `global-error.tsx` para erros no root layout

## 6. `not-found.tsx`
Mostra quando uma rota não é encontrada ou quando `notFound()` é chamado.

```typescript
// app/not-found.tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2>Página não encontrada</h2>
      <p>Não foi possível encontrar o recurso solicitado</p>
      <Link 
        href="/"
        className="mt-4 px-4 py-2 bg-blue-500 text-white rounded"
      >
        Voltar para Home
      </Link>
    </div>
  )
}
```

Particularidades:
- Pode ser usado globalmente ou por segmento
- Ativado automaticamente em 404s
- Pode ser ativado manualmente com `notFound()`
- Recebe status 404 automaticamente

## 7. `route.ts`
Define API endpoints usando o Web Request/Response API.

```typescript
// app/api/users/route.ts
import { NextRequest } from 'next/server'

export async function GET(request: NextRequest) {
  const users = await db.users.findMany()
  return Response.json(users)
}

export async function POST(request: NextRequest) {
  const data = await request.json()
  
  const newUser = await db.users.create({
    data
  })
  
  return Response.json(newUser, { status: 201 })
}
```

Particularidades:
- Suporta GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
- Acesso direto ao Request/Response API
- Pode usar middlewares
- Não renderiza UI
- Útil para APIs e manipulação de dados
