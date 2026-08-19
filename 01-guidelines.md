# 01 · Guidelines

Dokument referencyjny do wklejenia jako kontekst projektu (CLAUDE.md / notatka w narzędziu AI). Obowiązuje przy każdym projekcie klienckim w ramach INIT Method. Uzupełnij [`04-context.md`](04-context.md) danymi konkretnego projektu — ten plik zawiera zasady stałe, niezależne od klienta.

**Uwaga o aktualności:** sekcje *SEO pod AI (GEO)*, *Prawo* i *Cloudflare* opisują szybko zmieniający się stan — zweryfikuj je przy większej rewizji dokumentu.

## Stack

| Warstwa | Narzędzie | Rola |
|---|---|---|
| Framework | **Astro**, `output: 'static'` | SSG — HTML w całości w buildzie, zero JS domyślnie |
| Style | **Vanilla CSS** | CSS Nesting, custom properties, bez frameworków (Bootstrap/Tailwind) |
| Animacje | **GSAP** (+ ScrollTrigger) | Jedyna sankcjonowana biblioteka JS poza vanilla — do scroll-triggered i złożonych animacji |
| CMS | **Decap CMS** (backend `github` + OAuth-proxy na Cloudflare Worker) | Edycja treści przez klienta, commit do repo, bez Netlify |
| Treść | **Astro Content Collections** + Zod | Walidacja frontmatteru, typowany dostęp do treści z Decap |
| Hosting | **Cloudflare Pages** | Build z gita, edge CDN, darmowy tier bez limitu requestów |
| DNS / SSL / WAF | **Cloudflare** | Domena, certyfikaty, ochrona przed botami/DDoS |
| Audyt SEO | **DataForSEO** | Narzędzie procesowe (workflow), nie zależność runtime |

Domyślny wybór: **Astro static + Cloudflare Pages** dla całego katalogu projektów (strony wizytówkowe/ofertowe/blogi). SSR (adapter `@astrojs/cloudflare`, Workers) tylko gdy projekt tego realnie wymaga — patrz sekcja 13.

## 1. Zasady ogólne

- **DRY** — trzeci powtórzony blok = wydziel komponent `.astro`/zmienną CSS.
- **KISS** — brak abstrakcji i bibliotek tam, gdzie wystarczy vanilla JS/CSS lub natywne API przeglądarki.
- **Mobile-first** — style od najmniejszego viewportu w górę (`min-width` w media queries).
- **Zgodność ze standardami** — aktualna specyfikacja HTML/CSS/JS, Core Web Vitals, Google Search Central.

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
│   ├── content/             # Content Collections — tu commituje Decap
│   │   └── config.ts        # schematy Zod
│   ├── layouts/
│   ├── pages/
│   ├── scripts/             # vanilla JS / inicjalizacja GSAP
│   └── styles/               # global.css (custom properties, reset)
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

- `src/pages/` wymagany — brak = brak routingu.
- Własny kod (CSS/JS) zawsze w `src/`, nigdy w `public/` — tylko wtedy Astro go zbunduje i zoptymalizuje.
- `public/` wyłącznie dla plików trafiających 1:1 do builda.

**Konfiguracja bazowa (`astro.config.mjs`):**

```js
import { defineConfig } from 'astro/config';
import sitemap from '@astrojs/sitemap';

export default defineConfig({
  site: 'https://domena.pl',    // WYMAGANE — bez tego sitemap i canonicale wskażą na localhost
  trailingSlash: 'always',       // jedna, spójna forma URL — trzymaj się konsekwentnie
  integrations: [sitemap()],
  // adapter NIE jest potrzebny dla output: 'static' — Cloudflare Pages serwuje build bezpośrednio
});
```

## 3. Konwencje nazewnictwa

| Element | Konwencja | Przykład |
|---|---|---|
| Zmienne, funkcje JS | camelCase | `getUserData`, `isMenuOpen` |
| Klasy CSS, ID | kebab-case | `.hero-section`, `#main-nav` |
| Pliki komponentów Astro | PascalCase | `Header.astro`, `ContactForm.astro` |
| Selektory pod JS | prefiks `js-` | `.js-toggle-menu` |

- Wszystkie nazwy (zmienne, klasy, ID, funkcje) — **zawsze po angielsku**. Wyjątek: anchory URL istotne SEO/UX dla PL-użytkownika (`#cennik`, `#kontakt`).
- **JS nigdy nie odwołuje się do `id`** — zawsze dedykowana klasa `js-*`, oddzielona od klas stylistycznych. Zmiana stylu nie może przypadkiem zerwać skryptu i odwrotnie.

```html
<!-- źle -->
<button id="menu-toggle" class="btn btn-icon">...</button>
<script>document.getElementById('menu-toggle')...</script>

<!-- dobrze -->
<button class="btn btn-icon js-menu-toggle">...</button>
<script>document.querySelector('.js-menu-toggle')...</script>
```

## 4. HTML — semantyka

- Tagi HTML5: `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, `figure`, `time`, `dialog`.
- Jeden `h1` na stronę, logiczna kolejność `h2 → h3` bez przeskoków.
- `alt` na każdym obrazie treściowym (`alt=""` dla dekoracyjnych).
- Formularze: `<label for="">` powiązane z polem, `required`, `autocomplete`, właściwy `type`.

## 5. CSS — vanilla, nowoczesny

- Bez frameworków (Bootstrap/Tailwind), chyba że projekt tego jawnie wymaga.
- **CSS Nesting** natywny (bez preprocesora):

```css
.card {
  padding: 1rem;

  & .card-title { font-size: 1.25rem; }
  &:hover { box-shadow: 0 2px 8px rgb(0 0 0 / 10%); }
}
```

- Jednostki świadomie: `rem` (typografia/spacing), `%`/`fr` (siatki), `svh/dvh` (pełnoekranowe sekcje — `dvh` zamiast `vh` na mobile, unika skoku przez pasek adresu), `clamp()` (płynna typografia bez sterty media queries), `min()`/`max()`.
- Layout: flexbox/grid zamiast floatów/absolute tam, gdzie możliwe.

**Custom properties w `:root`** (raz w `src/styles/global.css`, nigdy nie hardkoduj wartości w wielu plikach):

```css
:root {
  /* Kolory */
  --color-primary: #1a56db;
  --color-text: #111827;
  --color-bg: #ffffff;
  --color-border: #e5e7eb;

  /* Typografia */
  --font-base: 'Inter', system-ui, sans-serif;
  --font-size-lg: clamp(1.25rem, 2vw, 1.75rem);
  --line-height-base: 1.6;

  /* Spacing */
  --space-xs: 0.25rem;
  --space-sm: 0.5rem;
  --space-md: 1rem;
  --space-lg: 2rem;
  --space-xl: 4rem;

  /* Layout */
  --container-max-width: 1200px;
  --radius-base: 0.5rem;
  --shadow-sm: 0 1px 3px rgb(0 0 0 / 10%);
  --transition-base: 0.2s ease-in-out;
}
```

Konwencja: `--kategoria-wariant`, kebab-case, po angielsku.

## 6. JavaScript — vanilla + GSAP

- `const`/`let` wyłącznie — zero `var`.
- `querySelector`/`querySelectorAll`, nie `getElementById`.
- Nowoczesna składnia: arrow functions, template literals, destructuring, `async/await`, moduły ES, `?.`, `??`.
- Jedyne źródło prawdy dla API/składni: **MDN** — nie kopiuj z blogów/SO bez weryfikacji.
- Interakcje zawsze przez `.js-*` (sekcja 3).

**GSAP — standard stacku, nie wyjątek:**

- Do prostych przejść/hover/fade — zawsze czysty CSS (`transition`, `@keyframes`), taniej niż JS.
- GSAP + `ScrollTrigger` — do scroll-triggered animacji, timeline'ów, złożonych sekwencji.
- Ładuj tylko na stronach, które faktycznie animacji używają — nie globalnie do każdej podstrony (waga pliku wpływa na CWV).
- **Musi respektować `prefers-reduced-motion`** — GSAP nie robi tego automatycznie:

```js
const prefersReducedMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
if (!prefersReducedMotion) {
  // dopiero tutaj inicjalizuj animacje GSAP
}
```

**Gotcha: Astro View Transitions + GSAP.** Jeśli projekt używa `<ClientRouter />` (View Transitions), `DOMContentLoaded` odpala się tylko raz — przy kolejnych nawigacjach nie. Inicjalizuj GSAP/ScrollTrigger na evencie `astro:page-load`, nie `DOMContentLoaded`, inaczej animacje przestają działać po pierwszej nawigacji SPA-like.

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

- `interface` — kształty obiektów, propsy komponentów. `type` — unie, przecięcia, aliasy.
- Unikaj `enum` (nieintuicyjne zachowanie) — używaj union literal types (`type Theme = 'light' | 'dark'`) lub `as const` + `typeof`.
- `unknown` zamiast `any` dla danych z zewnątrz przed walidacją. `satisfies` zamiast `as`. `import type` dla importów wyłącznie-typów.
- Unikaj non-null assertion (`!`) — sprawdzaj jawnie (`if`, `?.`).
- Dane z zewnątrz (formularz, CMS, API) — Zod + `z.infer<typeof schema>`, typ i walidacja nigdy się nie rozjeżdżają.

## 8. Wydajność / Core Web Vitals

- Obrazy poniżej foldu: `loading="lazy"`. Obraz LCP (hero): **bez** lazy, `loading="eager"` + `fetchpriority="high"`.
- `width`/`height` (lub `aspect-ratio`) na każdym obrazie — zapobiega CLS.
- `astro:assets` (`<Image />`) zamiast surowych `<img>` z `public/`, gdzie to możliwe — auto AVIF/WebP, `srcset`.

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.jpg';
---
<!-- LCP — priorytetowo -->
<Image src={heroImage} alt="Opis" widths={[400, 800, 1200]} sizes="(max-width: 768px) 100vw, 1200px" loading="eager" fetchpriority="high" format="avif" />
<!-- poniżej foldu -->
<Image src={heroImage} alt="Opis" loading="lazy" format="webp" />
```

- Fonty: self-hosted (bez CDN — szybciej, bez dodatkowego DNS, bez transferu IP do zewnętrznego serwera → RODO), `.woff2`, variable font, `font-display: swap`, `preload` tylko dla fontu krytycznego.
- Cel: LCP < 2.5s, INP < 200ms, CLS < 0.1 (Lighthouse / PageSpeed Insights).

## 9. SEO

- Unikalny `<title>` (≤ 60 zn.) i `<meta name="description">` (≤ 155–160 zn.) na każdej podstronie.
- Jeden `h1` zgodny z intencją wyszukiwania danej podstrony.
- Semantyczny URL (kebab-case, bez zbędnych parametrów).
- JSON-LD tam, gdzie dotyczy: `LocalBusiness`, `Article`, `FAQPage`, `BreadcrumbList`.
- `robots.txt` + `sitemap.xml` (`@astrojs/sitemap`).
- **Self-canonical domyślnie na każdej podstronie** — generowany automatycznie z `Astro.url` + `site`, nie ręcznie. Wyjątek: świadome duplikaty (filtr/sortowanie, wersja do druku, UTM) → canonical wskazuje wersję główną.

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
const canonical = canonicalOverride ?? new URL(Astro.url.pathname, Astro.site).href;
const ogImage = new URL(image, Astro.site).href;
---
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>{title}</title>
<meta name="description" content={description} />
<link rel="canonical" href={canonical} />
{noindex && <meta name="robots" content="noindex, follow" />}
<meta property="og:type" content="website" />
<meta property="og:title" content={title} />
<meta property="og:description" content={description} />
<meta property="og:url" content={canonical} />
<meta property="og:image" content={ogImage} />
<meta name="twitter:card" content="summary_large_image" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="manifest" href="/site.webmanifest" />
```

## 10. Lokalne SEO (projekty dla lokalnych firm)

- **NAP spójny** (Name, Address, Phone) — identyczne dane firmy na stronie, w stopce, w Google Business Profile i innych wizytówkach (Panorama Firm, Facebook). Niespójność to jeden z najczęstszych błędów obniżających lokalny ranking.
- Nazwa miasta/regionu naturalnie w `title`/`h1`/treści kluczowych podstron — bez keyword stuffingu.
- Google Business Profile założony/zweryfikowany, powiązany z tą samą domeną.
- Mapa Google osadzona na stronie kontaktowej (embed z prawidłowym adresem).
- Pełny adres, telefon, godziny — w tekście, nie tylko na obrazku.
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

- **`prefers-reduced-motion`** — globalny fallback w `global.css`:

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

GSAP tego nie respektuje automatycznie — sprawdzaj jawnie w JS (sekcja 6).

## 12. Bezpieczeństwo

**Nagłówki** — plik `public/_headers` (Cloudflare Pages czyta ten sam format co Netlify):

```
/*
  X-Frame-Options: SAMEORIGIN
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), microphone=(), camera=()
  Strict-Transport-Security: max-age=63072000; includeSubDomains
  Content-Security-Policy: default-src 'self'; img-src 'self' data: https:; script-src 'self'; style-src 'self' 'unsafe-inline'
```

- **Cloudflare dashboard, SSL/TLS → Overview:** tryb **Full (strict)**. **Edge Certificates:** Always Use HTTPS = on. HSTS włączaj dopiero po potwierdzeniu, że wszystkie subdomeny wspierają HTTPS (preload jest trudny do cofnięcia).
- **WAF managed rules + Bot Fight Mode** — włącz w dashboardzie, darmowe na każdym planie, minimalny wysiłek za realną ochronę.
- SRI (`integrity` + `crossorigin`) przy skryptach z zewnętrznego CDN.
- **Formularze:** honeypot jako pierwsza linia obrony przed spamem, walidacja client + service-side (Web3Forms/Formspree).
- Żaden sekret w kodzie/commitach — zmienne środowiskowe w Cloudflare Pages (dashboard → Settings → Environment variables). `PUBLIC_*` trafia do bundla JS wysyłanego do przeglądarki — traktuj jako jawne.
- `npm audit` cyklicznie, zwłaszcza przed przekazaniem projektu.

## 13. Renderowanie: SSG domyślnie

**Treść, która ma być zaindeksowana, nigdy nie może zależeć wyłącznie od JS po stronie klienta (CSR).**

- **SSG** (`output: 'static'`, domyślne) — strona wizytówkowa/ofertowa/blog → zawsze. Zero JS wymagane do zobaczenia treści, najlepsze pod SEO i CWV.
- **SSR** (adapter `@astrojs/cloudflare`, Cloudflare Workers) — tylko gdy treść realnie zależy od requestu/sesji (panel klienta, dane real-time). Zmienia strukturę deploy (Workers zamiast czystego Pages) — decyzja świadoma, nie domyślna.
- Astro Islands: reszta strony to statyczny HTML, JS tylko lokalnie na interaktywnych fragmentach (`client:visible`, `client:idle`).

## 14. SEO pod AI/LLM (GEO)

- Treść w HTML, nie tylko w JS — wiele crawlerów AI nie wykonuje JavaScriptu (patrz sekcja 13).
- Jasna hierarchia nagłówków — modele segmentują treść po `h1→h2→h3`.
- Jedna jednoznaczna odpowiedź na pytanie w jednym akapicie — zwiększa szansę cytowania w AI Overviews/ChatGPT.
- JSON-LD (`FAQPage`, `HowTo`, `LocalBusiness`) — bezpośredni sygnał dla modeli, czym jest dana treść:

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

- `public/llms.txt` — nieformalny standard wskazujący modelom najważniejsze treści (nie zastępuje `robots.txt`, tylko uzupełnia).

**`robots.txt`** — domyślnie wpuszczaj wszystko (klasyczne wyszukiwarki + boty AI search-time i treningowe), ogranicz świadomie per projekt:

```
User-agent: *
Allow: /
Disallow: /admin/
Disallow: /dziekujemy/
Disallow: /*?utm_

Sitemap: https://TWOJA-DOMENA.pl/sitemap-index.xml
```

Jeśli klient nie chce treści w zbiorach treningowych AI, ale ma zostać widoczny w cytowaniach: `Disallow: /` dla `GPTBot`, `ClaudeBot`, `Google-Extended` — zostaw `Allow: /` dla `OAI-SearchBot`, `ChatGPT-User`, `Claude-SearchBot`, `PerplexityBot`.

⚠️ Każda dyrektywa musi należeć do grupy pod `User-agent:` — luźne `Disallow:` bez nagłówka grupy są ignorowane.

## 15. Prawo: RODO, EAA, cookies

Nie jest to porada prawna — przypadki graniczne konsultuj z prawnikiem.

- **Polityka prywatności** — obowiązkowa, jeśli strona zbiera jakiekolwiek dane (formularz kontaktowy też). Zawiera: administratora, cel i podstawę prawną (art. 6 RODO), okres przechowywania, odbiorców danych, prawa osoby.
- **Klauzula RODO (art. 13)** przy formularzu — krótka, bezpośrednio przy polu, nie tylko link.
- Checkboxy zgód — zawsze **opt-in**, nigdy domyślnie zaznaczone.
- **Cookies** — banner z granulacją (niezbędne / analityczne / marketingowe), skrypty trackingowe (GA4, Meta Pixel) ładowane dopiero **po** zgodzie, nie przed.
  - Rozważ **Cloudflare Web Analytics** zamiast/obok GA4 — cookieless, nie wymaga bannera zgody dla samego trackingu ruchu.
- **EAA** — obowiązuje od 28.06.2025 w UE, dotyczy usług B2C. Mikroprzedsiębiorstwa (<10 zatrudnionych, obrót/suma bilansowa <2 mln EUR) zwolnione — zaznacz to klientowi, nie zakładaj automatycznie. Standard referencyjny: WCAG 2.1 AA (pokrywa się z sekcją 11).
- **Regulamin** — wymagany tylko przy sprzedaży/koncie użytkownika, nie przy zwykłej wizytówce.

## 16. Decap CMS

Backend `github` (OAuth przez własny Cloudflare Worker jako proxy — **nie** `git-gateway`, który wymaga Netlify Identity i wprowadzałby zależność od Netlify w stacku czysto-Cloudflare).

`public/admin/config.yml`:

```yaml
backend:
  name: github
  repo: nazwa-org/nazwa-repo
  branch: main
  base_url: https://twoj-oauth-worker.workers.dev

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

- Pola w `config.yml` muszą odzwierciedlać schemat Zod w `src/content/config.ts` — literówka w kluczu jednego bez drugiego wysypuje build albo psuje CMS UI po cichu.
- Klient edytuje treść przez `/admin` → Decap commituje bezpośrednio do repo → Cloudflare Pages buduje i wdraża automatycznie (patrz sekcja 17).

## 17. Przygotowanie do wdrożenia

**Domena i DNS:**
- Nameservery domeny wskazane na Cloudflare (rejestrator → NS records).
- Rekord `CNAME`/`A` projektu Pages — **proxied (pomarańczowa chmurka)**, nie "DNS only" — inaczej WAF/CDN/SSL Cloudflare nie działają.

**SSL/TLS (dashboard → SSL/TLS):**
- Tryb: **Full (strict)**.
- Edge Certificates → Always Use HTTPS: **on**.
- Automatic HTTPS Rewrites: on.
- HSTS: włączaj świadomie, dopiero po weryfikacji wszystkich subdomen (trudne do cofnięcia po `preload`).

**Przekierowania — `public/_redirects`** (odpowiednik `.htaccess` na Cloudflare Pages; `.htaccess` to mechanizm Apache i tu nie ma zastosowania):

```
# stary URL → nowy, 301
/stara-podstrona   /nowa-podstrona   301

# apex → www (albo odwrotnie — wybierz jedną formę, spójnie z sekcją 3 astro.config)
https://domena.pl/*   https://www.domena.pl/:splat   301!
```

Dla przekierowań na poziomie całej strefy (nie tylko w obrębie jednego projektu Pages) — **Redirect Rules** w dashboardzie Cloudflare zamiast `_redirects`.

**Cache — `public/_headers`:**

```
# hashowane assety z builda Astro — cache na rok, bezpiecznie (nazwa pliku zmienia się przy zmianie treści)
/_astro/*
  Cache-Control: public, max-age=31536000, immutable

# HTML — zawsze świeże
/*.html
  Cache-Control: public, max-age=0, must-revalidate
```

**Checklist dashboardu Cloudflare przed pierwszym deployem:**
- [ ] Custom domain podpięty do projektu Pages.
- [ ] DNS record proxied (🟠).
- [ ] SSL/TLS: Full (strict), Always Use HTTPS: on.
- [ ] WAF managed rules + Bot Fight Mode: on.
- [ ] `_redirects` i `_headers` obecne w `public/` przed pierwszym buildem.
- [ ] Environment variables (klucze API, `PUBLIC_*`) ustawione w Settings projektu, nie w kodzie.
- [ ] Cloudflare Web Analytics podpięty (opcjonalnie, cookieless).

## 18. Gotowe szablony

Kopiuj 1:1 do nowego projektu — zero pisania od zera.

**`.gitignore`** (do roota, przed pierwszym commitem — jeśli sekret trafi do historii gita, samo dodanie do `.gitignore` już go nie usunie):

```
# --- Zależności ---
node_modules/

# --- Build / output ---
dist/
.astro/
.wrangler/

# --- Zmienne środowiskowe (WRAŻLIWE — nigdy nie commituj) ---
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

**Strona 404** (`src/pages/404.astro` — Astro rozpoznaje ten plik automatycznie):

```astro
---
Astro.response.status = 404; // istotne tylko w trybie SSR, w SSG hosting robi to sam
---
<html lang="pl">
<head>
  <meta charset="UTF-8" />
  <title>404 – strona nie została znaleziona</title>
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

**Stopka z automatycznym rokiem** — nigdy nie trzeba ręcznie aktualizować co styczeń:

```html
<footer>
  <p>&copy; <span class="js-current-year">2026</span> Nazwa Firmy. Wszelkie prawa zastrzeżone.</p>
</footer>
```

```js
// src/scripts/footer-year.js
const yearEl = document.querySelector('.js-current-year');
if (yearEl) yearEl.textContent = new Date().getFullYear();
```

Wartość w `<span>` to fallback statyczny — jeśli JS się nie wykona, stopka i tak pokaże sensowny rok.

**`public/site.webmanifest`** (plik, na który wskazuje `<link rel="manifest">` w sekcji 9):

```json
{
  "name": "Nazwa Firmy — pełna nazwa",
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

**Kod:** struktura zgodna z sekcją 2 · `tsconfig` na `strictest` · zero duplikacji (DRY) · nazewnictwo zgodne z sekcją 3 · JS przez `.js-*`, nigdy `#id` · `.gitignore` uzupełniony **przed** pierwszym commitem · brak sekretów w repo.

**SEO:** `site` ustawione w `astro.config.mjs` · `trailingSlash` spójne · unikalny title/description na każdej podstronie · self-canonical wszędzie · jeden `h1` · `robots.txt` + `sitemap.xml` · strona 404 z `noindex, follow` · OG + Twitter Card · JSON-LD (min. `LocalBusiness`/`Organization`) · `llms.txt`.

**Lokalne SEO** (przy projektach dla lokalnych firm): NAP spójny wszędzie · schema `LocalBusiness` z adresem/geo/godzinami · Google Business Profile założony i powiązany z domeną · mapa Google na stronie kontaktowej · adres/telefon/godziny w tekście, nie tylko na obrazku.

**Wydajność:** LCP bez lazy + `fetchpriority="high"` · obrazy poniżej foldu lazy z `width`/`height` · fonty self-hosted `.woff2` + `swap` · Lighthouse: LCP < 2.5s, INP < 200ms, CLS < 0.1.

**Accessibility:** kontrast WCAG AA · nawigacja klawiaturą + `:focus-visible` · skip link · `alt` wszędzie · `prefers-reduced-motion` respektowane (w tym w GSAP).

**Bezpieczeństwo:** `_headers` z CSP/HSTS/X-Frame-Options · SSL/TLS Full (strict) + Always Use HTTPS · WAF + Bot Fight Mode on · formularz z honeypotem · `npm audit` czysty.

**Prawo:** polityka prywatności (jeśli formularz) · klauzula RODO przy formularzu · zgody opt-in · banner cookies z granulacją (jeśli tracking) · sprawdzone EAA.

**Cloudflare/deploy:** DNS proxied · `_redirects` z mapowaniem starych URL (jeśli redesign) · custom domain aktywny · zmienne środowiskowe w dashboardzie, nie w kodzie.

**CMS:** `/admin` dostępny, OAuth Worker działa, klient przetestował dodanie/edycję treści.

**Ostatnie sprawdzenie:** favicon + `site.webmanifest` · rok w stopce automatyczny · GSC + analytics podłączone i zweryfikowane · test na realnym urządzeniu mobilnym · linki wewnętrzne/zewnętrzne bez 404, `target="_blank" rel="noopener"` przy zewnętrznych.

## 20. Referencje

- Astro: [docs.astro.build](https://docs.astro.build/) · [struktura projektu](https://docs.astro.build/en/basics/project-structure/) · [obrazy](https://docs.astro.build/en/guides/images/) · [sitemap](https://docs.astro.build/en/guides/integrations-guide/sitemap/) · [View Transitions](https://docs.astro.build/en/guides/view-transitions/)
- MDN: [developer.mozilla.org](https://developer.mozilla.org/)
- GSAP: [gsap.com/docs](https://gsap.com/docs/)
- Decap CMS: [decapcms.org/docs](https://decapcms.org/docs/)
- Cloudflare Pages: [developers.cloudflare.com/pages](https://developers.cloudflare.com/pages/)
- Google Search Central: [developers.google.com/search](https://developers.google.com/search)
- Core Web Vitals: [web.dev/articles/vitals](https://web.dev/articles/vitals)
- WCAG: [w3.org/WAI/standards-guidelines/wcag](https://www.w3.org/WAI/standards-guidelines/wcag/)
