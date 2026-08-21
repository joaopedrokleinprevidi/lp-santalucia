# Stack — Next.js App Router

Referência técnica das **Fases 10 e 12** do [SKILL.md](SKILL.md) — implementação e deploy. O que
rodar, onde cada arquivo mora, e o código que não se escreve de memória.

O projeto de referência deste repositório (`belezacompletabarreiro`) é **Vite + React**. A
factory adota **Next.js App Router** como padrão por um motivo só: `metadata`, JSON-LD, sitemap
e Open Graph precisam existir no HTML servido, e no Vite eles são trabalho manual que ninguém
mantém. Este arquivo é a ponte — o que traduz direto, o que se reescreve, e onde estão as
armadilhas.

## A ponte, arquivo por arquivo

| Referência (Vite) | Next.js App Router | Trabalho |
|---|---|---|
| `index.html` `<head>` escrito à mão | `export const metadata` em `app/layout.tsx` | Reescreve. Não existe arquivo de `<head>` |
| `<meta name="theme-color">`, `viewport` | `export const viewport` | Reescreve. Dentro de `metadata` o Next avisa e ignora |
| `<script type="application/ld+json">` no HTML | `<script dangerouslySetInnerHTML>` em `app/page.tsx` | Traduz, com escape (ver [JSON-LD](#json-ld-o-que-coloca-a-clínica-no-mapa)) |
| `public/robots.txt`, `public/sitemap.xml` | `app/robots.ts`, `app/sitemap.ts` | Reescreve e **apaga os estáticos** |
| `@fontsource-variable/*` via `@import` no CSS | `next/font/google` em `lib/fonts.ts` | Reescreve |
| `@tailwindcss/vite` | `@tailwindcss/postcss` | Troca o plugin; o `@theme` do CSS fica igual |
| `src/main.tsx` + `src/App.tsx` | `app/layout.tsx` + `app/page.tsx` | Reescreve |
| `src/components/{chapters,layout,media,ui,providers}/` | idêntico | Direto — só entra `'use client'` onde há hook |
| `src/lib/gsap.ts` | idêntico + `'use client'` + guard de `window` | Quase direto |
| `SmoothScrollProvider.tsx` | idêntico + `'use client'` | Quase direto |
| `src/data/site.ts`, `src/generated/media.ts` | idêntico | Direto. Mantenha sem `process.env` |
| `scripts/prepare-assets.mjs` | idêntico, saída para `public/` | Direto, mais a extração de frames |
| Plugin `heroPosterPreload` do `vite.config.ts` | `<link rel="preload">` no `layout.tsx` | Reescreve, e fica menor |
| `vercel.json` com `framework: "vite"`, `outputDirectory: "dist"` | `framework: "nextjs"`, **sem** `outputDirectory` | Corrija ou o deploy sobe uma pasta vazia |

---

## 1. Estrutura de pastas

A pasta do projeto é a **mesma** desde a Fase 0. As Fases 0 a 4 escreveram `design/`,
`scripts/` e `assets-source/` dentro dela; o scaffold da §2 roda no fim da Fase 5 e cria `src/`
e `app/`; as Fases 6a e 6b já gravam em `src/data/`; a Fase 10 acrescenta o resto, no mesmo lugar.

```
<slug>/                          ← a pasta em que o pipeline roda desde a Fase 0
├─ design/                       Fases 0–8: briefing.json, lacunas.md, design-system.json,
│  │                             pesquisa.md, creative-direction.json, landing-blueprint.md,
│  │                             image-prompts.md, motion-prompts.md
│  └─ renders/                   Fase 9: NN-id-secao.png e .mp4 gerados pelo dev
├─ assets-source/                material cru do cliente + entrada do pipeline — FORA do repo
├─ assets/fonts/                 .ttf lido pelo opengraph-image no build — DENTRO do repo
├─ scripts/
│  ├─ asset-inventory.mjs        Fase 1
│  ├─ check-banned-copy.mjs      Fases 4 e 6
│  └─ prepare-assets.mjs         Fase 10
├─ src/
│  ├─ app/
│  │  ├─ layout.tsx            server — metadata, viewport, <html lang="pt-BR">, fontes
│  │  ├─ page.tsx              server — JSON-LD + a ordem dos capítulos
│  │  ├─ opengraph-image.tsx   ImageResponse 1200×630 (ou opengraph-image.jpg estático)
│  │  ├─ sitemap.ts            gerado
│  │  ├─ robots.ts             gerado
│  │  ├─ manifest.ts           gerado (opcional; substitui public/site.webmanifest)
│  │  ├─ not-found.tsx
│  │  └─ globals.css           @import 'tailwindcss' + @theme + mecânica de capítulo
│  ├─ components/
│  │  ├─ chapters/             ChapterHero, ChapterJourney… — um arquivo por capítulo
│  │  ├─ layout/               PageChrome, Footer
│  │  ├─ media/                Picture, ChapterFilm, FrameSequence
│  │  ├─ providers/            SmoothScrollProvider
│  │  └─ ui/                   Action, Words, Eyebrow, ScrollCue, BrandMark, Scrim
│  ├─ hooks/                   useChapterTimeline, useChapterReveals, useFrameSequence,
│  │                           useReducedMotion
│  ├─ lib/                     gsap.ts, smoothScroll.ts, fonts.ts, schema.ts, seo.ts
│  ├─ data/                    site.ts — toda a copy e os dados de contato
│  └─ generated/               media.ts + media.json (saída do pipeline, nunca editado à mão)
├─ public/
│  ├─ media/                   derivados AVIF/WebP/MP4, com hash no nome
│  ├─ frames/<secao>.<hash>/   frame_0001.webp … (canvas frame sequence)
│  └─ favicon.svg, apple-touch-icon.png
├─ next.config.ts, postcss.config.mjs, tsconfig.json, vercel.json
```

`assets-source/` guarda os originais pesados e fica **fora** do repositório (está no
`.gitignore`); `public/media/` e `public/frames/`, que são os derivados leves com hash no nome,
**entram** no repositório. Inverter os dois é o erro que mais derruba o deploy — veja
[publicar-lp](../publicar-lp/SKILL.md) §1, checagem 2.

A divisão de `components/` é a mesma do referência de propósito: `chapters/` são as unidades
narrativas (uma por seção do blueprint), `ui/` são as peças reutilizadas entre elas, `media/`
isola tudo que carrega bytes, `providers/` é o único lugar com estado global, `layout/` é o
que não pertence a nenhum capítulo. Um componente que não se encaixa em nenhuma delas
normalmente é um capítulo disfarçado.

**Armadilha:** `public/robots.txt` e `public/sitemap.xml` conflitam com `app/robots.ts` e
`app/sitemap.ts`. O Next ou falha o build com *conflicting public file and page file*, ou o
arquivo estático vence em silêncio — e nos dois casos a versão gerada nunca vai ao ar. Ao
portar do referência, apague os dois estáticos.

---

## 2. Scaffold

Roda **no fim da Fase 5**, não na 10. A Fase 6a grava `src/data/story.ts` e a 6b grava
`src/data/site.ts`; `create-next-app` aborta numa pasta que já contém `src/`, então rodá-lo
depois da 6a obriga a apagar e reescrever os dois arquivos.

```bash
# Ponto final: cria o projeto DENTRO da pasta que já existe, sem aninhar um <slug>/<slug>/.
npx create-next-app@latest . --typescript --tailwind --app --src-dir --import-alias "@/*" --eslint
npm i gsap lenis lucide-react
npm i -D sharp ffmpeg-static ffprobe-static @types/node
```

| Pacote | Onde | Por que existe |
|---|---|---|
| `next`, `react`, `react-dom` | dep | React 19.2. O App Router é a razão de o projeto ser Next |
| `gsap` | dep | Timelines e ScrollTrigger. Nunca por CDN — uma segunda cópia registra um segundo ScrollTrigger e os dois brigam pelo mesmo scroller |
| `lenis` | dep | Smooth scroll. Pacote **`lenis`**, não `@studio-freight/lenis` — foi renomeado e o antigo está congelado |
| `lucide-react` | dep | Ícones como componente React, tree-shakeable. Evita subir um sprite SVG à mão |
| `tailwindcss` + `@tailwindcss/postcss` | dev | Tailwind 4. No Next o plugin é o de PostCSS, não o de Vite |
| `sharp` | dev | AVIF/WebP responsivo, LQIP, crop, OG image. Na Vercel o otimizador de imagem do Next já tem o seu; este é o do pipeline |
| `ffmpeg-static` | dev | Transcode desktop/mobile e extração de frames WebP. Binário via npm — **nunca** instale ffmpeg global nem escreva caminho fixo, é a origem clássica do pipeline que "funciona aqui" |
| `ffprobe-static` | dev | Duração e fps reais do master, para dimensionar o orçamento de frames |
| `typescript`, `@types/node` | dev | TS 5.9. `@types/node` é obrigatório: os scripts do pipeline usam `node:fs/promises` |

O `.` importa. `create-next-app <slug>` criaria um subdiretório novo, e `design/`, `scripts/` e
`assets-source/` — escritos pelas Fases 0 a 4 — ficariam no diretório **de cima**, fora do
projeto e fora do repositório. Os caminhos de `design/renders/` param de resolver, o pipeline de
assets não acha as mídias, e o `npm run build` passa mesmo assim. O comando avisa se a pasta tem
arquivos conflitantes (`package.json`, `app/`); `design/` e `assets-source/` não conflitam.

Registre o pipeline no `package.json` no mesmo momento em que criar o script, ou `npm run assets`
responde `Missing script: assets` na primeira vez que alguém precisar dele — inclusive o você de
daqui a seis meses, seguindo o próprio README:

```jsonc
"scripts": {
  "dev": "next dev",
  "build": "next build",
  "start": "next start",
  "lint": "next lint",
  "typecheck": "tsc --noEmit",
  "assets": "node scripts/prepare-assets.mjs"
}
```

`postcss.config.mjs`:

```js
export default { plugins: { '@tailwindcss/postcss': {} } }
```

`next.config.ts` fica curto de propósito:

```ts
import type { NextConfig } from 'next'

const config: NextConfig = {
  // O pipeline já emite AVIF+WebP em build. Isto só vale para o que ainda passar
  // por next/image (ver seção 7).
  images: { formats: ['image/avif', 'image/webp'] },
}

export default config
```

**Não use `output: 'export'`.** `sitemap.ts`, `robots.ts` e `opengraph-image` funcionam, mas
`headers()` do `next.config` deixa de existir e o otimizador de imagem some. Não há ganho
nenhum numa página hospedada na Vercel.

---

## 3. SEO — a razão de o framework ser este

Cinco arquivos. Se algum faltar, o Next não é melhor que o Vite e o projeto devia ter sido
Vite.

### `src/lib/seo.ts` — a URL canônica, num lugar só

```ts
/**
 * Resolvida em build. Fica fora de `data/site.ts` de propósito: `data/site.ts` é
 * importado por client components, e `process.env.VERCEL_PROJECT_PRODUCTION_URL`
 * não existe no bundle do cliente. Um valor no servidor e outro no cliente vira
 * mismatch de hidratação no primeiro link do rodapé.
 */
export const SITE_URL =
  process.env.NEXT_PUBLIC_SITE_URL ??
  (process.env.VERCEL_PROJECT_PRODUCTION_URL
    ? `https://${process.env.VERCEL_PROJECT_PRODUCTION_URL}`
    : 'http://localhost:3000')

/** ≤60 caracteres antes do travessão, senão o Google corta o nome do negócio. */
export const TITLE = 'Beleza Completa — Estética e Harmonização Facial no Barreiro, BH'

/** 140–160 caracteres. O metadata, o JSON-LD e o card do WhatsApp leem esta. */
export const DESCRIPTION =
  'Clínica de estética facial e corporal no Barreiro, Belo Horizonte. Protocolos ' +
  'personalizados, resultados naturais e atendimento humanizado. Agende sua avaliação.'
```

### `src/app/layout.tsx`

```tsx
import type { Metadata, Viewport } from 'next'

import { SmoothScrollProvider } from '@/components/providers/SmoothScrollProvider'
import { clinic } from '@/data/site'
import { display, sans } from '@/lib/fonts'
import { DESCRIPTION, SITE_URL, TITLE } from '@/lib/seo'
import { media } from '@/generated/media'
import './globals.css'

export const metadata: Metadata = {
  // Sem isto toda URL relativa em openGraph/alternates vira erro de build.
  metadataBase: new URL(SITE_URL),
  title: { default: TITLE, template: `%s — ${clinic.name}` },
  description: DESCRIPTION,
  applicationName: clinic.name,
  authors: [{ name: clinic.name, url: SITE_URL }],
  // Uma LP tem uma URL. O canonical existe para matar o /?fbclid=… e o /index.
  alternates: { canonical: '/' },
  openGraph: {
    type: 'website',
    locale: 'pt_BR',
    url: '/',
    siteName: clinic.name,
    title: TITLE,
    description: DESCRIPTION,
    // `images` fica ausente: `app/opengraph-image.tsx` preenche sozinho.
  },
  twitter: { card: 'summary_large_image', title: TITLE, description: DESCRIPTION },
  robots: {
    index: true,
    follow: true,
    googleBot: {
      index: true,
      follow: true,
      'max-image-preview': 'large',
      'max-snippet': -1,
      'max-video-preview': -1,
    },
  },
  icons: { icon: '/favicon.svg', apple: '/apple-touch-icon.png' },
  manifest: '/site.webmanifest',
  formatDetection: { telephone: true },
}

export const viewport: Viewport = {
  width: 'device-width',
  initialScale: 1,
  viewportFit: 'cover',
  colorScheme: 'light',
  themeColor: '#32151E',
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="pt-BR" className={`${sans.variable} ${display.variable}`}>
      <body>
        {/*
          O still do hero é o candidato a LCP. React 19 iça <link rel="preload">
          para o <head>, então isto não precisa de plugin de build como no Vite —
          e o nome com hash vem do manifest, não de um placeholder em string.
        */}
        <link
          rel="preload"
          as="image"
          type="image/avif"
          imageSrcSet={media.videos.hero.poster.avif}
          imageSizes="100vw"
          fetchPriority="high"
        />
        <a href="#conteudo" className="skip-link">
          Pular para o conteúdo
        </a>
        <SmoothScrollProvider>{children}</SmoothScrollProvider>
      </body>
    </html>
  )
}
```

**Armadilha:** `openGraph: { images: undefined }` ou `images: []` **bloqueia** a imagem gerada
por `opengraph-image.tsx`. Metadata baseada em arquivo tem prioridade sobre o export, mas a
chave presente com valor vazio é lida como "não quero imagem". Se existe
`app/opengraph-image.tsx`, simplesmente não escreva `images`.

**Armadilha:** `themeColor`, `colorScheme`, `viewport` e `width` dentro de `metadata` são
ignorados com aviso desde o Next 14. Vão em `export const viewport`.

### `src/app/opengraph-image.tsx`

O que aparece quando o link cai no WhatsApp. 1200×630 é a única medida que os três (WhatsApp,
Facebook, LinkedIn) recortam igual.

```tsx
import { readFile } from 'node:fs/promises'
import { join } from 'node:path'

import { ImageResponse } from 'next/og'

import { clinic } from '@/data/site'

export const alt = `Fachada e recepção da ${clinic.name}, no Barreiro, Belo Horizonte.`
export const size = { width: 1200, height: 630 }
export const contentType = 'image/png'

// Lidos uma vez, no escopo do módulo: nada aqui depende da requisição.
// O satori não lê woff2 — precisa ser ttf, otf ou woff, e com os glifos
// acentuados dentro. Uma fonte só-latin escreve "Estética" como "Est tica".
// assets/fonts/ é versionado — NÃO confunda com assets-source/, que está no .gitignore.
// Este arquivo é lido no build da Vercel; fora do repositório, o build quebra em ENOENT.
const displayFont = await readFile(join(process.cwd(), 'assets/fonts/Display-600.ttf'))
const photo = await readFile(join(process.cwd(), 'public/media/og-image.jpg'))

export default function Image() {
  return new ImageResponse(
    (
      <div style={{ display: 'flex', width: '100%', height: '100%', position: 'relative' }}>
        <img
          src={`data:image/jpeg;base64,${photo.toString('base64')}`}
          width={1200}
          height={630}
          style={{ position: 'absolute', inset: 0, objectFit: 'cover' }}
        />
        <div
          style={{
            display: 'flex',
            flexDirection: 'column',
            justifyContent: 'flex-end',
            width: '100%',
            padding: 72,
            background: 'linear-gradient(0deg, rgba(50,21,30,0.92) 12%, rgba(50,21,30,0) 62%)',
            color: '#FFFFFF',
            fontFamily: 'Display',
          }}
        >
          <div style={{ fontSize: 30, letterSpacing: 6, opacity: 0.82, textTransform: 'uppercase' }}>
            {clinic.district} · {clinic.city}
          </div>
          <div style={{ fontSize: 76, lineHeight: 1.05, marginTop: 16 }}>{clinic.name}</div>
          <div style={{ fontSize: 34, opacity: 0.9, marginTop: 8 }}>{clinic.tagline}</div>
        </div>
      </div>
    ),
    {
      ...size,
      fonts: [{ name: 'Display', data: displayFont, style: 'normal', weight: 600 }],
    },
  )
}
```

**Armadilha:** o satori só entende um subconjunto de CSS e exige `display: flex` explícito em
qualquer elemento com mais de um filho. O erro é literal — *Expected \<div\> to have explicit
display: flex* — e só aparece quando a rota é renderizada, não no `tsc`.

**Quando não usar `ImageResponse`:** se o card é só uma foto com o logo por cima, o
`prepare-assets.mjs` já emite exatamente isso. Coloque o arquivo em `src/app/opengraph-image.jpg`
e apague o `.tsx` — menos código, mesmo resultado. Use `ImageResponse` quando o card carrega
texto que precisa acompanhar o `data/site.ts` (nome, bairro, telefone).

### JSON-LD: o que coloca a clínica no mapa

`src/lib/schema.ts`:

```ts
import { clinic, links, openingHours, treatments } from '@/data/site'
import { DESCRIPTION, SITE_URL } from '@/lib/seo'

/**
 * Escolha o subtipo mais específico de LocalBusiness que existir. O genérico
 * perde os campos que o Google usa no painel lateral.
 *
 *   veterinária/pet → VeterinaryCare      salão/estética → BeautySalon
 *   consultório     → MedicalClinic       odontologia    → Dentist
 *   academia        → ExerciseGym         restaurante    → Restaurant
 *   nenhum encaixa  → LocalBusiness
 */
const BUSINESS_TYPE = 'VeterinaryCare'

/**
 * 24 horas se escreve 00:00–23:59 nos sete dias — é a convenção documentada.
 * `opens: '00:00', closes: '00:00'` é lido como "fechado o dia inteiro".
 */
const ALWAYS_OPEN = [
  {
    '@type': 'OpeningHoursSpecification',
    dayOfWeek: ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday', 'Sunday'],
    opens: '00:00',
    closes: '23:59',
  },
]

/** Horário normal: linhas sem `opens` são dias fechados e simplesmente não entram. */
function businessHours() {
  return openingHours
    .filter((row) => row.opens && row.closes)
    .map((row) => ({
      '@type': 'OpeningHoursSpecification',
      dayOfWeek: [...row.days],
      opens: row.opens,
      closes: row.closes,
    }))
}

export function businessSchema({ open24h = false } = {}) {
  return {
    '@context': 'https://schema.org',
    '@type': BUSINESS_TYPE,
    '@id': `${SITE_URL}/#negocio`,
    name: clinic.name,
    // A mesma description do metadata. Duas versões divergentes do que o negócio
    // é são um sinal contraditório para quem lê os dois.
    description: DESCRIPTION,
    url: SITE_URL,
    image: `${SITE_URL}/opengraph-image`,
    telephone: clinic.phoneE164,
    priceRange: '$$', // 1–4 cifrões. Nunca uma faixa em reais: o Google ignora e vira ruído.
    currenciesAccepted: 'BRL',
    address: {
      '@type': 'PostalAddress',
      streetAddress: clinic.street,
      addressLocality: clinic.city,
      addressRegion: clinic.state,
      postalCode: clinic.postalCode,
      addressCountry: 'BR',
    },
    // Latitude/longitude do pino real no Google Maps, com 6 casas.
    // Chutar coordenada coloca o negócio na rua errada — pior que omitir.
    geo: { '@type': 'GeoCoordinates', latitude: -19.977_1, longitude: -44.021_4 },
    areaServed: { '@type': 'City', name: clinic.city },
    sameAs: [clinic.instagramUrl],
    openingHoursSpecification: open24h ? ALWAYS_OPEN : businessHours(),
    makesOffer: treatments.map((name) => ({
      '@type': 'Offer',
      itemOffered: { '@type': 'Service', name },
    })),
    potentialAction: {
      '@type': 'ReserveAction',
      target: { '@type': 'EntryPoint', urlTemplate: links.whatsapp },
    },
  }
}

/**
 * `aggregateRating` só entra com número real, público e conferível — a média e a
 * contagem que estão hoje no perfil do Google. Inventar é violação de política:
 * custa ação manual e o rich snippet inteiro some. Sem review, omita o campo.
 */
export function withRating<T extends object>(schema: T, average?: number, count?: number) {
  if (!average || !count) return schema
  return {
    ...schema,
    aggregateRating: {
      '@type': 'AggregateRating',
      ratingValue: average.toFixed(1),
      reviewCount: count,
      bestRating: '5',
      worstRating: '1',
    },
  }
}

/**
 * `</script>` dentro de qualquer string fecharia a tag no meio do JSON e o resto
 * do objeto viraria HTML executável. Escapar `<` resolve os dois problemas de
 * uma vez, e o JSON continua válido — `<` volta a ser `<` no parse.
 */
export function jsonLd(schema: object): string {
  return JSON.stringify(schema).replace(/</g, '\\u003c')
}
```

`src/app/page.tsx` — server component, para o script sair no HTML da primeira resposta:

```tsx
import { ChapterHero } from '@/components/chapters/ChapterHero'
import { Footer } from '@/components/layout/Footer'
import { PageChrome } from '@/components/layout/PageChrome'
import { businessSchema, jsonLd } from '@/lib/schema'

export default function Home() {
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: jsonLd(businessSchema()) }}
      />
      <PageChrome />
      <main id="conteudo">
        <ChapterHero />
        {/* … os demais capítulos, na ordem do landing-blueprint.md */}
      </main>
      <Footer />
    </>
  )
}
```

**Armadilha:** se `page.tsx` virar `'use client'` para "facilitar", o JSON-LD passa a ser
injetado por JavaScript depois da hidratação. O Googlebot costuma renderizar, mas o WhatsApp,
o LinkedIn e o Slack não executam JS nenhum. A página perde o card e o rich snippet ao mesmo
tempo. Capítulos individuais podem ser client; `page.tsx` e `layout.tsx` não.

### `src/app/sitemap.ts` e `src/app/robots.ts`

```ts
// src/app/sitemap.ts
import type { MetadataRoute } from 'next'

import { SITE_URL } from '@/lib/seo'

export default function sitemap(): MetadataRoute.Sitemap {
  return [{ url: SITE_URL, lastModified: new Date(), changeFrequency: 'monthly', priority: 1 }]
}
```

```ts
// src/app/robots.ts
import type { MetadataRoute } from 'next'

import { SITE_URL } from '@/lib/seo'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: { userAgent: '*', allow: '/' },
    sitemap: `${SITE_URL}/sitemap.xml`,
  }
}
```

Gerados, e não estáticos, por um motivo prático: ambos carregam a URL do site. Quando o
domínio próprio entra na Fase 12, uma variável muda e os dois acompanham. Em arquivo estático,
alguém esquece o `sitemap.xml` apontando para o `.vercel.app` e o Google indexa o endereço errado.

### Por que não tem ISR nem rota dinâmica

Uma LP de negócio local tem **uma** rota, e o conteúdo dela é código: `data/site.ts`. Mudou o
horário, muda o commit, redeploy em menos de um minuto.

- **`revalidate` / ISR** transforma a rota prerenderizada numa função que revalida sob demanda.
  Custo: invocação, TTFB pior no primeiro acesso de cada região, e uma cache a mais para
  depurar. Ganho: zero — não há CMS nem banco de onde vir conteúdo novo.
- **`[slug]`** só se justifica com mais de uma página gerada de uma fonte de dados. Uma LP tem
  âncoras (`#jornada`), não rotas.
- **PPR / `cacheComponents`** resolve página com parte estática e parte dinâmica. Aqui é tudo
  estático; não há o que dividir.

Declare a intenção em `page.tsx`:

```ts
export const dynamic = 'force-static'
```

Isso não é decorativo: força o prerender e faz `cookies()`/`headers()` devolverem vazio em vez
de derrubar a página inteira para runtime em silêncio no dia em que alguém importar uma
biblioteca que os usa. Confirme no output do build — `/` precisa aparecer como `○ (Static)`.

---

## 4. Fontes

`src/lib/fonts.ts`:

```ts
import { Cormorant_Garamond, Inter } from 'next/font/google'

export const sans = Inter({
  // latin-ext não é opcional em português: sem ele, ç, ã, õ e ú caem para a
  // fonte de sistema no meio da palavra. "Harmonização" sai com duas fontes.
  subsets: ['latin', 'latin-ext'],
  display: 'swap',
  variable: '--font-inter',
})

export const display = Cormorant_Garamond({
  subsets: ['latin', 'latin-ext'],
  // Cormorant Garamond não é variable no Google Fonts: os pesos vão declarados,
  // e só os que a página realmente usa.
  weight: ['400', '600', '700'],
  display: 'swap',
  variable: '--font-cormorant',
})
```

No `globals.css`, o `@theme` do Tailwind 4 consome as variáveis — o resto do arquivo de tokens
do referência (`--text-hero`, `--ease-out-expo`, `.chapter`, o bloco de `prefers-reduced-motion`)
traduz sem uma linha de mudança:

```css
@import 'tailwindcss';

@theme {
  --font-sans: var(--font-inter), ui-sans-serif, system-ui, -apple-system, sans-serif;
  --font-display: var(--font-cormorant), ui-serif, Georgia, 'Times New Roman', serif;
  /* … o resto dos tokens, idêntico ao referência */
}
```

**Por que isso elimina o CLS.** `next/font/google` baixa os arquivos em build e os serve do
mesmo domínio: some o DNS + TLS para `fonts.gstatic.com` e some a possibilidade de a fonte
chegar depois do primeiro paint. O que de fato zera o salto é o `adjustFontFallback`, ligado
por padrão: o Next lê as métricas da face real e gera um `@font-face` de fallback local com
`size-adjust`, `ascent-override` e `descent-override` calculados para ocupar **a mesma caixa**.
Com `display: 'swap'`, a troca acontece — e nada se move, porque as duas faces medem igual.

Consequência para o motion, e ela importa: os reveals do referência medem linhas de layout ao
vivo (`lineGroups` em `lib/gsap.ts`). Com o fallback ajustado o salto é pequeno, mas as
quebras de linha ainda podem mudar. Continue chamando `ScrollTrigger.refresh()` depois de
`document.fonts.ready` (seção 5).

**Armadilha:** `preload` é `true` por padrão e exige `subsets`. Sem `subsets`, o build erra.
E não misture: importar `@fontsource-variable/inter` no CSS *além* do `next/font` baixa a
mesma fonte duas vezes e mata o benefício do `adjustFontFallback`.

---

## 5. GSAP + Lenis no App Router

A parte que mais quebra. Três erros distintos que se confundem entre si.

### Onde vai `'use client'`

| Arquivo | `'use client'`? | Motivo |
|---|---|---|
| `app/layout.tsx` | Não | Exporta `metadata`. Client component não pode; o build falha |
| `app/page.tsx` | Não | O JSON-LD precisa estar no HTML servido |
| `lib/gsap.ts` | **Sim** | Impede que um server component o importe por engano. Sem isso o erro só aparece no `next build` |
| `components/providers/SmoothScrollProvider.tsx` | **Sim** | `createContext`, `useEffect`, `new Lenis()` |
| `components/chapters/Chapter*.tsx` | **Sim**, as que animam | `useRef` + `useEffect` + ScrollTrigger |
| `components/ui/Picture.tsx`, `Eyebrow.tsx`, `Words.tsx` | Não, se só devolvem markup | Componente sem hook e sem handler não precisa ir para o bundle |
| `data/site.ts`, `generated/media.ts`, `lib/schema.ts` | Não | Dados puros, importáveis dos dois lados |

`'use client'` marca a **fronteira**, não "roda só no cliente": o módulo continua sendo
avaliado no Node durante o prerender para gerar o HTML. É daí que vem o erro do próximo bloco.

Truque que vale a pena: um server component pode passar filhos server para dentro de um client
component. `<SmoothScrollProvider>{children}</SmoothScrollProvider>` no `layout.tsx` mantém
todo capítulo estático fora do bundle, mesmo com o provider sendo client.

### `lib/gsap.ts`

```ts
'use client'

import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'

/**
 * Ponto único de registro. Todo import de GSAP no app passa por aqui.
 *
 * O guard não é paranoia: este módulo é avaliado uma vez no Node durante o
 * prerender. `ScrollTrigger.config` toca `window`, e sem o guard o `next build`
 * morre com "window is not defined" numa rota que nem tem interação.
 */
if (typeof window !== 'undefined') {
  gsap.registerPlugin(ScrollTrigger)
  // Barra de URL retrátil no mobile muda a altura do viewport no meio do scroll.
  ScrollTrigger.config({ ignoreMobileResize: true })
}

export { gsap, ScrollTrigger }

/**
 * ScrollTrigger mede a altura do documento na hora em que os triggers nascem.
 * Duas coisas mudam essa altura depois: a troca do fallback pela face real
 * (métricas diferentes reflowam a copy) e cada imagem sem width/height que
 * decodifica. Sem refresh, todo trigger abaixo da dobra dispara algumas centenas
 * de pixels fora do lugar — e o sintoma é "a animação começa cedo demais".
 */
export function refreshAfterAssets(): () => void {
  const refresh = () => ScrollTrigger.refresh()

  void document.fonts.ready.then(refresh)
  window.addEventListener('load', refresh)

  return () => window.removeEventListener('load', refresh)
}

// q(), words(), seal(), typographyReady(), lineGroups() — idênticos ao projeto
// de referência em src/lib/gsap.ts. Nada neles depende do framework.
```

**Anti-pattern:** observar `document.body` com `ResizeObserver` para chamar `refresh()`. Todo
pin do ScrollTrigger insere um pin-spacer, que muda a altura do body, que dispara o observer,
que chama refresh, que recria o spacer. Loop infinito com a aba a 100% de CPU. O ScrollTrigger
já cuida do resize de janela sozinho; `fonts.ready` + `load` cobrem o resto.

### `useReducedMotion` sem quebrar o prerender

```ts
'use client'

import { useSyncExternalStore } from 'react'

const QUERY = '(prefers-reduced-motion: reduce)'

function subscribe(onChange: () => void) {
  const mq = window.matchMedia(QUERY)
  mq.addEventListener('change', onChange)
  return () => mq.removeEventListener('change', onChange)
}

/**
 * Ler `window.matchMedia` no corpo do componente derruba o build: o corpo roda
 * no servidor durante o prerender. O terceiro argumento é o snapshot do
 * servidor — assume movimento, porque o bloco @media do CSS já protege o
 * primeiro paint, e React re-renderiza com o valor real logo após hidratar.
 */
export function useReducedMotion(): boolean {
  return useSyncExternalStore(subscribe, () => window.matchMedia(QUERY).matches, () => false)
}
```

### O provider do Lenis

```tsx
'use client'

import Lenis from 'lenis'
import { useEffect, useMemo, useRef, type ReactNode } from 'react'

import { gsap, refreshAfterAssets, ScrollTrigger } from '@/lib/gsap'
import { NATIVE_FALLBACK, SmoothScrollContext, type SmoothScrollApi } from '@/lib/smoothScroll'

/**
 * Lenis dirige a página; o ticker do GSAP dirige o Lenis.
 *
 * Um relógio só mantém posição de scroll, progresso do ScrollTrigger e as
 * timelines de capítulo no mesmo frame. Com dois rAF independentes o Lenis
 * atualiza a posição no loop A e o ScrollTrigger a lê no loop B — sempre um
 * frame atrasado. É exatamente a sensação de "o scrub está mole".
 */
export function SmoothScrollProvider({ children }: { children: ReactNode }) {
  const lenisRef = useRef<Lenis | null>(null)

  const api = useMemo<SmoothScrollApi>(
    () => ({
      scrollTo: (target, offset = 0) =>
        lenisRef.current
          ? lenisRef.current.scrollTo(target, { offset, duration: 1.4 })
          : NATIVE_FALLBACK.scrollTo(target, offset),
      jumpTo: (y) =>
        lenisRef.current
          ? lenisRef.current.scrollTo(y, { immediate: true, force: true })
          : NATIVE_FALLBACK.jumpTo(y),
      stop: () => (lenisRef.current ? lenisRef.current.stop() : NATIVE_FALLBACK.stop()),
      start: () => (lenisRef.current ? lenisRef.current.start() : NATIVE_FALLBACK.start()),
    }),
    [],
  )

  useEffect(() => {
    // matchMedia só aqui: no corpo do componente isto roda no servidor.
    if (window.matchMedia('(prefers-reduced-motion: reduce)').matches) return

    const lenis = new Lenis({
      duration: 1.05,
      easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
      wheelMultiplier: 1,
      touchMultiplier: 1.7,
      syncTouch: false,
    })
    lenisRef.current = lenis

    const onScroll = () => ScrollTrigger.update()
    lenis.on('scroll', onScroll)

    // O ticker entrega segundos; o Lenis quer milissegundos.
    const raf = (time: number) => lenis.raf(time * 1000)
    gsap.ticker.add(raf)
    // O lag smoothing do GSAP descarta o tempo de frames acima de 500 ms. Isso
    // salva animações por tempo e destrói as por scroll: o Lenis integra a
    // posição com o delta e, com tempo descartado, a página fica para trás do
    // dedo depois de qualquer travada. 0 desliga.
    gsap.ticker.lagSmoothing(0)

    const stopRefreshing = refreshAfterAssets()

    return () => {
      stopRefreshing()
      gsap.ticker.remove(raf)
      lenis.off('scroll', onScroll)
      lenis.destroy()
      lenisRef.current = null
    }
  }, [])

  return <SmoothScrollContext.Provider value={api}>{children}</SmoothScrollContext.Provider>
}
```

`lib/smoothScroll.ts` (contexto + `NATIVE_FALLBACK`) traduz do referência sem mudança, exceto
o `'use client'` no topo.

O Lenis precisa que `html` não tenha `scroll-behavior: smooth` — as duas suavizações brigam e o
resultado é um scroll elástico. Ou importe `lenis/dist/lenis.css` no provider, ou declare
`html { scroll-behavior: auto }` no `globals.css`, como o referência faz. Faça **um** dos dois.

### `gsap.context()` e o Strict Mode

Dois problemas diferentes, resolvidos em lugares diferentes:

1. **O plugin registrado duas vezes** — resolvido pelo módulo único acima. `registerPlugin` é
   idempotente, mas o import centralizado é o que impede uma segunda cópia de GSAP entrar por
   CDN e registrar um ScrollTrigger paralelo.
2. **Os triggers criados duas vezes** — este é o Strict Mode. O `next dev` roda com
   `reactStrictMode: true` por padrão, e o React 19 monta, desmonta e remonta cada componente
   uma vez. Sem cleanup, cada `ScrollTrigger.create()` roda duas vezes sobre o mesmo elemento.
   Os dois disputam o `scrub`, a progressão pula, e o bug **só existe em dev** — o que faz todo
   mundo perder uma tarde procurando no lugar errado.

O padrão, em todo hook de capítulo:

```tsx
'use client'

import { useEffect, useRef } from 'react'

import { gsap, ScrollTrigger } from '@/lib/gsap'

export function useChapterTimeline(scope: React.RefObject<HTMLElement | null>, scroll: number) {
  useEffect(() => {
    const node = scope.current
    if (!node) return

    // Tudo criado dentro do context fica registrado nele: tweens, triggers e os
    // estilos inline que o GSAP escreveu. revert() desfaz os três de uma vez.
    const ctx = gsap.context(() => {
      const tl = gsap.timeline({
        scrollTrigger: { trigger: node, start: 'top top', end: 'bottom bottom', scrub: true },
      })
      tl.to('[data-eyebrow]', { autoAlpha: 1, y: 0, duration: 0.035 }, 0.02)
      // …
    }, node)

    return () => ctx.revert()
  }, [scope, scroll])
}
```

Sem o `revert()` o sintoma clássico aparece: a segunda montagem lê como valor inicial o estado
final que a primeira já aplicou, e o elemento nasce onde deveria terminar.

### "window is not defined" no build — as quatro causas

| Sintoma no `next build` | Causa | Correção |
|---|---|---|
| Quebra num módulo com `'use client'` | Código de topo de módulo tocando `window`/`document`. `'use client'` não impede a avaliação no servidor | Guard `typeof window !== 'undefined'`, ou mova para dentro de `useEffect` |
| Quebra num arquivo sem `'use client'` | Server component importando `lib/gsap.ts` ou um capítulo animado | Ponha `'use client'` no módulo de GSAP para o erro aparecer no import errado |
| `new Lenis()` no topo do arquivo | Instância criada na importação, não na montagem | Sempre dentro de `useEffect` |
| `matchMedia` / `innerWidth` no corpo do componente | O corpo roda no prerender | `useSyncExternalStore` com snapshot de servidor, ou `useEffect` |

Aviso de `useLayoutEffect` no servidor: use `useEffect` como acima, ou adote `@gsap/react` —
o hook `useGSAP` embrulha `gsap.context()`, o cleanup e a troca isomórfica de layout effect.
É uma dependência a mais; o padrão manual acima é o que o projeto de referência usa.

---

## 6. Pipeline de assets

### Primeiro passo da Fase 10: recolher as mídias da Fase 9

O dev salvou tudo em `design/renders/`. O pipeline lê `assets-source/`. Junte os dois **antes**
de rodar qualquer coisa, ou o pipeline emite um manifest sem as imagens geradas e as seções
sobem vazias:

```bash
mkdir -p assets-source
cp design/renders/*.png design/renders/*.mp4 assets-source/ 2>/dev/null
ls assets-source/            # confira: uma linha por seção que o image-prompts.md pediu
```

Cópia, não movimentação: `design/renders/` continua sendo o registro do que o dev aprovou, e é
contra ele que o checklist do [handoff-imagens.md](handoff-imagens.md) é reconferido se uma seção
voltar a ficar errada. As fotos reais do cliente já estão em `assets-source/` desde a Fase 0.

### O script

`scripts/prepare-assets.mjs` do referência migra **sem alteração de lógica** — ele já lê
`assets-source/`, escreve em `public/media/` e emite um manifest tipado. Só mudam dois
caminhos: `MANIFEST` passa a apontar para `src/generated/media.ts` (que no Next continua sendo
`src/`), e o `media.json` deixa de ser necessário para o preload do hero — o `layout.tsx` lê o
`media.ts` direto, como TypeScript.

O que ele já faz e continua valendo: AVIF+WebP em cada largura, LQIP inline como data URI,
transcode desktop 1920×1080 / mobile 1280×720 com `hqdn3d`, hash do master no nome do arquivo,
poster extraído de um frame real, OG image 1200×630.

### O que acrescentar: frames para canvas frame sequence

```js
import { readdir, stat } from 'node:fs/promises'

/**
 * Sequências de frame. No máximo duas por página — cada uma custa de 2 a 8 MB,
 * e a terceira transforma a landing num download.
 */
const FRAME_SETS = [
  { key: 'transformacao', file: 'video-4-transformacao.mp4', fps: 15, width: 1600, quality: 78 },
]

const FRAME_FLOOR = 150 // abaixo disso o movimento estrobosa sob scrub
const FRAME_BUDGET = 8 * 1024 * 1024

async function buildFrames(set) {
  const input = path.join(SRC, set.file)
  const hash = await digest(input)

  /*
     O hash vai no nome do diretório, não do arquivo: `frame_%04d.webp` precisa
     ser previsível para o canvas montar o caminho. Sem o hash aqui, o header
     `immutable` do vercel.json congela a versão antiga na máquina de quem já
     visitou — e a correção nunca chega em quem viu o problema.
   */
  const dir = path.join(ROOT, 'public', 'frames', `${set.key}.${hash}`)
  await mkdir(dir, { recursive: true })

  await ffmpeg([
    '-i', input,
    '-vf', `fps=${set.fps},scale=${set.width}:-2:flags=lanczos`,
    '-c:v', 'libwebp',
    '-lossless', '0',
    '-quality', String(set.quality),
    '-compression_level', '6',
    path.join(dir, 'frame_%04d.webp'),
  ])

  const files = (await readdir(dir)).filter((f) => f.endsWith('.webp')).sort()
  const bytes = (await Promise.all(files.map((f) => stat(path.join(dir, f))))).reduce(
    (total, s) => total + s.size,
    0,
  )

  if (files.length < FRAME_FLOOR) {
    throw new Error(
      `${set.key}: ${files.length} frames (mínimo ${FRAME_FLOOR}). Suba o fps ou use um clipe mais longo.`,
    )
  }
  if (bytes > FRAME_BUDGET) {
    // Menos frames bons batem mais frames mastigados: corte fps antes de qualidade.
    console.warn(
      `  ! ${set.key}: ${(bytes / 1e6).toFixed(1)} MB acima do orçamento. Reduza fps, não quality.`,
    )
  }

  // O primeiro frame é o poster do fallback de reduced-motion e do carregamento.
  const poster = await responsiveImage(
    sharp(path.join(dir, files[0])),
    `${set.key}-poster.${hash}`,
    [1280, 1920],
    { quality: 66 },
  )

  console.log(`  frames ${set.key} — ${files.length} @ ${(bytes / 1e6).toFixed(1)} MB`)
  return { path: `/frames/${set.key}.${hash}`, count: files.length, bytes, poster }
}
```

No `main()`, limpe o diretório junto com o resto e some ao manifest:

```js
await rm(path.join(ROOT, 'public', 'frames'), { recursive: true, force: true })

const frames = {}
for (const set of FRAME_SETS) frames[set.key] = await buildFrames(set)
```

E no bloco que escreve o `media.ts`, acrescente o tipo e a chave:

```js
`export interface FrameSequence {\n` +
`  readonly path: string\n  readonly count: number\n` +
`  readonly bytes: number\n  readonly poster: ResponsiveImage\n}\n\n` +
// …e dentro do `satisfies`:
`  frames: Record<'transformacao', FrameSequence>\n`
```

O componente lê `media.frames.transformacao.count` — a contagem **nunca** é hardcoded. Um
número desatualizado no componente pinta frame preto no fim do scroll, e o erro só aparece
depois de alguém trocar o vídeo. O hook `useFrameSequence` está em
[video-to-website/react-port.md](../video-to-website/react-port.md); só acrescente `'use client'`.

---

## 7. `next/image` ou o `Picture` do referência

Os dois resolvem o mesmo problema por caminhos opostos: o `Picture` gera os derivados **em
build**, no seu pipeline; o `next/image` gera **em runtime**, no otimizador da Vercel.

| Situação | Componente | Motivo |
|---|---|---|
| Asset que passou pelo `prepare-assets.mjs` — hero, arte de capítulo, poster, journey | `Picture` | Os AVIF/WebP e o LQIP já existem. Passar de novo pelo `/_next/image` reprocessa o mesmo arquivo, cobra por isso e adiciona um salto de rede no LCP |
| Poster de vídeo e primeiro frame de canvas | `Picture` | Vêm do manifest com `lqip` e srcset prontos |
| Ilustração avulsa importada no código, fora do pipeline | `next/image` com import estático | O import estático dá `width`, `height` e `blurDataURL` de graça |
| Imagem remota (CMS, Google Places) | `next/image` com `remotePatterns` | É o único que otimiza o que você não controla |
| Frames de canvas | Nenhum dos dois | `new Image()` dentro do hook, caminho numerado, `public/frames/` servido cru |

A regra curta: **se o pipeline tocou o arquivo, use `Picture`.** Ele é 24 linhas de TSX
(`src/components/media/Picture.tsx` no referência), traduz sem uma mudança para o Next e não
tem `'use client'` — é markup puro.

**Armadilha:** `next/image` no hero sem `priority` não emite `<link rel="preload">`, e o LCP
regride uns 400–800 ms porque o navegador só descobre a imagem depois de montar o layout. Com
o `Picture`, o preload é o `<link>` explícito do `layout.tsx` e o problema não existe.

---

## 8. Deploy na Vercel

```bash
gh repo create <owner>/lp-<slug> --private --source . --remote origin --push
vercel link          # associa a pasta a um projeto; aceite os padrões
vercel --prod        # primeira publicação
```

Privado é o padrão da fábrica, e o pré-voo de segredos e de dado pessoal roda **antes** do
primeiro `git add`. O procedimento completo está em [publicar-lp](../publicar-lp/SKILL.md) §1.

`vercel.json`:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": "nextjs",
  "headers": [
    {
      "source": "/frames/(.*)",
      "headers": [{ "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }]
    },
    {
      "source": "/media/(.*)\\.(mp4|avif|webp)",
      "headers": [{ "key": "Cache-Control", "value": "public, max-age=31536000, immutable" }]
    },
    {
      "source": "/media/(.*)\\.(jpg|png)",
      "headers": [
        { "key": "Cache-Control", "value": "public, max-age=3600, stale-while-revalidate=604800" }
      ]
    },
    {
      "source": "/(.*)",
      "headers": [
        { "key": "X-Content-Type-Options", "value": "nosniff" },
        { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
        { "key": "X-Frame-Options", "value": "SAMEORIGIN" },
        { "key": "Permissions-Policy", "value": "camera=(), microphone=(), geolocation=()" }
      ]
    }
  ]
}
```

Três coisas que o `vercel.json` do referência tem e este **não pode** ter: `"framework": "vite"`,
`"buildCommand"` e `"outputDirectory": "dist"`. A Vercel detecta o Next sozinha; um
`outputDirectory` apontando para `dist` publica uma pasta que não existe e o site sobe em branco.

`immutable` só é seguro em caminho com hash. `/media/` e `/frames/<secao>.<hash>/` têm — é para
isso que o pipeline coloca o digest no nome. `/_next/static/` a Vercel já trata; não declare.
Nunca aplique `immutable` a `/`, a `/sitemap.xml` ou a qualquer HTML.

Alternativa: `headers()` no `next.config.ts`, que também funciona em `next start` local. Escolha
**um** dos dois lugares — a mesma `source` declarada nos dois é a origem de "mudei o cache e
nada mudou".

### Variáveis e domínio

```bash
vercel env add NEXT_PUBLIC_SITE_URL production     # cola a URL final quando existir
vercel env pull .env.local                          # traz para o dev; já está no .gitignore
vercel domains add <dominio-do-cliente>             # a CLI imprime os registros DNS
```

Enquanto não houver domínio, `SITE_URL` cai no `VERCEL_PROJECT_PRODUCTION_URL` e tudo aponta
para o `.vercel.app` — correto para publicar. No dia em que o domínio entrar: cadastre a
variável, **redeploy**, e confira canonical, sitemap e `og:url`. Sem o redeploy o Google indexa
o `.vercel.app` e o domínio novo nasce como duplicata do próprio site.

Deploy de preview a Vercel já marca com `X-Robots-Tag: noindex`. Não replique isso em produção
por engano ao copiar headers de um projeto para outro — é o erro que tira a página do índice
inteiro e leva semanas para alguém notar.

---

## 9. Checklist

```bash
npm run assets                    # pipeline: media/, frames/, generated/media.ts
npm run lint && npx tsc --noEmit
npm run build                     # '/' precisa sair como ○ (Static)
npm run start                     # produção local na porta 3000; frames não
                                  # carregam por file://, sempre HTTP
```

- [ ] `npm run build` sem warning novo, e `/` listado como `○ (Static)`
- [ ] `curl -s localhost:3000 | grep -c 'application/ld+json'` devolve `1` — o JSON-LD está no
      HTML servido, não injetado depois
- [ ] `curl -s localhost:3000/robots.txt` e `/sitemap.xml` respondem, e a URL dentro deles é a
      de produção
- [ ] `public/robots.txt` e `public/sitemap.xml` **não existem**
- [ ] `localhost:3000/opengraph-image` abre uma PNG 1200×630 com os acentos corretos
- [ ] Rich Results Test do Google aceita o JSON-LD sem erro; `aggregateRating` só presente se
      os reviews forem reais
- [ ] `<html lang="pt-BR">` no HTML servido
- [ ] Console do `next dev` sem "window is not defined" e sem aviso de `useLayoutEffect`
- [ ] Navegar até o fim e voltar com o Strict Mode ligado: nenhuma animação começa no estado
      final (sinal de `revert()` faltando)
- [ ] `document.fonts.ready` cumprido: nenhum trigger dispara fora de lugar depois do swap
- [ ] Throttle 4× de CPU: o scrub ainda acompanha o dedo
- [ ] `prefers-reduced-motion` ligado: sem canvas construído, poster visível, toda a copy legível
- [ ] Aba de rede: nenhum request para `fonts.gstatic.com`
- [ ] `curl -I <url>/media/<arquivo>.mp4` devolve `cache-control: ... immutable`
- [ ] Colar a URL de produção no WhatsApp mostra o card com a imagem
