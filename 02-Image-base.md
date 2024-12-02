# Next.js Image

## 1. Documentação do Next Image

https://nextjs.org/docs/pages/api-reference/components/image

## 2. Configuração Inicial

```javascript
// next.config.js
module.exports = {
  images: {
    domains: ['images.unsplash.com', 'exemplo.com'],
    deviceSizes: [640, 750, 828, 1080, 1200, 1920, 2048, 3840],
    imageSizes: [16, 32, 48, 64, 96, 128, 256, 384],
  },
}
```

## 3. Exemplos Práticos

### 3.1 Card de Produto E-commerce

```javascript
// components/ProductCard.js
import Image from 'next/image'

function ProductCard({ product }) {
  return (
    <div className="relative w-full max-w-sm rounded-lg overflow-hidden">
      <div className="relative h-64 w-full">
        <Image
          src={product.imageUrl}
          alt={product.name}
          fill
          sizes="(max-width: 768px) 100vw, 
                 (max-width: 1200px) 50vw,
                 33vw"
          style={{
            objectFit: 'cover',
          }}
          priority={product.featured}
        />
      </div>
      <div className="p-4">
        <h2 className="text-xl font-bold">{product.name}</h2>
        <p className="text-gray-600">{product.price}</p>
      </div>
    </div>
  )
}
```

### 3.2 Banner Hero Responsivo

```javascript
// components/HeroBanner.js
import Image from 'next/image'

function HeroBanner() {
  return (
    <div className="relative w-full h-[500px] md:h-[600px] lg:h-[700px]">
      <Image
        src="/banner-hero.jpg"
        alt="Banner Principal"
        fill
        priority
        sizes="100vw"
        style={{
          objectFit: 'cover',
          objectPosition: 'center',
        }}
      />
      <div className="absolute inset-0 bg-black bg-opacity-40">
        <div className="container mx-auto px-4 py-20">
          <h1 className="text-white text-4xl md:text-6xl font-bold">
            Título do Banner
          </h1>
        </div>
      </div>
    </div>
  )
}
```

### 3.3 Galeria de Imagens

```javascript
// components/ImageGallery.js
import Image from 'next/image'
import { useState } from 'react'

function ImageGallery({ images }) {
  const [selectedImage, setSelectedImage] = useState(0)

  return (
    <div className="max-w-4xl mx-auto">
      {/* Imagem Principal */}
      <div className="relative w-full h-[600px] mb-4">
        <Image
          src={images[selectedImage].url}
          alt={images[selectedImage].alt}
          fill
          priority
          sizes="(max-width: 768px) 100vw,
                 (max-width: 1200px) 80vw,
                 1200px"
          style={{
            objectFit: 'contain',
          }}
        />
      </div>

      {/* Miniaturas */}
      <div className="grid grid-cols-5 gap-2">
        {images.map((image, index) => (
          <button
            key={image.id}
            onClick={() => setSelectedImage(index)}
            className={`relative h-24 ${
              selectedImage === index ? 'ring-2 ring-blue-500' : ''
            }`}
          >
            <Image
              src={image.url}
              alt={`Miniatura ${index + 1}`}
              fill
              sizes="(max-width: 768px) 20vw, 10vw"
              style={{
                objectFit: 'cover',
              }}
            />
          </button>
        ))}
      </div>
    </div>
  )
}
```

### 3.4 Avatar com Fallback

```javascript
// components/UserAvatar.js
import Image from 'next/image'
import { useState } from 'react'

function UserAvatar({ user }) {
  const [imageError, setImageError] = useState(false)

  if (imageError || !user.avatarUrl) {
    return (
      <div className="w-12 h-12 rounded-full bg-gray-200 flex items-center justify-center">
        <span className="text-xl font-bold text-gray-600">
          {user.name.charAt(0)}
        </span>
      </div>
    )
  }

  return (
    <div className="relative w-12 h-12">
      <Image
        src={user.avatarUrl}
        alt={`Avatar de ${user.name}`}
        fill
        sizes="48px"
        className="rounded-full"
        onError={() => setImageError(true)}
        style={{
          objectFit: 'cover',
        }}
      />
    </div>
  )
}
```

## 4. Cenários Avançados

### 4.1 Imagem com Loading Blur Personalizado

```javascript
// components/BlurImage.js
import Image from 'next/image'

function BlurImage({ src, alt }) {
  return (
    <div className="relative aspect-video">
      <Image
        src={src}
        alt={alt}
        fill
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,/9j/4AAQSkZJRg..."  // Base64 da imagem em baixa resolução
        sizes="(max-width: 768px) 100vw,
               (max-width: 1200px) 50vw,
               33vw"
        style={{
          objectFit: 'cover',
        }}
      />
    </div>
  )
}
```

### 4.2 Lazy Loading Customizado

```javascript
// components/LazyImage.js
import Image from 'next/image'
import { useInView } from 'react-intersection-observer'

function LazyImage({ src, alt }) {
  const { ref, inView } = useInView({
    triggerOnce: true,
    threshold: 0.1,
  })

  return (
    <div ref={ref} className="relative h-80">
      {inView && (
        <Image
          src={src}
          alt={alt}
          fill
          sizes="(max-width: 768px) 100vw,
                 (max-width: 1200px) 50vw,
                 33vw"
          style={{
            objectFit: 'cover',
          }}
        />
      )}
    </div>
  )
}
```

## 5. Dicas de Otimização

### 5.1 Imagens Estáticas vs Dinâmicas
```javascript
// Imagem Estática (importação direta)
import bannerImage from '../public/banner.jpg'

function Banner() {
  return (
    <Image
      src={bannerImage}
      alt="Banner"
      placeholder="blur" // Funciona automaticamente com import
    />
  )
}

// Imagem Dinâmica (URL)
function DynamicImage({ url }) {
  return (
    <Image
      src={url}
      alt="Imagem dinâmica"
      width={800}
      height={600}
      unoptimized={process.env.NODE_ENV === 'development'} // Otimiza apenas em produção
    />
  )
}
```

### 5.2 Otimização de Performance

```javascript
// components/OptimizedImage.js
import Image from 'next/image'

function OptimizedImage({ src, alt, priority = false }) {
  const isAboveTheFold = priority // Imagens acima da dobra devem ter priority=true

  return (
    <Image
      src={src}
      alt={alt}
      width={1200}
      height={800}
      priority={isAboveTheFold}
      loading={isAboveTheFold ? 'eager' : 'lazy'}
      quality={85} // Bom equilíbrio entre qualidade e tamanho
      sizes="(max-width: 768px) 100vw,
             (max-width: 1200px) 50vw,
             33vw"
    />
  )
}
```

## 6. Tratamento de Erros

```javascript
// components/ImageWithFallback.js
import Image from 'next/image'
import { useState } from 'react'

function ImageWithFallback({ src, alt, ...props }) {
  const [error, setError] = useState(false)
  const [isLoading, setIsLoading] = useState(true)

  return (
    <div className="relative">
      {isLoading && (
        <div className="absolute inset-0 bg-gray-100 animate-pulse" />
      )}
      
      {!error ? (
        <Image
          src={src}
          alt={alt}
          {...props}
          onError={() => setError(true)}
          onLoadingComplete={() => setIsLoading(false)}
        />
      ) : (
        <div className="bg-gray-200 w-full h-full flex items-center justify-center">
          <span>Erro ao carregar imagem</span>
        </div>
      )}
    </div>
  )
}
```

## 7. Casos de Uso Específicos

### 7.1 Background Image Component

```javascript
// components/BackgroundImage.js
import Image from 'next/image'

function BackgroundImage({ src, children }) {
  return (
    <div className="relative min-h-screen">
      <Image
        src={src}
        alt=""
        fill
        priority
        sizes="100vw"
        style={{
          objectFit: 'cover',
          zIndex: -1,
        }}
      />
      <div className="relative z-10">
        {children}
      </div>
    </div>
  )
}
```

### 7.2 Imagem com Zoom

```javascript
// components/ZoomImage.js
import Image from 'next/image'
import { useState } from 'react'

function ZoomImage({ src, alt }) {
  const [isZoomed, setIsZoomed] = useState(false)

  return (
    <div 
      className="relative overflow-hidden cursor-zoom-in"
      onClick={() => setIsZoomed(!isZoomed)}
    >
      <Image
        src={src}
        alt={alt}
        fill
        sizes="(max-width: 768px) 100vw, 50vw"
        style={{
          objectFit: 'contain',
          transform: isZoomed ? 'scale(1.5)' : 'scale(1)',
          transition: 'transform 0.3s ease',
        }}
      />
    </div>
  )
}
```

## 8. Considerações de Acessibilidade

```javascript
// components/AccessibleImage.js
import Image from 'next/image'

function AccessibleImage({ src, alt, caption }) {
  return (
    <figure>
      <div className="relative aspect-video">
        <Image
          src={src}
          alt={alt}
          fill
          sizes="(max-width: 768px) 100vw,
                 (max-width: 1200px) 50vw,
                 33vw"
          style={{
            objectFit: 'cover',
          }}
          aria-describedby={caption ? 'image-caption' : undefined}
        />
      </div>
      {caption && (
        <figcaption 
          id="image-caption"
          className="mt-2 text-sm text-gray-600"
        >
          {caption}
        </figcaption>
      )}
    </figure>
  )
}
```

## 9. Debug e Troubleshooting

```javascript
// utils/imageDebug.js
export function debugImageLoading(src) {
  if (process.env.NODE_ENV === 'development') {
    console.log(`🖼️ Carregando imagem: ${src}`)
    console.time(`Imagem ${src}`)
    
    return {
      onLoadingComplete: () => {
        console.timeEnd(`Imagem ${src}`)
        console.log(`✅ Imagem carregada: ${src}`)
      },
      onError: (e) => {
        console.error(`❌ Erro ao carregar imagem: ${src}`, e)
      }
    }
  }
  
  return {}
}

// Uso
function DebugImage({ src, alt }) {
  return (
    <Image
      src={src}
      alt={alt}
      width={800}
      height={600}
      {...debugImageLoading(src)}
    />
  )
}
```
