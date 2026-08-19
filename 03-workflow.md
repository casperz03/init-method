# 03 · Workflow

Ścieżka produkcyjna projektu - 10 etapów, od discovery do utrzymania po starcie. Każdy etap ma jasne wyjście (deliverable), zanim przejdziesz dalej. Pomijanie etapów (najczęściej: Badanie SEO albo Architektura informacji) to najczęstsza przyczyna przeróbek na późniejszym etapie, nie oszczędność czasu.

**Powiązanie z resztą INIT Method:** [`02-brief-template.md`](02-brief-template.md) wypełniasz w etapie 1 · [`01-guidelines.md`](01-guidelines.md) stosujesz w etapach 4–9 (odnośniki § przy każdym punkcie) · [`04-context.md`](04-context.md) aktualizujesz na bieżąco, w miarę podejmowania decyzji - nie czekaj do końca projektu.

Domyślny stack realizacji: **Astro + Cloudflare Pages + Decap CMS** (zgodnie z `01-guidelines.md`). WordPress/inny hosting - tylko jako świadomy wyjątek, jeśli klient tego wymaga (np. przejęcie utrzymania przez inny zespół), nie jako równorzędna domyślna opcja.

## 1. Discovery i briefing

- Wywiad z klientem: model biznesowy, USP, obszar działania → wypełnij `02-brief-template.md`.
- Analiza konkurencji: 3–5 lokalnych + 1–2 wzorcowych.
- Cele i mierzalne KPI (leady, telefony, formularz).
- Zakres i harmonogram - zapis w CRM/notatniku projektowym.

**Wyjście etapu:** wypełniony brief, zaakceptowany zakres i harmonogram.

## 2. Badanie SEO

- Keyword research w DataForSEO: frazy główne + long-tail, klastry.
- Audyt techniczny (tylko przy redesignie): Screaming Frog, GSC.
- Mapa treści: frazy przypisane do konkretnych podstron.

**Wyjście etapu:** mapa treści z przypisanymi frazami.

## 3. Architektura informacji

- Sitemap: płaska struktura, max 3 poziomy.
- Briefy treściowe: fraza główna, nagłówki H2/H3, długość, CTA.
- User flow: ścieżka od wejścia do konwersji.

**Wyjście etapu:** sitemap i briefy treściowe gotowe do designu.

## 4. Design (Figma)

> **Dla AI dostarczyć (kontekst do designu):** kontekst biznesu klienta, zrzuty stron, które się podobają, mood board. Im więcej konkretu na wejściu, tym mniej rund poprawek.

- Wireframe: układ i hierarchia, bez kolorów/fontów.
- Design system: paleta 2-kolorowa + akcent, skala typografii, siatka 8px.
- UI hi-fi: mobile-first, kontrast wg WCAG AA ustalony już w designie, nie poprawiany po fakcie ([`01-guidelines.md` §11](01-guidelines.md#11-accessibility-a11y)).
- Akceptacja klienta: max 1–2 warianty, ustalona z góry liczba rund poprawek.

**Wyjście etapu:** projekt hi-fi zaakceptowany przez klienta.

## 5. Development

- Setup projektu zgodny z [`01-guidelines.md` §2–3](01-guidelines.md#2-struktura-projektu) (struktura, `astro.config.mjs`, nazewnictwo).
- Komponenty i layout: semantyczny HTML, poprawna hierarchia nagłówków ([§4](01-guidelines.md#4-html---semantyka)).
- SEO on-page: unikalne meta title/description, schema.org ([§9](01-guidelines.md#9-seo)).
- Integracja CMS ([Decap, §16](01-guidelines.md#16-decap-cms)): test edycji z perspektywy klienta, nie tylko developera.

**Wyjście etapu:** działający build ze szkieletem treści, CMS podłączony i przetestowany.

## 6. Wdrożenie treści

- Copywriting wg briefów z etapu 3 - pod frazę, ale dla człowieka.
- Obrazy: kompresja WebP/AVIF, alt opisowy z kontekstem ([§8](01-guidelines.md#8-wydajność--core-web-vitals)).
- Publikacja: self-review - zero CTA-placeholderów i tekstów "lorem ipsum" na produkcji.

**Wyjście etapu:** strona z docelową treścią, bez placeholderów.

## 7. Optymalizacja przed startem

- Core Web Vitals: LCP < 2.5s, CLS < 0.1, INP < 200ms ([§8](01-guidelines.md#8-wydajność--core-web-vitals)).
- Dostępność WCAG/EAA: kontrast, focus states, nawigacja klawiaturą ([§11](01-guidelines.md#11-accessibility-a11y), [§15](01-guidelines.md#15-prawo-rodo-eaa-cookies)).
- RODO/cookies: realny wybór, polityka prywatności, zgody opt-in ([§15](01-guidelines.md#15-prawo-rodo-eaa-cookies)).
- `sitemap.xml` i `robots.txt`: sprawdź, czy nie blokują strony przypadkiem ([§9](01-guidelines.md#9-seo), [§14](01-guidelines.md#14-seo-pod-aillm-geo)).

**Wyjście etapu:** strona spełniająca Master checklist z [`01-guidelines.md` §19](01-guidelines.md#19-master-checklist-przed-przekazaniem-projektu).

## 8. Testy i własny przegląd kodu

- Responsywność: realne breakpointy 375/768/1024/1440px.
- Cross-browser: Chrome, Safari (iOS), Firefox.
- Formularze, linki, CTA - testowane ręcznie, nie tylko wizualnie.
- Własny code review: `console.log`, martwy kod, hardcody. **Zrób przerwę przed review** - świeże oczy łapią więcej.
- Poprawki wg priorytetu: blokujące > wizualne > kosmetyczne.

**Wyjście etapu:** lista poprawek posortowana wg priorytetu, zero błędów blokujących.

## 9. Wdrożenie produkcyjne

- Deploy: Cloudflare Pages ([§17](01-guidelines.md#17-przygotowanie-do-wdrożenia)). Hostinger/inny hosting tylko przy WordPressie.
- Domena, SSL, DNS - jeden wariant www/non-www wymuszony, spójnie z `trailingSlash` ([§2](01-guidelines.md#2-struktura-projektu)).
- Przekierowania 301: mapowanie starych URL 1:1, jeśli to redesign (`_redirects`, [§17](01-guidelines.md#17-przygotowanie-do-wdrożenia)).

**Wyjście etapu:** strona live na docelowej domenie, HTTPS wymuszony, przekierowania poprawne.

## 10. Po starcie

- Zgłoszenie sitemap w Google Search Console, indeksacja kluczowych podstron.
- Monitoring indeksacji przez pierwsze 2 tygodnie.
- GA4 lub Cloudflare Web Analytics ([§15](01-guidelines.md#15-prawo-rodo-eaa-cookies)): cele/eventy pod konwersje, połączenie z GSC.

**Wyjście etapu:** projekt zamknięty, klient przeszkolony z CMS, monitoring uruchomiony.
