# Loading e Error Pages

## Estrutura do Projeto
```
app/
  ├── dashboard/
  │   ├── loading.tsx
  │   ├── error.tsx
  │   └── page.tsx
  └── layout.tsx
```

## Arquivos e Explicações

### 1. Layout Raiz (app/layout.tsx)
```tsx
export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="pt-BR">
      <body>{children}</body>
    </html>
  )
}
```
**Explicação**: Layout raiz define a estrutura HTML básica. Todo conteúdo da aplicação será renderizado no `children`. É o primeiro nível da aplicação.

### 2. Loading (app/dashboard/loading.tsx)
```tsx
export default function Loading() {
  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="w-16 h-16 border-4 border-blue-500 border-t-transparent rounded-full animate-spin" />
    </div>
  )
}
```
**Explicação**: 
- Exibido automaticamente pelo Next.js durante carregamentos
- Spinner animado usando Tailwind CSS
- Intercepta carregamentos na rota /dashboard
- Substitui o conteúdo durante estados de loading

### 3. Error (app/dashboard/error.tsx)
```tsx
'use client'
import { AlertCircle } from 'lucide-react'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

export default function Error({
  error,
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex items-center justify-center min-h-screen p-4">
      <Alert variant="destructive" className="max-w-md">
        <AlertCircle className="h-4 w-4" />
        <AlertTitle>Erro</AlertTitle>
        <AlertDescription>
          {error.message}
          <button 
            onClick={reset}
            className="mt-2 w-full bg-red-500 text-white p-2 rounded hover:bg-red-600"
          >
            Tentar Novamente
          </button>
        </AlertDescription>
      </Alert>
    </div>
  )
}
```
**Explicação**:
- Marcado com 'use client' por usar interatividade
- Recebe props `error` (detalhes do erro) e `reset` (função para retry)
- Usa componentes shadcn/ui para UI consistente
- Exibe mensagem de erro dinâmica
- Botão de retry chama função `reset` para nova tentativa
- Digest é um hash único do erro para debugging

### 4. Page (app/dashboard/page.tsx)
```tsx
'use client'
import { useState, useEffect } from 'react'

// Função de fetch simulando API real
async function getData() {
  // Simula delay e erro aleatório
  await new Promise(resolve => setTimeout(resolve, 2000))
  if (Math.random() > 0.5) {
    throw new Error('Falha ao carregar dados')
  }
  return { message: 'Dados carregados com sucesso!' }
}

export default function DashboardPage() {
  const [data, setData] = useState(null)

  useEffect(() => {
    getData()
      .then(setData)
      .catch(error => {
        throw error // Propaga para Error.tsx
      })
  }, [])

  return (
    <div className="flex items-center justify-center min-h-screen">
      <div className="p-6 bg-green-100 rounded-lg">
        <h1 className="text-2xl font-bold text-green-800">
          {data?.message || 'Carregando...'}
        </h1>
      </div>
    </div>
  )
}
```
**Explicação**:
- Página principal do dashboard
- getData() simula API com:
  - 2 segundos de delay (ativa loading.tsx)
  - 50% chance de erro (ativa error.tsx)
- useEffect inicia fetch ao montar
- Erros são propagados para error.tsx
- Layout centralizado com Tailwind

## Como Testar
1. Crie os arquivos na estrutura mostrada
2. Instale dependências:
```bash
npm install lucide-react @/components/ui/alert
```
3. Acesse `/dashboard` e atualize várias vezes para ver:
   - Loading (2s)
   - Sucesso ou Erro (50/50)
   - Teste o botão de retry nos erros

O App Router gerencia automaticamente os estados da página através dos arquivos especiais.
