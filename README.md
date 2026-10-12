# Chemix Mózg – dokumentacja aplikacji (Allegro REST API)

Dokumentacja aplikacji integrującej się z Allegro REST API, zgłoszonej w Allegro Developer Portal przez firmę Chemix.

## Czym jest aplikacja

**Chemix Mózg** to wewnętrzny asystent analityczny firmy Chemix (sprzedawca na Allegro: konta `Chemix_sklep_pl`, `chemixshop`, `clean_house`). Służy wyłącznie właścicielom i pracownikom firmy do analizy własnej sprzedaży: porównania ofert z danymi magazynowymi i cenowymi z systemu ERP (Comarch ERP Optima), kontroli opłat i prowizji oraz raportowania.

Aplikacja:
- **działa wyłącznie w trybie odczytu** – nie tworzy, nie edytuje i nie zamyka ofert, nie zmienia zamówień, nie wysyła wiadomości do kupujących, nie wystawia ocen;
- **nie jest publiczna** – nie jest oferowana innym sprzedawcom ani kupującym;
- nie przetwarza danych w celach marketingowych i nie udostępnia ich podmiotom trzecim do ich własnych celów.

## Uprawnienia (scopes)

Aplikacja prosi wyłącznie o uprawnienia do odczytu:

| Uprawnienie | Po co |
|---|---|
| `allegro:api:sale:offers:read` | Odczyt własnych ofert (cena, stan, status, czas wysyłki) – porównanie ze stanami i cenami w ERP, wykrywanie ofert bez stanu, kontrola, czy towary zalegające w magazynie zostały wystawione |
| `allegro:api:orders:read` | Odczyt własnych zamówień – zestawienia sprzedaży, zwrotów i anulacji, uzgadnianie z dokumentami w ERP |
| `allegro:api:billing:read` | Odczyt opłat i prowizji na koncie – wyliczenie marży po prowizji Allegro, uzgadnianie faktur od Allegro |
| `allegro:api:payments:read` | Odczyt historii płatności – uzgadnianie wpływów z zamówieniami i dokumentami w ERP |
| `allegro:api:profile:read` | Identyfikacja, którego z kont firmy dotyczy autoryzacja (login konta) |

Aplikacja **nie korzysta** z żadnego uprawnienia typu `write` ani z uprawnień do wiadomości, ocen, sporów, licytacji i reklam.

## Do czego używamy Allegro REST API

| Zasób API | Uprawnienie | Po co |
|---|---|---|
| `GET /me` | `allegro:api:profile:read` | Rozpoznanie konta po autoryzacji |
| `GET /sale/offers` | `allegro:api:sale:offers:read` | Lista ofert sprzedawcy (status, cena, stan) |
| `GET /sale/product-offers/{offerId}` | `allegro:api:sale:offers:read` | Szczegóły oferty (EAN/produkt, cena, stan, czas wysyłki) do powiązania z towarem w ERP |
| `GET /order/checkout-forms` | `allegro:api:orders:read` | Zamówienia w zadanym okresie (pozycje, ilości, kwoty, status) |
| `GET /order/customer-returns` | `allegro:api:orders:read` | Zwroty – statystyki zwrotów per towar |
| `GET /billing/billing-entries`, `GET /billing/billing-types` | `allegro:api:billing:read` | Opłaty i prowizje per oferta/zamówienie |
| `GET /payments/payment-operations` | `allegro:api:payments:read` | Operacje płatnicze do uzgodnień |

## Autoryzacja

- OAuth 2.0 **Device Code Flow** (aplikacja działa na serwerze bez przeglądarki).
- Właściciel konta loguje się na allegro.pl i sam potwierdza kod urządzenia dla każdego z kont. Aplikacja nigdy nie zna hasła do konta.
- `client_id` i `client_secret` aplikacji przechowywane są w zmiennych środowiskowych serwera (nie w kodzie, nie w repozytorium).
- Tokeny dostępu i odświeżania przechowywane są na serwerze firmy w pliku poza repozytorium kodu, dostępnym tylko dla usługi aplikacji; tokeny są odświeżane automatycznie, a w razie potrzeby właściciel może w każdej chwili odwołać dostęp w ustawieniach konta Allegro.

## Dane i bezpieczeństwo

- **Minimalizacja danych osobowych:** do analiz wykorzystywane są dane handlowe (identyfikatory ofert i zamówień, produkty, ilości, kwoty, daty, statusy, opłaty). Dane osobowe kupujących (imię i nazwisko, adres, e-mail, telefon, login) **są odfiltrowywane przed analizą i nie są zapisywane** w raportach ani na dysku.
- **Asystent AI:** analizy i raporty przygotowuje asystent oparty na modelu językowym Claude (Anthropic) działający na serwerze firmy. Do modelu przekazywane są wyłącznie dane handlowe opisane wyżej – bez danych osobowych kupujących.
- **Przechowywanie:** zapisywane są wyłącznie raporty zbiorcze (np. sprzedaż per towar, opłaty per oferta) na potrzeby firmy. Surowe odpowiedzi API nie są archiwizowane; dane pośrednie starsze niż 90 dni są usuwane.
- **Odbiorcy:** wyniki trafiają wyłącznie do właścicieli i pracowników firmy Chemix (wewnętrzny komunikator firmowy). Dane nie są udostępniane podmiotom trzecim.
- **Limity API:** aplikacja wykonuje niewielką liczbę zapytań (analizy na żądanie i raporty okresowe), korzysta ze stronicowania i ograniczonej równoległości, respektuje kody 429 i nagłówki limitów (ponawianie z opóźnieniem), aby nie przekraczać limitów Allegro.
- **Zgodność:** aplikacja przestrzega Regulaminu REST API Allegro – korzysta z API wyłącznie w zakresie własnych kont sprzedawcy, nie omija limitów, nie pobiera danych innych sprzedawców i nie wykorzystuje danych do celów niezwiązanych z obsługą własnej sprzedaży.

## Technologia

- Serwer: Node.js (usługa w chmurze Railway), dostęp wyłącznie dla firmy.
- ERP: Comarch ERP Optima (MS SQL) – **tylko odczyt** (dokumenty handlowe, towary, stany, ceny).
- Komunikacja wyników: wewnętrzny serwer Discord firmy (dostęp tylko dla zatwierdzonych pracowników).

## Kontakt

Właściciel aplikacji: Chemix, e-mail: bartosz@chemix.sklep.pl

---

## English summary

**Chemix Mózg** is an internal, **read-only** analytics assistant used only by Chemix owners and employees (Allegro seller accounts `Chemix_sklep_pl`, `chemixshop`, `clean_house`). It reads the seller's own offers (`allegro:api:sale:offers:read`), orders and returns (`allegro:api:orders:read`), fees (`allegro:api:billing:read`), payment operations (`allegro:api:payments:read`) and the account login (`allegro:api:profile:read`) in order to compare them with stock and prices in the company ERP (Comarch ERP Optima), calculate margins after Allegro fees and prepare internal reports. It never creates or modifies offers or orders and requests no write scopes. Authorization uses OAuth 2.0 Device Code Flow; app secrets are kept in server environment variables and tokens in a file outside the code repository. Buyers' personal data is filtered out before analysis and is not stored; analyses are prepared by an AI assistant (Claude, Anthropic) that receives commercial data only. Results are shared only internally; no data is shared with third parties. API usage is low-volume and respects rate limits. Contact: bartosz@chemix.sklep.pl
