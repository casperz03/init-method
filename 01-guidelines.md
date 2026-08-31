# 01 · Guidelines

Dokument referencyjny do wklejenia jako kontekst projektu (CLAUDE.md / notatka w narzędziu AI). Obowiązuje przy każdym projekcie klienckim w ramach INIT Method. Uzupełnij [`04-context.md`](04-context.md) danymi konkretnego projektu - ten plik zawiera zasady stałe, niezależne od klienta.

**Uwaga o aktualności:** sekcje *SEO pod AI (GEO)*, *Prawo* i *Cloudflare* opisują szybko zmieniający się stan - zweryfikuj je przy większej rewizji dokumentu.

## Stack

| Warstwa | Narzędzie | Rola |
|---|---|---|
| Framework | **Astro**, `output: 'static'` | SSG - HTML w całości w buildzie, zero JS domyślnie |
| Style | **Vanilla CSS** | CSS Nesting, custom properties, bez frameworków (Bootstrap/Tailwind) |
| Animacje | **GSAP** (+ ScrollTrigger) | Jedyna sankcjonowana biblioteka JS poza vanilla - do scroll-triggered i złożonych animacji |
| CMS | **Decap CMS** (backend `github` + OAuth-proxy na Cloudflare Worker) | Edycja treści przez klienta, commit do repo, bez Netlify |
| Treść | **Astro Content Collections** + Zod | Walidacja frontmatteru, typowany dostęp do treści z Decap |
| Hosting | **Cloudflare Pages** | Build z gita, edge CDN, darmowy tier bez limitu requestów |
| DNS / SSL / WAF | **Cloudflare** | Domena, certyfikaty, ochrona przed botami/DDoS |
| Audyt SEO | **DataForSEO** | Narzędzie procesowe (workflow), nie zależność runtime |

Domyślny wybór: **Astro static + Cloudflare Pages** dla całego katalogu projektów (strony wizytówkowe/ofertowe/blogi). SSR (adapter `@astrojs/cloudflare`, Workers) tylko gdy projekt tego realnie wymaga - patrz sekcja 13.

## 1. Zasady ogólne

- **DRY** - trzeci powtórzony blok = wydziel komponent `.astro`/zmienną CSS.
- **KISS** - brak abstrakcji i bibliotek tam, gdzie wystarczy vanilla JS/CSS lub natywne API przeglądarki.
- **Mobile-first** - style od najmniejszego viewportu w górę, nowoczesna składnia range: `@media (width >= 768px)` zamiast `@media (min-width: 768px)`.
- **Zgodność ze standardami** - [Baseline](https://web.dev/baseline) jako źródło prawdy o wsparciu w przeglądarkach ("czy mogę tego użyć" ma jedną odpowiedź, nie subiektywną ocenę), Core Web Vitals, Google Search Central.
- **Lintery od pierwszego commitu** - `npm install -D eslint-plugin-astro eslint-config-prettier prettier-plugin-astro`.
- **Komentarze** - tylko tam, gdzie kod sam nie tłumaczy "dlaczego" (ukryte ograniczenie, obejście buga, nieoczywisty powód decyzji). Zawsze po angielsku, nawet w projekcie PL-językowym - spójność z resztą kodu i narzędziami.

**Praca z AI/agentem w tym repo:**
- Przed implementacją nieznanego API/integracji - sprawdź aktualną dokumentację (MCP z docs, WebSearch), nie polegaj na pamięci modelu. Dokumentacja bibliotek zmienia się szybciej niż cykl treningowy.
- Po każdej iteracji: commituj lokalnie, ale **zawsze pytaj przed PR/push** - i dodaj krótkie podsumowanie, co zrobiłeś w tej iteracji.

## 2. Struktura projektu

```
/
├── public/
│   ├── admin/              # Decap CMS (config.yml, index.html)
│   ├── _redirects          # przekierowania Cloudflare Pages
│   ├── _headers            # nagłówki bezpieczeństwa + cache
│   ├── robots.txt
│   └── llms.txt
├── src/
│   ├── assets/              # obrazy przetwarzane przez Astro (astro:assets)
│   ├── components/
│   ├── content/             # Content Collections - tu commituje Decap
│   │   └── config.ts        # schematy Zod
│   ├── layouts/
│   ├── pages/
│   ├── scripts/             # vanilla JS / inicjalizacja GSAP
│   └── styles/               # global.css (custom properties, reset)
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

- `src/pages/` wymagany - brak = brak routingu.
- Własny kod (CSS/JS) zawsze w `src/`, nigdy w `public/` - tylko wtedy Astro go zbunduje i zoptymalizuje.
- `public/` wyłącznie dla plików trafiających 1:1 do builda.
- Skopiowane pliki INIT Method (`01-guidelines.md`, `04-context.md` i pochodne jak `CLAUDE.md`/`AGENTS.md`) trzymaj w `context/` w rootcie repo, nie luzem - jedno miejsce, w którym widać cały kontekst projektu naraz.

**Konfiguracja bazowa (`astro.config.mjs`):**

```js
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://domena.pl',    // WYMAGANE - bez tego sitemap i canonicale wskażą na localhost
  trailingSlash: 'always',       // jedna, spójna forma URL - trzymaj się konsekwentnie
  integrations: [sitemap()],
  // adapter NIE jest potrzebny dla output: 'static' - Cloudflare Pages serwuje build bezpośrednio
});
```

⚠️ `trailingSlash: 'always'` a sitemapa: `@astrojs/sitemap` normalizuje `<loc>` dla strony głównej dopiero po serializacji (wewnątrz `write-sitemap.js`), czego nie widać wprost w dokumentacji integracji. Przy niespójnym wyborze `trailingSlash` self-canonical (§9, liczony z `Astro.url`) i wpis w sitemapie mogą się cicho rozjechać, bez błędu buildu - sprawdź to ręcznie po pierwszym buildzie, nie zakładaj zgodności.

## 3. Konwencje nazewnictwa

| Element | Konwencja | Przykład |
|---|---|---|
| Zmienne, funkcje JS | camelCase | `getUserData`, `isMenuOpen` |
| Klasy CSS, ID | kebab-case | `.hero-section`, `#main-nav` |
| Pliki komponentów Astro | PascalCase | `Header.astro`, `ContactForm.astro` |
| Selektory pod JS | prefiks `js-` | `.js-toggle-menu` |

- Wszystkie nazwy (zmienne, klasy, ID, funkcje) - **zawsze po angielsku**. Wyjątek: anchory URL istotne SEO/UX dla PL-użytkownika (`#cennik`, `#kontakt`).
- **JS nigdy nie odwołuje się do `id`** - zawsze dedykowana klasa `js-*`, oddzielona od klas stylistycznych. Zmiana stylu nie może przypadkiem zerwać skryptu i odwrotnie.

```html
<!-- źle -->
<button id="menu-toggle" class="btn btn-icon">...</button>
<script>document.getElementById('menu-toggle')...</script>

<!-- dobrze -->
<button class="btn btn-icon js-menu-toggle">...</button>
<script>document.querySelector('.js-menu-toggle')...</script>
```

## 4. HTML - semantyka

| Tag | Zastosowanie |
|---|---|
| `header` | Nagłówek strony lub sekcji (nie tylko góra strony - też np. nagłówek `article`) |
| `nav` | Główna nawigacja i inne bloki linków nawigacyjnych (breadcrumbs, paginacja) |
| `main` | Jedna na stronę - główna, unikalna treść (poza tym, co się powtarza w header/footer/nav) |
| `section` | Tematyczna grupa treści z własnym nagłówkiem - nie zamiennik `div` |
| `article` | Samodzielna treść, sensowna poza kontekstem strony (post bloga, karta usługi, opinia) |
| `aside` | Treść poboczna względem głównej (sidebar, powiązane linki, cytat) |
| `footer` | Stopka strony lub sekcji (autor, data, linki powiązane) |
| `figure` + `figcaption` | Obraz/diagram/kod z podpisem, referencyjny z treści głównej |
| `time` | Data/czas czytelna maszynowo (`datetime=""`) - istotne pod schema i crawlery |
| `dialog` | Natywne modale - wbudowany focus trap i `::backdrop`, bez własnej implementacji JS |

- Jeden `h1` na stronę, logiczna kolejność `h2 → h3` bez przeskoków.
- `alt` na każdym obrazie treściowym (`alt=""` dla dekoracyjnych).
- Formularze: `<label for="">` powiązane z polem, `required`, `autocomplete`, właściwy `type`.

## 5. CSS - vanilla, nowoczesny

- Bez frameworków (Bootstrap/Tailwind), chyba że projekt tego jawnie wymaga.
- **CSS Nesting** natywny (bez preprocesora):

```css
.card {
  padding: 1rem;

  & .card-title { font-size: 1.25rem; }
  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 10%); }
}
```

- Jednostki świadomie: `rem` (typografia/spacing), `%`/`fr` (siatki), `svh/dvh` (pełnoekranowe sekcje - `dvh` zamiast `vh` na mobile, unika skoku przez pasek adresu), `clamp()` (płynna typografia bez sterty media queries), `min()`/`max()`.
- Layout: flexbox/grid zamiast floatów/absolute tam, gdzie możliwe.

**Warstwy (`@layer`)** - kolejność deklaracji ustala priorytet bez walki ze specyficznością, jedna deklaracja na cały projekt w `global.css`:

```css
@layer tokens, reset, components;
```

**Kolory: OKLCH, nie HEX** - percepcyjnie jednolita przestrzeń (te same kroki liczbowe = ten sam odbierany kontrast), łatwiejsze świadome tworzenie wariantów (jaśniej/ciemniej bez zgadywania). Dwuwarstwowo: prymitywy (surowa paleta) → semantyka (jedyne zmienne używane poza warstwą `tokens`):

```css
@layer tokens {
  :root {
    --color-gray-50: oklch(98% 0.002 271);
    --color-gray-900: oklch(19.5% 0.00002 271);
    --color-blue-500: oklch(55.8% 0.189 259.9);

    --color-bg-canvas: var(--color-gray-50);
    --color-text-primary: var(--color-gray-900);
    --color-accent: var(--color-blue-500);
  }

  @media (prefers-color-scheme: dark) {
    :root {
      --color-bg-canvas: var(--color-gray-900);
      --color-text-primary: var(--color-gray-50);
    }
  }
}
```

Konwencja: `--kategoria-wariant`, kebab-case, po angielsku. Pełny zestaw tokenów (spacing, typografia, cienie, z-index, czas trwania animacji) i pełny reset elementów - patrz [§18, Gotowe szablony](#18-gotowe-szablony).

## 6. JavaScript - vanilla + GSAP

- `const`/`let` wyłącznie - zero `var`.
- `querySelector`/`querySelectorAll`, nie `getElementById`.
- Nowoczesna składnia: arrow functions, template literals, destructuring, `async/await`, moduły ES, `?.`, `??`.
- Jedyne źródło prawdy dla API/składni: **MDN** - nie kopiuj z blogów/SO bez weryfikacji.
- Interakcje zawsze przez `.js-*` (sekcja 3).

**GSAP - standard stacku, nie wyjątek:**

- Do prostych przejść/hover/fade - zawsze czysty CSS (`transition`, `@keyframes`), taniej niż JS.
- GSAP + `ScrollTrigger` - do scroll-triggered animacji, timeline'ów, złożonych sekwencji. **Wymaga jawnej rejestracji pluginu** przed użyciem, inaczej nie działa (bez błędu w konsoli - po prostu cicho nic się nie animuje):

```js
import { gsap } from 'gsap';
import { ScrollTrigger } from 'gsap/ScrollTrigger';
gsap.registerPlugin(ScrollTrigger);
```

- Ładuj tylko na stronach, które faktycznie animacji używają - nie globalnie do każdej podstrony (waga pliku wpływa na CWV).
- **Musi respektować `prefers-reduced-motion`** - GSAP nie robi tego automatycznie:

```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (!prefersReducedMotion) {
  // dopiero tutaj inicjalizuj animacje GSAP
}
```

**Gotcha: Astro View Transitions + GSAP.** Jeśli projekt używa `<ClientRouter />` (View Transitions), `DOMContentLoaded` odpala się tylko raz - przy kolejnych nawigacjach nie. Inicjalizuj GSAP/ScrollTrigger na evencie `astro:page-load`, nie `DOMContentLoaded`, inaczej animacje przestają działać po pierwszej nawigacji SPA-like.

**Ładowanie skryptów:** domyślnie `defer` (nie blokuje parsowania, dostęp do pełnego DOM). `async` tylko dla niezależnych od DOM/kolejności (analytics). Nigdy `<script>` bez `async`/`defer` w `<head>`. Komponenty `.astro` ze `<script>` są domyślnie bundlowane jako moduły (`type="module"`, efektywnie `defer`).

## 7. TypeScript

`tsconfig.json`:

```json
{
  "extends": "astro/tsconfigs/strictest",
  "compilerOptions": {
    "noUncheckedIndexedAccess": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  }
}
```

- `interface` - kształty obiektów, propsy komponentów. `type` - unie, przecięcia, aliasy.
- Unikaj `enum` (nieintuicyjne zachowanie) - używaj union literal types (`type Theme = 'light' | 'dark'`) lub `as const` + `typeof`.
- `unknown` zamiast `any` dla danych z zewnątrz przed walidacją. `satisfies` zamiast `as`. `import type` dla importów wyłącznie-typów.
- Unikaj non-null assertion (`!`) - sprawdzaj jawnie (`if`, `?.`).
- Dane z zewnątrz (formularz, CMS, API) - Zod + `z.infer<typeof schema>`, typ i walidacja nigdy się nie rozjeżdżają.

## 8. Wydajność / Core Web Vitals

- Obrazy poniżej foldu: `loading="lazy"`. Obraz LCP (hero): **bez** lazy, `loading="eager"` + `fetchpriority="high"`.
- `width`/`height` (lub `aspect-ratio`) na każdym obrazie - zapobiega CLS.
- `astro:assets` (`<Image />`) zamiast surowych `<img>` z `public/`, gdzie to możliwe - auto AVIF/WebP, `srcset`.

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---
<!-- LCP - priorytetowo -->
<Image src={heroImage} alt="Opis" widths={[400, 800, 1200]} sizes="(max-width: 768px) 100vw, 1200px" loading="eager" fetchpriority="high" format="avif" />
<!-- poniżej foldu -->
<Image src={heroImage} alt="Opis" loading="lazy" format="webp" />
```

- Fonty: self-hosted (bez CDN - szybciej, bez dodatkowego DNS, bez transferu IP do zewnętrznego serwera → RODO), `.woff2`, variable font, `font-display: swap`, `preload` tylko dla fontu krytycznego.
- Cel: LCP < 2.5s, INP < 200ms, CLS < 0.1 (Lighthouse / PageSpeed Insights).

## 9. SEO

- Unikalny `<title>` (≤ 60 zn.) i `<meta name="description">` (≤ 155-160 zn.) na każdej podstronie.
- Jeden `h1` zgodny z intencją wyszukiwania danej podstrony.
- Semantyczny URL (kebab-case, bez zbędnych parametrów).
- JSON-LD tam, gdzie dotyczy: `LocalBusiness`, `Article`, `FAQPage`, `BreadcrumbList`.
- `robots.txt` + `sitemap.xml` (`@astrojs/sitemap`).
- **Self-canonical domyślnie na każdej podstronie** - generowany automatycznie z `Astro.url` + `site`, nie ręcznie. Wyjątek: świadome duplikaty (filtr/sortowanie, wersja do druku, UTM) → canonical wskazuje wersję główną.

**Komponent `SEOHead.astro`** (jeden punkt prawdy, zamiast duplikować meta tagi na każdej stronie):

```astro
---
interface Props {
  title: string;
  description: string;
  image?: string;
  noindex?: boolean;
  canonicalOverride?: string;
}
const { title, description, image = '/og-default.jpg', noindex = false, canonicalOverride } = Astro.props;
const siteName = 'Nazwa Firmy';
const canonical = canonicalOverride ?? new URL(Astro.url.pathname, Astro.site).href;
const ogImage = new URL(image, Astro.site).href;
---
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>{title}</title>
<meta name="description" content={description} />
{!noindex && <link rel="canonical" href={canonical} />}
{noindex && <meta name="robots" content="noindex, follow" />}
<meta property="og:type" content="website" />
<meta property="og:site_name" content={siteName} />
<meta property="og:locale" content="pl_PL" />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:url" content={canonical} />
<meta property="og:image" content={ogImage} />
<meta property="og:image:width" content="1200" />
<meta property="og:image:height" content="630" />
<meta property="og:image:alt" content={title} />
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:title" content={title} />
<meta name="twitter:description" content={description} />
<meta name="twitter:image" content={ogImage} />
<meta name="theme-color" content="#1a56db" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="icon" href="/favicon.ico" sizes="32x32" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
```

⚠️ **`canonical` musi być warunkowy** (`{!noindex && ...}`) - bezwarunkowy `<link rel="canonical">` obok `noindex` (np. na stronie 404) daje dwa sprzeczne sygnały naraz: "to jest kanoniczny adres" i "nie indeksuj tego adresu". Łatwy do przeoczenia, bo strona wygląda i działa normalnie - to defekt, który propaguje się cicho na każdy projekt zbudowany z tego szablonu.

**`<link rel="manifest">` - opcjonalny, nie domyślny.** Dla statycznej wizytówki bez service workera manifest nie daje nic poza ikoną na ekranie głównym telefonu. Dodawaj go (razem z plikiem z §18) tylko gdy realnie ma to znaczenie dla projektu (np. planowane PWA) - w przeciwnym razie pomiń i zaloguj w `04-context.md`, że to świadomy default tego projektu, nie odstępstwo od metody.

## 10. Lokalne SEO (projekty dla lokalnych firm)

- **NAP spójny** (Name, Address, Phone) - identyczne dane firmy na stronie, w stopce, w Google Business Profile i innych wizytówkach (Panorama Firm, Facebook). Niespójność to jeden z najczęstszych błędów obniżających lokalny ranking.
- Nazwa miasta/regionu naturalnie w `title`/`h1`/treści kluczowych podstron - bez keyword stuffingu.
- Google Business Profile założony/zweryfikowany, powiązany z tą samą domeną.
- Mapa Google osadzona na stronie kontaktowej (embed z prawidłowym adresem).
- Pełny adres, telefon, godziny - w tekście, nie tylko na obrazku.
- Sekcja opinii/referencji klientów widoczna na stronie, jeśli dotyczy.

**Schema `LocalBusiness`:**

```astro
<script type="application/ld+json" set:html={JSON.stringify({
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "Nazwa Firmy",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "ul. Przykładowa 1",
    "addressLocality": "Miasto",
    "postalCode": "00-000",
    "addressCountry": "PL"
  },
  "geo": { "@type": "GeoCoordinates", "latitude": 52.2297, "longitude": 21.0122 },
  "telephone": "+48123456789",
  "openingHours": "Mo-Fr 09:00-17:00"
})} />
```

## 11. Accessibility (a11y)

- Kontrast tekstu min. WCAG AA (4.5:1).
- Pełna nawigacja klawiaturą, widoczny `:focus-visible`.
- ARIA tylko tam, gdzie semantyka HTML nie wystarcza.
- **Skip link** na początku `<body>`:

```html
<a href="#main-content" class="skip-link">Przejdź do treści</a>
...
<main id="main-content">...</main>
```

```css
.skip-link {
  position: absolute;
  inset-inline-start: var(--space-md);
  inset-block-start: var(--space-md);
  z-index: 999;
  padding: var(--space-sm) var(--space-md);
  background: var(--color-bg);
  transform: translateY(-200%);
  transition: transform var(--transition-base);

  &:focus-visible { transform: translateY(0); }
}
```

- **`prefers-reduced-motion`** - globalny fallback w `global.css`:

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

GSAP tego nie respektuje automatycznie - sprawdzaj jawnie w JS (sekcja 6).

## 12. Bezpieczeństwo

**Nagłówki** - plik `public/_headers` (Cloudflare Pages czyta ten sam format co Netlify):

```
/*
  X-Frame-Options: SAMEORIGIN
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), microphone=(), camera=()
  Strict-Transport-Security: max-age=63072000; includeSubDomains
  Content-Security-Policy: default-src 'self'; img-src 'self' data: https:; script-src 'self'; style-src 'self' 'unsafe-inline'
```

⚠️ **`script-src 'self'` blokuje inline-skrypty, które Astro potrafi wygenerować samo.** Małe, bezimportowe chunki JS Vite domyślnie inline'uje w HTML (poniżej `assetsInlineLimit`) - taki `<script>` bez `nonce`/hasha łamie CSP po cichu (menu mobilne, akordeon czy dowolna interakcja bez zewnętrznego importu przestaje działać, zero błędu poza CSP violation w konsoli). Wyłącz inlining plików `.js` w `astro.config.mjs`:

```js
export default defineConfig({
  vite: {
    build: {
      assetsInlineLimit: (filePath) => (filePath.endsWith('.js') ? false : undefined),
    },
  },
});
```

- **Cloudflare dashboard, SSL/TLS → Overview:** tryb **Full (strict)**. **Edge Certificates:** Always Use HTTPS = on. HSTS włączaj dopiero po potwierdzeniu, że wszystkie subdomeny wspierają HTTPS (preload jest trudny do cofnięcia).
- **WAF managed rules + Bot Fight Mode** - włącz w dashboardzie, darmowe na każdym planie, minimalny wysiłek za realną ochronę.
- SRI (`integrity` + `crossorigin`) przy skryptach z zewnętrznego CDN.
- **Formularze:** honeypot jako pierwsza linia obrony przed spamem, walidacja client + service-side (Web3Forms/Formspree).
- Żaden sekret w kodzie/commitach - zmienne środowiskowe w Cloudflare Pages (dashboard → Settings → Environment variables). `PUBLIC_*` trafia do bundla JS wysyłanego do przeglądarki - traktuj jako jawne.
- `npm audit` cyklicznie, zwłaszcza przed przekazaniem projektu.

## 13. Renderowanie: SSG domyślnie

**Treść, która ma być zaindeksowana, nigdy nie może zależeć wyłącznie od JS po stronie klienta (CSR).**

- **SSG** (`output: 'static'`, domyślne) - strona wizytówkowa/ofertowa/blog → zawsze. Zero JS wymagane do zobaczenia treści, najlepsze pod SEO i CWV.
- **SSR** (adapter `@astrojs/cloudflare`, Cloudflare Workers) - tylko gdy treść realnie zależy od requestu/sesji (panel klienta, dane real-time). Zmienia strukturę deploy (Workers zamiast czystego Pages) - decyzja świadoma, nie domyślna.
- Astro Islands: reszta strony to statyczny HTML, JS tylko lokalnie na interaktywnych fragmentach (`client:visible`, `client:idle`).

## 14. SEO pod AI/LLM (GEO)

- Treść w HTML, nie tylko w JS - wiele crawlerów AI nie wykonuje JavaScriptu (patrz sekcja 13).
- Jasna hierarchia nagłówków - modele segmentują treść po `h1→h2→h3`.
- Jedna jednoznaczna odpowiedź na pytanie w jednym akapicie - zwiększa szansę cytowania w AI Overviews/ChatGPT.
- JSON-LD (`FAQPage`, `HowTo`, `LocalBusiness`) - bezpośredni sygnał dla modeli, czym jest dana treść:

```astro
<script type="application/ld+json" set:html={JSON.stringify({
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [{
    "@type": "Question",
    "name": "Pytanie klienta w naturalnym języku?",
    "acceptedAnswer": { "@type": "Answer", "text": "Zwięzła, konkretna odpowiedź w jednym akapicie." }
  }]
})} />
```

- `public/llms.txt` - nieformalny standard wskazujący modelom najważniejsze treści (nie zastępuje `robots.txt`, tylko uzupełnia).

**`robots.txt`** - domyślnie wpuszczaj wszystko (klasyczne wyszukiwarki + boty AI search-time i treningowe), ogranicz świadomie per projekt:

```
# robots.txt - wygenerowany wg INIT Method (github.com/casperz03/init-method)
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /dziekujemy/
Disallow: /*?utm_

Sitemap: https://TWOJA-DOMENA.pl/sitemap-index.xml
```

**Boty treningowe AI** (blokuj tylko na wyraźne życzenie klienta - "nie chcę, żeby moja treść trenowała modele"): `GPTBot`, `ClaudeBot`, `CCBot`, `Meta-ExternalAgent`, `Bytespider`, `Applebot-Extended`.

⚠️ **`Google-Extended` nie da się czysto rozdzielić na "trening" i "cytowania"** - w przeciwieństwie do OpenAI/Anthropic (gdzie `GPTBot`/`ClaudeBot` to trening, a `OAI-SearchBot`/`Claude-SearchBot` to osobny sygnał cytowań), `Google-Extended` steruje jednocześnie treningiem **i** groundingiem/cytowaniami Gemini. Zablokowanie go nie chroni przed niczym więcej niż samo wyłączenie treningu (Google potwierdza brak wpływu na pozycje i AI Overviews), a kosztuje cytowania w Gemini. Blokuj świadomie, wiedząc że to pakiet "wszystko albo nic" - nie licz na to, że da się zjeść ciastko i je mieć.

⚠️ **`Applebot-Extended` ≠ `Applebot`.** Zablokowanie pierwszego (trening Apple Intelligence) nie wycina drugiego - `Applebot` nadal indeksuje pod Siri i Spotlight. To dwa różne user-agenty o łudząco podobnych nazwach.

Boty search-time/cytowania, warte trzymania otwartymi niezależnie od powyższego: `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `Claude-User`, `PerplexityBot`, `Perplexity-User`.

⚠️ Każda dyrektywa musi należeć do grupy pod `User-agent:` - luźne `Disallow:` bez nagłówka grupy są ignorowane.

## 15. Prawo: RODO, EAA, cookies

Nie jest to porada prawna - przypadki graniczne konsultuj z prawnikiem.

- **Polityka prywatności** - obowiązkowa, jeśli strona zbiera jakiekolwiek dane (formularz kontaktowy też). Zawiera: administratora, cel i podstawę prawną (art. 6 RODO), okres przechowywania, odbiorców danych, prawa osoby.
- **Klauzula RODO (art. 13)** przy formularzu - krótka, bezpośrednio przy polu, nie tylko link.
- Checkboxy zgód - zawsze **opt-in**, nigdy domyślnie zaznaczone.
- **Cookies** - banner z granulacją (niezbędne / analityczne / marketingowe), skrypty trackingowe (GA4, Meta Pixel) ładowane dopiero **po** zgodzie, nie przed.
  - Rozważ **Cloudflare Web Analytics** zamiast/obok GA4 - cookieless, nie wymaga bannera zgody dla samego trackingu ruchu. ⚠️ Nie mierzy zdarzeń własnych (kliknięcie `mailto:`/`tel:`, wysłanie formularza) - jeśli projekt musi śledzić konwersje, nie tylko ruch, samo CWA nie wystarczy.
- **EAA** - obowiązuje od 28.06.2025 w UE, dotyczy usług B2C. Mikroprzedsiębiorstwa (<10 zatrudnionych, obrót/suma bilansowa <2 mln EUR) zwolnione - zaznacz to klientowi, nie zakładaj automatycznie. Standard referencyjny: WCAG 2.1 AA (pokrywa się z sekcją 11).
- **Regulamin** - wymagany tylko przy sprzedaży/koncie użytkownika, nie przy zwykłej wizytówce.

## 16. Decap CMS

Backend `github` (OAuth przez własny Cloudflare Worker jako proxy - **nie** `git-gateway`, który wymaga Netlify Identity i wprowadzałby zależność od Netlify w stacku czysto-Cloudflare).

`public/admin/config.yml`:

```yaml
backend:
  name: github
  repo: nazwa-org/nazwa-repo
  branch: main
  base_url: https://twoj-oauth-worker.workers.dev
  auth_endpoint: auth   # ścieżka autoryzacji na Workerze - dopasuj do implementacji proxy

media_folder: "src/assets/uploads"
public_folder: "/assets/uploads"

collections:
  - name: "pages"
    label: "Strony"
    folder: "src/content/pages"
    create: true
    fields:
      - { label: "Tytuł", name: "title", widget: "string" }
      - { label: "Opis", name: "description", widget: "string" }
      - { label: "Treść", name: "body", widget: "markdown" }
```

`public/admin/index.html`:

```html
<!doctype html>
<html><head><script src="https://unpkg.com/decap-cms@^3.0.0/dist/decap-cms.js"></script></head>
<body></body></html>
```

- Pola w `config.yml` muszą odzwierciedlać schemat Zod w `src/content/config.ts` - literówka w kluczu jednego bez drugiego wysypuje build albo psuje CMS UI po cichu.
- Klient edytuje treść przez `/admin` → Decap commituje bezpośrednio do repo → Cloudflare Pages buduje i wdraża automatycznie (patrz sekcja 17).

## 17. Przygotowanie do wdrożenia

**Domena i DNS:**
- Nameservery domeny wskazane na Cloudflare (rejestrator → NS records).
- Rekord `CNAME`/`A` projektu Pages - **proxied (pomarańczowa chmurka)**, nie "DNS only" - inaczej WAF/CDN/SSL Cloudflare nie działają.

**SSL/TLS (dashboard → SSL/TLS):**
- Tryb: **Full (strict)**.
- Edge Certificates → Always Use HTTPS: **on**.
- Automatic HTTPS Rewrites: on.
- HSTS: włączaj świadomie, dopiero po weryfikacji wszystkich subdomen (trudne do cofnięcia po `preload`).

**Przekierowania - `public/_redirects`** (odpowiednik `.htaccess` na Cloudflare Pages; `.htaccess` to mechanizm Apache i tu nie ma zastosowania):

```
# stary URL → nowy, 301
/stara-podstrona   /nowa-podstrona   301

# przekierowanie z zachowaniem ścieżki (splat)
/blog/*   /aktualnosci/:splat   301
```

⚠️ `_redirects` na Cloudflare Pages **nie wspiera flagi force (`!`) z Netlify** i **nie obsługuje przekierowań na poziomie całej domeny** (np. apex → `www`) - działa wyłącznie na ścieżkach w obrębie jednego projektu Pages. Dla apex↔www i innych przekierowań na poziomie strefy - **Redirect Rules** w dashboardzie Cloudflare, nie `_redirects`.

**Cache - `public/_headers`:**

```
# hashowane assety z builda Astro - cache na rok, bezpiecznie (nazwa pliku zmienia się przy zmianie treści)
/_astro/*
  Cache-Control: public, max-age=31536000, immutable

# HTML - zawsze świeże
/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

⚠️ Cloudflare Pages **nie nadpisuje pasujących reguł `_headers`, tylko je łączy** - jeśli dwa wzorce pasują do tej samej ścieżki, wartości tego samego nagłówka zostają sklejone przecinkiem, nie nadpisane. Dlatego wzorce powyżej celowo się nie pokrywają (`/_astro/*` to tylko assety JS/CSS, `/*.html` to tylko strony) - nigdy nie dodawaj ogólnego `/*` obok bardziej szczegółowej reguły dla tego samego nagłówka, bo wynikowy `Cache-Control` będzie bez sensu dla obu.

**Checklist dashboardu Cloudflare przed pierwszym deployem:**
- [ ] Custom domain podpięty do projektu Pages.
- [ ] DNS record proxied (🟠).
- [ ] SSL/TLS: Full (strict), Always Use HTTPS: on.
- [ ] WAF managed rules + Bot Fight Mode: on.
- [ ] `_redirects` i `_headers` obecne w `public/` przed pierwszym buildem.
- [ ] Environment variables (klucze API, `PUBLIC_*`) ustawione w Settings projektu, nie w kodzie.
- [ ] Cloudflare Web Analytics podpięty (opcjonalnie, cookieless).

## 18. Gotowe szablony

Kopiuj 1:1 do nowego projektu - zero pisania od zera.

**`<head>` - podstawa** (uzupełnij `SEOHead.astro` z §9 o to poniżej):

```html
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<meta name="color-scheme" content="light dark" />
```

**Reset + design tokeny** (`src/styles/global.css` - dostosuj tokeny kolorów/fontów pod projekt, nie kopiuj mechanicznie tego, czego projekt nie używa):

```css
@import '@fontsource-variable/nazwa-fontu';

@layer tokens, reset, components;

@layer tokens {
  :root {
    /* Kolory: prymitywy - surowa paleta, nie używaj poza tą warstwą */
    --color-gray-50: oklch(98% 0.002 271);
    --color-gray-500: oklch(60% 0.005 271);
    --color-gray-900: oklch(19.5% 0.00002 271);
    --color-blue-500: oklch(55.8% 0.189 259.9);

    /* Kolory: semantyczne - jedyne dozwolone poza warstwą tokens */
    --color-bg-canvas: var(--color-gray-50);
    --color-text-primary: var(--color-gray-900);
    --color-text-muted: var(--color-gray-500);
    --color-accent: var(--color-blue-500);

    /* Typografia */
    --font-sans: 'Nazwa Fontu Variable', system-ui, sans-serif;
    --text-size-md: 1rem;
    --text-size-lg: clamp(1.125rem, 1.042rem + 0.417vw, 1.375rem);
    --text-size-2xl: clamp(1.75rem, 1.5rem + 1.25vw, 2.5rem);

    /* Spacing (siatka 4px/8px, nazwa = wartość w px) */
    --space-16: 1rem;
    --space-24: 1.5rem;
    --space-80: 5rem;

    /* Layout */
    --container-max-width: 1440px;
    --navbar-height: 80px;
  }

  @media (prefers-color-scheme: dark) {
    :root {
      --color-bg-canvas: var(--color-gray-900);
      --color-text-primary: var(--color-gray-50);
    }
  }
}

@layer reset {
  *, *::before, *::after {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  html {
    scroll-behavior: smooth;
    scroll-padding-top: var(--navbar-height);
    color-scheme: light dark;
  }

  body {
    min-height: 100dvh;
    font-family: var(--font-sans);
    background-color: var(--color-bg-canvas);
    color: var(--color-text-primary);
    line-height: 1.5;
    text-wrap: pretty;
    -webkit-font-smoothing: antialiased;
  }

  :where(h1, h2, h3, h4, h5, h6) { text-wrap: balance; }

  ul[role="list"], ol[role="list"] { list-style-type: none; }

  img, picture, svg, video, canvas {
    display: block;
    max-width: 100%;
    height: auto;
  }

  table { border-collapse: collapse; width: 100%; }
  textarea { resize: vertical; }

  input, button, textarea, select {
    font: inherit;
    color: inherit;
  }

  button, [type="button"], [type="submit"], select {
    appearance: none;
    background: none;
    border: none;
    cursor: pointer;
  }

  a { color: inherit; text-decoration: inherit; }

  :focus:not(:focus-visible) { outline: none; }

  @media (prefers-reduced-motion: reduce) {
    *, *::before, *::after {
      animation-duration: 0.01ms !important;
      animation-iteration-count: 1 !important;
      transition-duration: 0.01ms !important;
      scroll-behavior: auto !important;
    }
  }
}

@layer components {
  .container {
    width: 100%;
    max-width: var(--container-max-width);
    margin-inline: auto;
    padding-inline: var(--space-16);

    @media (width >= 768px) { padding-inline: var(--space-24); }
    @media (width >= 1024px) { padding-inline: var(--space-80); }
  }
}
```

To zestaw bazowy - pełny reset (cienie, z-index, czasy animacji, kompletny reset formularzy) rozbudowuj per projekt wg realnej potrzeby.

**`.gitignore`** (do roota, przed pierwszym commitem - jeśli sekret trafi do historii gita, samo dodanie do `.gitignore` już go nie usunie):

```
# --- Zależności ---
node_modules/

# --- Build / output ---
dist/
.astro/
.wrangler/

# --- Zmienne środowiskowe (WRAŻLIWE - nigdy nie commituj) ---
.env
.env.local
.env.*.local

# --- Logi ---
npm-debug.log*
pnpm-debug.log*

# --- Edytor / IDE ---
.vscode/*
!.vscode/extensions.json
.idea/

# --- System operacyjny ---
.DS_Store
Thumbs.db

# --- Testy / coverage ---
coverage/

# --- Klucze / certyfikaty ---
*.pem
*.key
```

**Strona 404** (`src/pages/404.astro` - Astro rozpoznaje ten plik automatycznie):

```astro
---
Astro.response.status = 404; // istotne tylko w trybie SSR, w SSG hosting robi to sam
---
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <title>404 - strona nie została znaleziona</title>
  <meta name="robots" content="noindex, follow" />
</head>
<body>
  <main>
    <h1>Strona nie istnieje</h1>
    <p>Szukana strona mogła zostać przeniesiona lub usunięta.</p>
    <a href="/" class="btn">Wróć na stronę główną</a>
  </main>
</body>
</html>
```

**Stopka z automatycznym rokiem** - jeden wiersz, zero JS:

```astro
<footer>
  <p>&copy; {new Date().getFullYear()} Nazwa Firmy. Wszelkie prawa zastrzeżone.</p>
</footer>
```

Wyrażenie `{}` w `.astro` liczy się w buildzie (SSG), nie w przeglądarce - stąd zero JS, zero `js-current-year`, zero osobnego pliku. Jedyny kompromis: rok "zamraża się" na wartości z ostatniego builda, nie z realnego czasu odwiedzającego. Dla stopki to nieistotne - projekt i tak rebuilduje się przy każdej zmianie treści przez Decap (§16), więc rok koryguje się sam najpóźniej przy pierwszym commicie po Nowym Roku. Jeśli projekt bywa całkowicie nieaktualizowany miesiącami na przełomie roku, rozważ dodanie tego jednego builda ręcznie w styczniu - to i tak tańsze niż utrzymywanie osobnego skryptu dla tak drobnej rzeczy.

**`public/site.webmanifest`** (plik, na który wskazuje `<link rel="manifest">` w sekcji 9):

```json
{
  "name": "Nazwa Firmy - pełna nazwa",
  "short_name": "Nazwa Firmy",
  "description": "Krótki opis strony/firmy.",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#1a56db",
  "lang": "pl",
  "icons": [
    { "src": "/android-chrome-192x192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/android-chrome-512x512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

`theme_color` w manifeście musi być spójny z `<meta name="theme-color">` w `<head>`.

## 19. Master checklist przed przekazaniem projektu

Pozycje z tagiem dotyczą tylko projektów z danym zakresem - pomiń je świadomie, jeśli projekt nie ma formularzy/CMS/lokalnego SEO, zamiast odznaczać każdą jako "nie dotyczy" z osobna.

**Kod:** struktura zgodna z sekcją 2 · `tsconfig` na `strictest` · zero duplikacji (DRY) · nazewnictwo zgodne z sekcją 3 · JS przez `.js-*`, nigdy `#id` · `.gitignore` uzupełniony **przed** pierwszym commitem · brak sekretów w repo.

**SEO:** `site` ustawione w `astro.config.mjs` · `trailingSlash` spójne · unikalny title/description na każdej podstronie · self-canonical wszędzie · jeden `h1` · `robots.txt` + `sitemap.xml` · strona 404 z `noindex, follow` · OG + Twitter Card · JSON-LD (min. `LocalBusiness`/`Organization`) · `llms.txt`.

**Lokalne SEO** `[local]` (przy projektach dla lokalnych firm): NAP spójny wszędzie · schema `LocalBusiness` z adresem/geo/godzinami · Google Business Profile założony i powiązany z domeną · mapa Google na stronie kontaktowej · adres/telefon/godziny w tekście, nie tylko na obrazku.

**Wydajność:** LCP bez lazy + `fetchpriority="high"` · obrazy poniżej foldu lazy z `width`/`height` · fonty self-hosted `.woff2` + `swap` · Lighthouse: LCP < 2.5s, INP < 200ms, CLS < 0.1.

**Accessibility:** kontrast WCAG AA · nawigacja klawiaturą + `:focus-visible` · skip link · `alt` wszędzie · `prefers-reduced-motion` respektowane (w tym w GSAP).

**Bezpieczeństwo:** `_headers` z CSP/HSTS/X-Frame-Options · SSL/TLS Full (strict) + Always Use HTTPS · WAF + Bot Fight Mode on · formularz z honeypotem `[form]` · `npm audit` czysty.

**Prawo:** polityka prywatności (jeśli formularz) · klauzula RODO przy formularzu · zgody opt-in · banner cookies z granulacją (jeśli tracking) · sprawdzone EAA.

**Cloudflare/deploy:** DNS proxied · `_redirects` z mapowaniem starych URL (jeśli redesign) · custom domain aktywny · zmienne środowiskowe w dashboardzie, nie w kodzie.

**CMS** `[cms]`: `/admin` dostępny, OAuth Worker działa, klient przetestował dodanie/edycję treści.

**Ostatnie sprawdzenie:** favicon + `site.webmanifest` · rok w stopce automatyczny · GSC + analytics podłączone i zweryfikowane · test na realnym urządzeniu mobilnym · linki wewnętrzne/zewnętrzne bez 404, `target="_blank" rel="noopener"` przy zewnętrznych.

## 20. Referencje

- Baseline (wsparcie przeglądarek): [web.dev/baseline](https://web.dev/baseline)
- Astro: [docs.astro.build](https://docs.astro.build/) · [struktura projektu](https://docs.astro.build/en/basics/project-structure/) · [obrazy](https://docs.astro.build/en/guides/images/) · [sitemap](https://docs.astro.build/en/guides/integrations-guide/sitemap/) · [View Transitions](https://docs.astro.build/en/guides/view-transitions/)
- MDN: [developer.mozilla.org](https://developer.mozilla.org/)
- GSAP: [gsap.com/docs](https://gsap.com/docs/)
- Decap CMS: [decapcms.org/docs](https://decapcms.org/docs/)
- Cloudflare Pages: [developers.cloudflare.com/pages](https://developers.cloudflare.com/pages/)
- Google Search Central: [developers.google.com/search](https://developers.google.com/search)
- Core Web Vitals: [web.dev/articles/vitals](https://web.dev/articles/vitals)
- WCAG: [w3.org/WAI/standards-guidelines/wcag](https://www.w3.org/WAI/standards-guidelines/wcag/)
