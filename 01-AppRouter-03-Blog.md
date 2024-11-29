# Criando um Blog usando os recursos do Next e do React

## 1. Configuração Inicial

Primeiro, crie um novo projeto Next.js com TypeScript e Tailwind:

```bash
npx create-next-app@latest my-blog --typescript --tailwind --app --src-dir
cd my-blog
npm install lucide-react axios
```

Instale o Shadcn/UI que é uma library para instalação progressiva de componentes visuais (https://ui.shadcn.com/docs):

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
import { Inter } from "next/font/google"
import Header from "@/components/Header"
import Footer from "@/components/Footer"
import Sidebar from "@/components/Sidebar"
import "./globals.css"

const inter = Inter({ subsets: ["latin"] })

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="en" className="h-full">
      <body className={`${inter.className} min-h-screen flex flex-col`}>
        <Header />
        <div className="flex flex-1">
          <Sidebar />
          <main className="flex-1 p-8 bg-gray-50">{children}</main>
        </div>
        <Footer />
      </body>
    </html>
  )
}
```

Em `src/app/page.tsx`:

```tsx
import Image from "next/image"
import Link from "next/link"
import { getPosts } from "@/lib/api"
import { ArrowRight, Zap, BookOpen, MessageCircle } from "lucide-react"

export default async function HomePage() {
  const recentPosts = (await getPosts()).slice(0, 3)

  return (
    <div className="max-w-6xl mx-auto">
      {/* Hero Section */}
      <section className="py-12 md:py-20">
        <div className="text-center">
          <h1 className="text-4xl md:text-6xl font-bold text-gray-900 mb-6">
            Bem-vindo ao Meu Blog
          </h1>
          <p className="text-xl text-gray-600 mb-8 max-w-2xl mx-auto">
            Um espaço dedicado a compartilhar conhecimento, experiências e
            descobertas no mundo do desenvolvimento web.
          </p>
          <Link
            href="/blog"
            className="inline-flex items-center px-6 py-3 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 transition-colors"
          >
            Ver todos os posts
            <ArrowRight className="ml-2 h-4 w-4" />
          </Link>
        </div>
      </section>

      {/* Featured Posts Section */}
      <section className="py-12 border-t border-gray-200">
        <h2 className="text-3xl font-bold text-gray-900 mb-8">
          Posts Recentes
        </h2>
        <div className="grid md:grid-cols-3 gap-8">
          {recentPosts.map((post) => (
            <article
              key={post.id}
              className="bg-white rounded-lg shadow-md overflow-hidden"
            >
              <div className="relative">
                <Image
                  src={`https://picsum.photos/seed/${post.id}/800/600`}
                  alt="Post cover"
                  width={400}
                  height={300}
                  className="object-cover w-full h-48"
                  priority={post.id <= 3}
                />
              </div>
              <div className="p-6">
                <h3 className="font-semibold text-xl mb-2">{post.title}</h3>
                <p className="text-gray-600 mb-4 line-clamp-2">{post.body}</p>
                <Link
                  href={`/blog/${post.id}`}
                  className="text-blue-600 hover:text-blue-800 inline-flex items-center"
                >
                  Ler mais
                  <ArrowRight className="ml-1 h-4 w-4" />
                </Link>
              </div>
            </article>
          ))}
        </div>
      </section>

      {/* Features Section */}
      <section className="py-12 border-t border-gray-200">
        <div className="grid md:grid-cols-3 gap-8 text-center">
          <div className="p-6">
            <div className="bg-blue-100 w-14 h-14 rounded-full flex items-center justify-center mx-auto mb-4">
              <Zap className="w-6 h-6 text-blue-600" />
            </div>
            <h3 className="text-xl font-semibold mb-2">Conteúdo Atualizado</h3>
            <p className="text-gray-600">
              Artigos e tutoriais sempre atualizados com as últimas tecnologias.
            </p>
          </div>
          <div className="p-6">
            <div className="bg-blue-100 w-14 h-14 rounded-full flex items-center justify-center mx-auto mb-4">
              <BookOpen className="w-6 h-6 text-blue-600" />
            </div>
            <h3 className="text-xl font-semibold mb-2">Tutoriais Práticos</h3>
            <p className="text-gray-600">
              Aprenda com exemplos práticos e código fonte completo.
            </p>
          </div>
          <div className="p-6">
            <div className="bg-blue-100 w-14 h-14 rounded-full flex items-center justify-center mx-auto mb-4">
              <MessageCircle className="w-6 h-6 text-blue-600" />
            </div>
            <h3 className="text-xl font-semibold mb-2">Comunidade Ativa</h3>
            <p className="text-gray-600">
              Participe das discussões e compartilhe suas experiências.
            </p>
          </div>
        </div>
      </section>

      {/* CTA Section */}
      <section className="py-12 border-t border-gray-200">
        <div className="text-center">
          <h2 className="text-3xl font-bold mb-4">Comece a Explorar</h2>
          <p className="text-gray-600 mb-8 max-w-2xl mx-auto">
            Descubra todos os nossos artigos, tutoriais e dicas para se tornar
            um desenvolvedor melhor.
          </p>
          <Link
            href="/blog"
            className="inline-flex items-center px-6 py-3 bg-blue-600 text-white font-medium rounded-lg hover:bg-blue-700 transition-colors"
          >
            Explorar o Blog
            <ArrowRight className="ml-2 h-4 w-4" />
          </Link>
        </div>
      </section>
    </div>
  )
}
```

## 4. Criando o Componente Sidebar

Em `src/components/Sidebar.tsx`:

```tsx
import Link from "next/link"
import { Home, BookOpen, User, Settings } from "lucide-react"

export default function Sidebar() {
  return (
    <aside className="w-64 bg-white border-r border-gray-200 flex-shrink-0">
      <div className="sticky top-0 p-6">
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
      </div>
    </aside>
  )
}
```

## 5. Configurando a API

Em `src/lib/api.ts`:

```typescript
import axios from "axios"

// Usando a API do JSONPlaceholder como exemplo
const api = axios.create({
  baseURL: "https://jsonplaceholder.typicode.com",
})

export interface Post {
  id: number
  title: string
  body: string
  userId: number
}

export async function getPosts(): Promise<Post[]> {
  const response = await api.get<Post[]>("/posts")
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
import Image from "next/image"
import Link from "next/link"
import { getPosts } from "@/lib/api"

export default async function BlogPage() {
  const posts = await getPosts()

  return (
    <div className="max-w-4xl mx-auto">
      <h1 className="text-3xl font-bold mb-8">Blog Posts</h1>
      <div className="grid gap-8">
        {posts.map((post) => (
          <article
            key={post.id}
            className="bg-white rounded-lg shadow-md overflow-hidden flex flex-col md:flex-row"
          >
            <div className="relative md:w-64">
              <Image
                src={`https://picsum.photos/seed/${post.id}/800/600`}
                alt="Post cover"
                width={320}
                height={240}
                className="object-cover w-full h-48 md:h-full"
                priority={post.id <= 2}
              />
            </div>
            <div className="p-6 flex-grow">
              <h2 className="text-xl font-semibold mb-2">{post.title}</h2>
              <p className="text-gray-600 mb-4">
                {post.body.substring(0, 150)}...
              </p>
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
import Image from "next/image"
import { getPost } from "@/lib/api"

export default async function PostPage({
  params,
}: {
  params: { slug: string }
}) {
  const post = await getPost(parseInt(params.slug))

  return (
    <article className="max-w-3xl mx-auto">
      <div className="relative mb-8">
        <Image
          src={`https://picsum.photos/seed/${post.id}/800/600`}
          alt="Post cover"
          width={800}
          height={400}
          className="object-cover w-full h-[400px] rounded-lg"
          priority
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

### Atualize o `next.config.ts`:

```typescript
/** @type {import('next').NextConfig} */
const nextConfig = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "picsum.photos",
      },
    ],
  },
}

module.exports = nextConfig
```

### Instale o plugin de typography:

```bash
npm install -D @tailwindcss/typography
```
