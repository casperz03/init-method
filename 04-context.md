# 04 · Project Context

Ten plik jedzie razem z projektem - wklej go (obok `01-guidelines.md`) jako kontekst dla AI/dewelopera w konkretnym repo klienckim. W przeciwieństwie do `01-guidelines.md` (zasady stałe) i `02-brief-template.md` (jednorazowy wywiad), ten plik **żyje przez cały projekt** - aktualizuj go na bieżąco, w miarę podejmowania decyzji, nie tylko raz na starcie.

---

## Projekt

| Pole | Wartość |
|---|---|
| Nazwa klienta | |
| Domena docelowa | |
| Repo | |
| Status | ☐ w budowie · ☐ live · ☐ w utrzymaniu |
| Ostatnia aktualizacja tego pliku | |

## 1. Stack projektu

Domyślnie: Astro (static) + vanilla CSS + GSAP + Decap CMS + Cloudflare Pages, zgodnie z `01-guidelines.md`.

- [ ] Bez odstępstw od domyślnego stacku.
- [ ] Odstępstwa (opisz co i dlaczego - patrz sekcja 5):

## 2. Branding

- **Kolory** (wartości do `:root` w `global.css`, `01-guidelines.md` §5):

```css
--color-primary: #______;
--color-text: #______;
--color-bg: #______;
```

- **Fonty:** (nazwa, źródło - self-hosted zgodnie z §8)
- **Logo:** (lokalizacja pliku w repo)
- **Ton komunikacji:** (z `02-brief-template.md` §5 - jedno zdanie-przykład "jak oni")

## 3. Treść i struktura

- Rzeczywista lista podstron (może różnić się od pierwotnego briefu):
  - …
- Collections w Decap CMS (`public/admin/config.yml`) i odpowiadające im schematy w `src/content/config.ts` - trzymaj oba w sync, patrz `01-guidelines.md` §16.
- Język(i) treści.

## 4. Dostępy i zasoby

Same linki/nazwy - **żadnych haseł/kluczy w tym pliku**, te żyją w zmiennych środowiskowych Cloudflare (`01-guidelines.md` §12, §17).

| Zasób | Link/identyfikator |
|---|---|
| Repo GitHub | |
| Projekt Cloudflare Pages | |
| OAuth Worker (Decap) | |
| Panel `/admin` | |
| Google Search Console | |
| GA4 / Cloudflare Web Analytics | |
| Google Business Profile | |

## 5. Odstępstwa od `01-guidelines.md`

Każde odstępstwo od domyślnych zasad - wpisz **co** i **dlaczego**, żeby za pół roku (Ty albo ktoś inny) nie zgadywał, czy to błąd, czy świadoma decyzja.

| Sekcja `01-guidelines.md` | Odstępstwo | Powód |
|---|---|---|
| np. §13 (SSG domyślnie) | SSR (`@astrojs/cloudflare`) | Panel klienta z danymi per-sesja |
| | | |

## 6. Kontakty

| Rola | Osoba | Kontakt |
|---|---|---|
| Decyzyjna po stronie klienta | | |
| Wykonawca | | |
| Kto edytuje treść po starcie | | |

## 7. Log decyzji

Krótki, chronologiczny zapis - nie protokół, tylko decyzje, które warto pamiętać.

| Data | Decyzja | Powód |
|---|---|---|
| | | |
