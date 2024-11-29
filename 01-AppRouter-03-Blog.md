# Criando um Blog Moderno com Next.js 14

Este guia mostrará como criar um blog utilizando Next.js 14, TypeScript e Tailwind CSS, seguindo as melhores práticas de 2024.

## 1. Configuração Inicial

Primeiro, crie um novo projeto Next.js com TypeScript e Tailwind:

```bash
npx create-next-app@latest my-blog --typescript --tailwind --app --src-dir
cd my-blog
npm install lucide-react axios
```

Instale o Shadcn/UI que é uma library para instalação progressiva de components visuais:

```bash
npx shadcn@latest init
npx shadcn@latest add sidebar
```

## 2. Estrutura de Pastas

```plaintext
src/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── blog/
│       ├── [slug]/
│       │   └── page.tsx
│       └── page.tsx
├── components/
│   ├── Header.tsx
│   ├── Footer.tsx
│   └── Sidebar.tsx
└── lib/
    └── api.ts
```

## 3. Configuração do Layout Principal

Em `src/app/layout.tsx`:

```tsx
import { Inter } from 'next/font/google'
import Header from '@/components/Header'
import Footer from '@/components/Footer'
import Sidebar from '@/components/Sidebar'

const inter = Inter({ subsets: ['latin'] })

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en">
      <body className={inter.className}>
        <div className="min-h-screen bg-gray-50">
          <Header />
          <div className="flex">
            <Sidebar />
            <main className="flex-1 p-8">
              {children}
            </main>
          </div>
          <Footer />
        </div>
      </body>
    </html>
  )
}
```

## 4. Criando o Componente Sidebar

Em `src/components/Sidebar.tsx`:

```tsx
import Link from 'next/link'
import { Home, BookOpen, User, Settings } from 'lucide-react'

export default function Sidebar() {
  return (
    <aside className="w-64 min-h-screen bg-white border-r border-gray-200 px-4 py-6">
      <nav className="space-y-4">
        <Link 
          href="/" 
          className="flex items-center space-x-3 text-gray-700 hover:text-blue-600 transition-colors"
        >
          <Home size={20} />
          <span>Home</span>
        </Link>
        <Link 
          href="/blog" 
          className="flex items-center space-x-3 text-gray-700 hover:text-blue-600 transition-colors"
        >
          <BookOpen size={20} />
          <span>Blog</span>
        </Link>
        <Link 
          href="/about" 
          className="flex items-center space-x-3 text-gray-700 hover:text-blue-600 transition-colors"
        >
          <User size={20} />
          <span>Sobre</span>
        </Link>
      </nav>
    </aside>
  )
}
```

## 5. Configurando a API

Em `src/lib/api.ts`:

```typescript
import axios from 'axios'

// Usando a API do JSONPlaceholder como exemplo
const api = axios.create({
  baseURL: 'https://jsonplaceholder.typicode.com'
})

export interface Post {
  id: number
  title: string
  body: string
  userId: number
}

export async function getPosts(): Promise<Post[]> {
  const response = await api.get<Post[]>('/posts')
  return response.data
}

export async function getPost(id: number): Promise<Post> {
  const response = await api.get<Post>(`/posts/${id}`)
  return response.data
}
```

## 6. Página Principal do Blog

Em `src/app/blog/page.tsx`:

```tsx
import Image from 'next/image'
import Link from 'next/link'
import { getPosts } from '@/lib/api'

export default async function BlogPage() {
  const posts = await getPosts()

  return (
    <div className="max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-8">Blog Posts</h1>
      <div className="grid gap-6">
        {posts.map((post) => (
          <article key={post.id} className="bg-white rounded-lg shadow-md overflow-hidden">
            <div className="relative h-48 w-full">
              <Image
                src={`/api/placeholder/800/400`}
                alt="Post cover"
                fill
                className="object-cover"
              />
            </div>
            <div className="p-6">
              <h2 className="text-xl font-semibold mb-2">{post.title}</h2>
              <p className="text-gray-600 mb-4">{post.body.substring(0, 150)}...</p>
              <Link 
                href={`/blog/${post.id}`}
                className="text-blue-600 hover:text-blue-800 font-medium"
              >
                Ler mais →
              </Link>
            </div>
          </article>
        ))}
      </div>
    </div>
  )
}
```

## 7. Página Individual do Post

Em `src/app/blog/[slug]/page.tsx`:

```tsx
import Image from 'next/image'
import { getPost } from '@/lib/api'

export default async function PostPage({
  params
}: {
  params: { slug: string }
}) {
  const post = await getPost(parseInt(params.slug))

  return (
    <article className="max-w-3xl mx-auto">
      <div className="relative h-64 w-full mb-8">
        <Image
          src={`/api/placeholder/1200/600`}
          alt="Post cover"
          fill
          className="object-cover rounded-lg"
        />
      </div>
      <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
      <div className="prose lg:prose-xl">
        <p>{post.body}</p>
      </div>
    </article>
  )
}
```

## 8. Componente Header

Em `src/components/Header.tsx`:

```tsx
export default function Header() {
  return (
    <header className="bg-white border-b border-gray-200">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex justify-between items-center h-16">
          <div className="flex-shrink-0">
            <h1 className="text-2xl font-bold text-gray-900">Meu Blog</h1>
          </div>
        </div>
      </div>
    </header>
  )
}
```

## 9. Componente Footer

Em `src/components/Footer.tsx`:

```tsx
export default function Footer() {
  return (
    <footer className="bg-white border-t border-gray-200">
      <div className="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
        <p className="text-center text-gray-500">
          © {new Date().getFullYear()} Meu Blog. Todos os direitos reservados.
        </p>
      </div>
    </footer>
  )
}
```

## 10. Configurações Finais

### Atualize o `tailwind.config.js`:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      typography: {
        DEFAULT: {
          css: {
            maxWidth: '100%',
          },
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
  ],
}
```

### Instale o plugin de typography:

```bash
npm install -D @tailwindcss/typography
```
