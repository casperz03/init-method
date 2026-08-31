# 05 · Pułapki

Krótkie, jednozdaniowe przypomnienia o nieoczywistych zachowaniach, które kosztowały czas na prawdziwym projekcie. Ten plik jest do trzymania przed oczami przez cały projekt, nie do przeczytania raz przy starcie - wklej go w całości do `CLAUDE.md`/`AGENTS.md` nowego projektu, obok `04-context.md`. Szczegóły i uzasadnienie każdego punktu: [`01-guidelines.md`](01-guidelines.md), sekcja podana w nawiasie.

1. `_redirects` na Cloudflare Pages nie ma flagi force (`!`) z Netlify i nie obsługuje przekierowań domenowych (apex ↔ www) - użyj Redirect Rules w dashboardzie. ([§17](01-guidelines.md#17-przygotowanie-do-wdrożenia))
2. `_headers` na Cloudflare Pages nie nadpisuje pasujących reguł, tylko je łączy (comma-join) - nakładające się wzorce dla tego samego nagłówka dają bezsensowną wartość. ([§17](01-guidelines.md#17-przygotowanie-do-wdrożenia))
3. GSAP `ScrollTrigger` wymaga jawnego `gsap.registerPlugin(ScrollTrigger)` przed użyciem - bez tego cicho nic się nie animuje, bez błędu w konsoli. ([§6](01-guidelines.md#6-javascript---vanilla--gsap))
4. Przy Astro View Transitions (`<ClientRouter />`) inicjalizuj GSAP/ScrollTrigger na evencie `astro:page-load`, nie `DOMContentLoaded` - ten drugi odpala się tylko raz. ([§6](01-guidelines.md#6-javascript---vanilla--gsap))
5. `SEOHead.astro`: `canonical` musi być warunkowy (`{!noindex && ...}`) - inaczej strona z `noindex` (np. 404) dostaje sprzeczne sygnały canonical + noindex jednocześnie. ([§9](01-guidelines.md#9-seo))
6. `trailingSlash: 'always'` i `@astrojs/sitemap` mogą się cicho rozjechać z self-canonical dla strony głównej - sprawdź ręcznie po pierwszym buildzie. ([§2](01-guidelines.md#2-struktura-projektu))
7. CSP `script-src 'self'` blokuje małe, bezimportowe chunki JS, które Vite domyślnie inline'uje w HTML - wyłącz inlining plików `.js` przez `assetsInlineLimit` w `astro.config.mjs`. ([§12](01-guidelines.md#12-bezpieczeństwo))
8. `Google-Extended` w robots.txt blokuje jednocześnie trening AI **i** grounding/cytowania Gemini - nie da się tego rozdzielić tak jak przy `GPTBot`/`OAI-SearchBot`. ([§14](01-guidelines.md#14-seo-pod-aillm-geo))
9. `Applebot-Extended` ≠ `Applebot` - zablokowanie pierwszego (trening) nie wycina drugiego (Siri, Spotlight). ([§14](01-guidelines.md#14-seo-pod-aillm-geo))
10. Cloudflare Web Analytics nie mierzy zdarzeń własnych (`mailto:`, `tel:`, wysłanie formularza) - tylko ruch. Do konwersji potrzebne coś więcej. ([§15](01-guidelines.md#15-prawo-rodo-eaa-cookies))
11. `git-gateway` w Decap CMS wymaga usługi Netlify Identity - w stacku czysto-Cloudflare użyj backendu `github` + własny OAuth-Worker. ([§16](01-guidelines.md#16-decap-cms))
