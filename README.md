# Next.js

## O que é Next.js?

Next.js é uma framework de React que revolucionou o desenvolvimento web full-stack. Desenvolvido pela Vercel, ele oferece uma experiência de desenvolvimento completa com foco em performance, escalabilidade e produtividade.

### 1.1 Características Principais

#### Server Components
Os Server Components são um dos pilares do Next.js moderno. Eles permitem que você execute código React diretamente no servidor, oferecendo várias vantagens:

- Redução do JavaScript enviado ao cliente
- Acesso direto a recursos do servidor (banco de dados, sistema de arquivos)
- Melhor performance inicial
- Segurança aprimorada (credenciais ficam no servidor)

```jsx
// app/page.tsx
// Este é um Server Component por padrão
async function HomePage() {
  const data = await fetch('https://api.exemplo.com/dados')
  const produtos = await data.json()

  return (
    <div>
      {produtos.map(produto => (
        <ProdutoCard key={produto.id} {...produto} />
      ))}
    </div>
  )
}
```

#### Client Components
Quando você precisa de interatividade no navegador, use Client Components:

```jsx
'use client' // Esta diretiva marca o componente como Client Component

import { useState } from 'react'

export default function Contador() {
  const [count, setCount] = useState(0)

  return (
    <button onClick={() => setCount(count + 1)}>
      Contagem: {count}
    </button>
  )
}
```
