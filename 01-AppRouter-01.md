# App Router

## Introdução
O App Router representa uma mudança fundamental na arquitetura do Next.js, introduzindo um novo paradigma de desenvolvimento baseado em React Server Components (RSC). Esta mudança não é apenas uma atualização incremental, mas uma reimaginação completa de como construímos aplicações web modernas.

## Fundamentos do App Router

### Arquitetura Básica
O App Router utiliza uma estrutura baseada em diretórios dentro da pasta `app`, onde cada pasta representa um segmento de rota. Esta abordagem, conhecida como "file-system based routing", é mais intuitiva e oferece maior controle sobre o comportamento da aplicação.

```typescript
// Estrutura básica de diretórios
app/
├── layout.tsx            // Layout raiz da aplicação
├── page.tsx             // Página inicial
├── globals.css          // Estilos globais
├── types/               // Tipos TypeScript compartilhados
│   └── index.ts
├── components/          // Componentes reutilizáveis
│   ├── ui/             // Componentes de UI
│   └── features/       // Componentes específicos de features
└── lib/                // Utilitários e configurações
    └── utils.ts
```

## Server Components vs Client Components

### Server Components (Padrão)
Server Components são o padrão no App Router e oferecem várias vantagens:

```typescript
// app/components/ServerComponent.tsx
import { headers } from 'next/headers'
import { sql } from '@vercel/postgres'

type User = {
  id: number
  name: string
}

async function getUsers(): Promise<User[]> {
  const headersList = headers()
  const { rows } = await sql`SELECT * FROM users`
  return rows
}

export default async function UserList() {
  const users = await getUsers()
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  )
}
```

### Client Components
Quando precisamos de interatividade no cliente:

```typescript
'use client'

// app/components/ClientComponent.tsx
import { useState, useEffect } from 'react'

type ClientComponentProps = {
  initialCount: number
}

export default function Counter({ initialCount }: ClientComponentProps) {
  const [count, setCount] = useState(initialCount)
  
  useEffect(() => {
    // Este código só roda no navegador
    console.log('Component montado')
  }, [])

  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  )
}
```

## Rotas Dinâmicas e Avançadas

### Rotas Dinâmicas
```typescript
// app/blog/[slug]/page.tsx
type BlogPostParams = {
  params: {
    slug: string
  }
}

type Post = {
  title: string
  content: string
}

async function getPost(slug: string): Promise<Post> {
  const response = await fetch(`https://api.example.com/posts/${slug}`)
  if (!response.ok) throw new Error('Failed to fetch post')
  return response.json()
}

export default async function BlogPost({ params }: BlogPostParams) {
  const post = await getPost(params.slug)
  
  return (
    <article>
      <h1>{post.title}</h1>
      <div>{post.content}</div>
    </article>
  )
}
```

### Parallel Routes
```typescript
// app/@modal/(.)[id]/page.tsx
type ModalParams = {
  params: {
    id: string
  }
}

export default function Modal({ params }: ModalParams) {
  return (
    <div className="fixed inset-0 bg-black/50">
      <div className="bg-white p-6 rounded-lg">
        Conteúdo do modal {params.id}
      </div>
    </div>
  )
}
```

## Data Fetching e Server Actions

### Data Fetching
O App Router introduz novos padrões para busca de dados:

```typescript
// app/lib/data.ts
type Product = {
  id: number
  name: string
  price: number
}

export async function getProducts(): Promise<Product[]> {
  const res = await fetch('https://api.example.com/products', {
    next: {
      revalidate: 3600 // Revalidar a cada hora
    }
  })
  
  if (!res.ok) {
    throw new Error('Failed to fetch products')
  }
  
  return res.json()
}
```

### Server Actions
```typescript
// app/actions.ts
'use server'

type FormData = {
  name: string
  email: string
}

export async function createUser(data: FormData) {
  try {
    // Validação
    if (!data.email.includes('@')) {
      throw new Error('Email inválido')
    }

    // Processamento no servidor
    await sql`
      INSERT INTO users (name, email)
      VALUES (${data.name}, ${data.email})
    `

    return { success: true }
  } catch (error) {
    return { error: (error as Error).message }
  }
}

// Uso em um componente
'use client'
// app/components/UserForm.tsx
import { createUser } from '../actions'
import { useFormState } from 'react-dom'

const initialState = {
  error: null,
  success: false
}

export default function UserForm() {
  const [state, formAction] = useFormState(createUser, initialState)

  return (
    <form action={formAction}>
      <input name="name" type="text" required />
      <input name="email" type="email" required />
      <button type="submit">Criar Usuário</button>
      {state.error && <p className="text-red-500">{state.error}</p>}
      {state.success && <p className="text-green-500">Usuário criado!</p>}
    </form>
  )
}
```

## Metadata e SEO

O App Router oferece um sistema robusto para gerenciamento de metadata:

```typescript
// app/blog/[slug]/page.tsx
import { Metadata } from 'next'

type Props = {
  params: { slug: string }
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const post = await getPost(params.slug)
  
  return {
    title: post.title,
    description: post.excerpt,
    openGraph: {
      title: post.title,
      description: post.excerpt,
      images: [post.coverImage]
    }
  }
}
```

## Middleware e Autenticação

```typescript
// middleware.ts
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const token = request.cookies.get('token')
  
  if (!token && request.nextUrl.pathname.startsWith('/dashboard')) {
    return NextResponse.redirect(new URL('/login', request.url))
  }
  
  return NextResponse.next()
}

export const config = {
  matcher: '/dashboard/:path*'
}
```

## Melhores Práticas e Otimizações

1. **Colocação Estratégica de 'use client'**
   - Mantenha a diretiva 'use client' o mais baixo possível na árvore de componentes
   - Separe lógica de servidor e cliente em arquivos diferentes

2. **Otimização de Imagens**
```typescript
import Image from 'next/image'

export default function OptimizedImage() {
  return (
    <Image
      src="/large-image.jpg"
      alt="Descrição"
      width={800}
      height={600}
      placeholder="blur"
      blurDataURL="data:image/jpeg;base64,..."
    />
  )
}
```

3. **Streaming e Suspense**
```typescript
import { Suspense } from 'react'

export default function Page() {
  return (
    <Suspense fallback={<LoadingSkeleton />}>
      <SlowComponent />
    </Suspense>
  )
}
```
