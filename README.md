# 👾 INIT Method

Metodyka szybkiego i jakościowego tworzenia stron internetowych.

INIT Method to zestaw zasad, workflow i kontekstu projektowego, który pozwala powtarzalnie realizować projekty - od briefu klienta, przez planowanie i development, aż po wdrożenie i utrzymanie.

Projekt powstał na bazie doświadczenia, zmęczenia AI-slopem i zmęczenia powtarzalnymi procesami.

## Cel

Celem INIT Method jest ograniczenie pracy od zera przy każdym nowym projekcie oraz utrzymanie stałego poziomu jakości.

Nowy projekt powinien otrzymać gotowy fundament:
- standardy budowania stron,
- workflow realizacji projektu,
- brief klienta,
- kontekst projektu.

## Priorytety

Niezależnie od projektu, każda strona budowana w oparciu o INIT Method ma spełniać ten sam standard:
- **czysta architektura** - kod bez zbędnych abstrakcji, łatwy do utrzymania i rozbudowy,
- **SEO-friendly** - techniczne SEO, semantyka, dane strukturalne od pierwszego dnia, nie jako poprawka na końcu,
- **szybkie ładowanie i pełna optymalizacja** - Core Web Vitals jako twardy wymóg, nie sugestia,
- **UX-friendly** - intuicyjna nawigacja i interakcje, bez zbędnego tarcia,
- **accessibility-friendly** - zgodność z WCAG, strona dostępna niezależnie od sposobu korzystania z niej,
- **najnowsze techniki** - aktualne standardy i podejścia, bez trzymania się przestarzałych praktyk "bo zawsze tak było".

## Tech-stack

Projekt domyślnie przygotowany pod: **Astro**, **vanilla CSS**, **GSAP**, **Cloudflare**, **DataForSEO** - czyli mój stack, ale z powodzeniem można go zmienić pod siebie.

## Co zawiera projekt

| Plik | Opis |
|---|---|
| [`01-guidelines.md`](01-guidelines.md) | Zasady budowania stron i higiena projektowa - standardy kodu, dostępność, performance, SEO. |
| [`02-brief-template.md`](02-brief-template.md) | Szablon briefu klienta - jakie informacje zebrać przed startem projektu. |
| [`03-workflow.md`](03-workflow.md) | Workflow realizacji projektu - ścieżka produkcyjna krok po kroku. |
| [`04-context.md`](04-context.md) | Kontekst projektu - co musi znać developer i AI, żeby pracować spójnie ze stackiem i decyzjami. |

## Jak używać

1. Skopiuj cztery pliki `.md` do nowego projektu.
2. Wypełnij `02-brief-template.md` na podstawie rozmowy z klientem.
3. Uzupełnij `04-context.md` o specyfikę danego projektu (branding, treści, ograniczenia).
4. Prowadź projekt zgodnie z `03-workflow.md`, trzymając się zasad z `01-guidelines.md`.

*INIT Method nie jest gotowym szablonem strony. Jest systemem organizacji i realizacji projektu, który można wykorzystać jako bazę przy każdym kolejnym wdrożeniu.*