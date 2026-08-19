# 02 · Brief Template

Wypełniany w [etapie 1 workflow](03-workflow.md#1-discovery-i-briefing) - podczas wywiadu z klientem, nie po nim z pamięci. Im więcej konkretu tutaj, tym mniej rund poprawek na etapie designu i mniej pytań "od nowa" w trakcie developmentu.

Po wypełnieniu: dane biznesowe/prawne zostają w tym pliku, dane techniczne/brandingowe przepisz do [`04-context.md`](04-context.md) - to on jest kontekstem ładowanym do AI podczas developmentu, nie ten brief.

---

## 1. Dane podstawowe

| Pole | Wartość |
|---|---|
| Nazwa firmy | |
| Branża / obszar działania | |
| Forma prawna, NIP | |
| Adres (do stopki, schema `LocalBusiness`, GBP - musi być **identyczny** wszędzie, patrz `01-guidelines.md` §10) | |
| Telefon, e-mail kontaktowy | |
| Godziny działania | |
| Osoba decyzyjna po stronie klienta (kto akceptuje etapy?) | |
| Kanał komunikacji roboczej (mail / Slack / WhatsApp itd.) | |

## 2. Biznes i cele

- **Model biznesowy** - co i komu klient sprzedaje, jak dziś pozyskuje klientów.
- **USP** - czym klient różni się od konkurencji (w jego własnych słowach, nie w Twoich domysłach).
- **Grupa docelowa** - kto ma wejść na stronę: demografia, potrzeby, etap decyzji zakupowej. Jeśli więcej niż jedna grupa - priorytet.
- **Cel strony** - leady telefoniczne? formularz? sprzedaż online? wizerunek/portfolio? rezerwacje?
- **Mierzalne KPI** - konkretna liczba/wskaźnik, nie "więcej klientów". Np. "10 formularzy/miesiąc", "wzrost pozycji na 5 fraz do top 10".
- **Budżet i harmonogram** - twardy deadline (event, sezon, kampania)? Jeśli tak - od niego licz wstecz etapy z `03-workflow.md`.

## 3. Konkurencja i inspiracje

| Typ | Adres / nazwa | Co się podoba / nie podoba |
|---|---|---|
| Konkurent lokalny 1 | | |
| Konkurent lokalny 2 | | |
| Konkurent lokalny 3 | | |
| Strona wzorcowa 1 (niekoniecznie branża) | | |
| Strona wzorcowa 2 | | |

**Dla AI dostarczyć:** zrzuty ekranu stron, które klientowi się podobają, i mood board, jeśli istnieje - patrz `03-workflow.md` etap 4.

## 4. Zakres projektu

- [ ] Nowa strona / [ ] Redesign istniejącej (jeśli redesign: adres obecnej strony → wymagany audyt techniczny + mapowanie 301, `01-guidelines.md` §17)
- Liczba i lista podstron (np. Home, Usługi, O nas, Cennik, Kontakt, Blog):
  - …
- Funkcjonalności poza standardową wizytówką:
  - [ ] Formularz kontaktowy - pola: …
  - [ ] Rezerwacja/kalendarz online
  - [ ] Blog / aktualności
  - [ ] Sklep / płatności online
  - [ ] Wielojęzyczność - języki: …
  - [ ] Panel klienta / logowanie
  - [ ] Integracja z zewnętrznym systemem (CRM, ERP, booking) - jaki: …
- **Zakres CMS** - które sekcje/treści klient chce edytować samodzielnie po starcie (wpływa na model `content/` w Decap, `01-guidelines.md` §16)?

## 5. Treść i branding

- **Źródło treści** - klient dostarcza gotowe teksty / potrzebny copywriting od zera / częściowo.
- **Ton komunikacji** - formalny / swobodny / ekspercki / przyjazny. Przykład zdania, które brzmi "jak oni".
- **Materiały istniejące:**
  - [ ] Logo (wektor: SVG/AI/EPS)
  - [ ] Brand book / księga znaku
  - [ ] Paleta kolorów (kody HEX, jeśli ustalone)
  - [ ] Fonty (nazwy, licencje)
  - [ ] Zdjęcia własne (jakość/rozdzielczość wystarczająca do web?)
  - [ ] Zdjęcia do zakupu/wygenerowania - budżet na stocki?
- **Czego unikać** - kolory/style/elementy, których klient wprost nie chce.

## 6. SEO i widoczność

- Obecna strona (jeśli istnieje): pozycje na kluczowe frazy, ruch organiczny, dane z GSC - dostęp do konta?
- Frazy priorytetowe (wstępnie, doprecyzowane w etapie 2 workflow przez DataForSEO).
- Google Business Profile: istnieje i zweryfikowany? / trzeba założyć? - dostęp do konta.
- Obecność w innych katalogach (Panorama Firm, branżowe) - do weryfikacji spójności NAP.
- Czy klient chce, by treść trafiała do trenowania modeli AI (`GPTBot`, `ClaudeBot`) - wpływa na `robots.txt`, `01-guidelines.md` §14.

## 7. Techniczne

- **Domena:** istnieje / trzeba zarejestrować. Rejestrator, dostęp do panelu DNS.
- **Hosting/CMS obecny** (jeśli redesign): co i gdzie, czy będzie wygaszany.
- **Docelowy hosting:** Cloudflare Pages (domyślnie, `01-guidelines.md`) - potwierdź brak przeciwwskazań (np. wymóg klienta co do konkretnego dostawcy).
- Istniejące konta: Google Analytics / GA4, Google Search Console, Meta Pixel - dostępy czy zakładamy nowe.
- Integracje zewnętrzne wymagające kluczy API (płatności, mapy, CRM) - kto je dostarcza i kiedy.

## 8. Prawne

- Czy strona zbiera dane osobowe (formularz, newsletter, konto)? → wymagana polityka prywatności, `01-guidelines.md` §15.
- Czy klient sprzedaje usługę/produkt online? → wymagany regulamin.
- Liczba zatrudnionych i orientacyjny obrót/suma bilansowa → kwalifikacja pod EAA (mikroprzedsiębiorstwo <10 osób i <2 mln EUR = zwolnienie, ale zaznacz to informacyjnie, nie jako gwarancję prawną).
- Czy klient ma już politykę prywatności/regulamin do zaadaptowania, czy trzeba przygotować od zera (rekomendacja: konsultacja z prawnikiem klienta).

## 9. Design i UX

- Styl wizualny: minimalistyczny / korporacyjny / kreatywny / lokalny-swojski itd.
- Referencje wizualne (linki, patrz sekcja 3).
- Wymagania a11y ponad standard WCAG AA (jeśli klient/branża tego wymaga - np. sektor publiczny).
- Animacje: oczekiwany poziom (statyczna wizytówka vs. rozbudowane scroll-animacje GSAP) - wpływa na wycenę i czas developmentu.

## 10. Po starcie

- Kto zarządza treścią po starcie - klient samodzielnie (CMS) czy zlecane wykonawcy?
- Potrzebne szkolenie z CMS - dla ilu osób, w jakiej formie (call, nagranie, instrukcja pisemna)?
- Plan rozwoju strony (blog, sklep, kolejne języki) - wpływa na architekturę już teraz, żeby nie przerabiać struktury później.
- Kto po starcie monitoruje analitykę/pozycje - klient, agencja, nikt (ustal oczekiwania wprost)?

---

**Status briefu:** ☐ szkic · ☐ w trakcie wywiadu · ☐ zaakceptowany przez klienta · ☐ przeniesiony do `04-context.md`
