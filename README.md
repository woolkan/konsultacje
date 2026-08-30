# Konsultacje — założenia produktu i MVP

> Status: dokument roboczy  
> Data: 2026-08-30

## 1. Cel projektu

Celem projektu jest stworzenie aplikacji / landing page'a umożliwiającego klientowi:

1. zapoznanie się z ofertą konsultacji,
2. wybór dostępnego terminu,
3. tymczasową rezerwację wybranego terminu,
4. podanie danych potrzebnych do realizacji konsultacji,
5. opłacenie konsultacji online przez Stripe,
6. otrzymanie potwierdzenia rezerwacji i komunikacji e-mail,
7. otrzymanie informacji związanych z płatnością / dokumentem sprzedaży,
8. późniejsze zarządzanie komunikacją dotyczącą konsultacji.

Głównym KPI produktu jest **opłacona i potwierdzona rezerwacja konsultacji**.

---

## 2. Priorytet obecnego etapu

Na obecnym etapie nie budujemy finalnego, dopracowanego projektu graficznego.

Priorytety:

- funkcjonalność,
- UX,
- Mobile First,
- backend,
- model danych,
- logika dostępności terminów,
- integracja Stripe,
- komunikacja e-mail,
- bezpieczeństwo i odporność na edge case'y,
- zgodność z RODO i podstawowymi obowiązkami wobec polskich klientów.

Warstwa wizualna ma być celowo prosta i funkcjonalna.

### Kierunek wizualny MVP

- Mobile First,
- kolor główny: **navy blue**,
- biel,
- neutralne szarości,
- wysoki kontrast,
- prosta typografia,
- minimalna liczba dekoracji,
- brak rozbudowanych animacji,
- jasne CTA,
- pełne stany: loading / error / success / disabled / expired.

Docelowy branding i finalny layout będą projektowane później na bazie gotowego i sprawdzonego flow.

---

## 3. Główny flow użytkownika

Podstawowy lejek:

```text
Landing page
    ↓
Wybór terminu
    ↓
Tymczasowa blokada terminu
    ↓
Dane klienta
    ↓
Podsumowanie zamówienia
    ↓
Stripe
    ↓
Potwierdzenie płatności przez webhook
    ↓
Potwierdzenie rezerwacji
    ↓
E-mail + kalendarz / link do spotkania
```

Prosty model UX:

1. Oferta
2. Termin
3. Dane
4. Płatność
5. Potwierdzenie

---

## 4. Mobile First

Aplikacja ma być projektowana od początku pod urządzenia mobilne.

Desktop jest rozszerzeniem wersji mobilnej, a nie odwrotnie.

### Założenia UX

- brak klasycznego szerokiego kalendarza desktopowego pomniejszonego na telefon,
- czytelny wybór dnia,
- lista dostępnych godzin,
- duże touch targety,
- czytelne podsumowanie wybranego terminu,
- sticky CTA na dole ekranu tam, gdzie poprawia to konwersję,
- minimalna liczba kroków i pól,
- po wyborze terminu użytkownik od razu wie, na jak długo termin jest zablokowany.

Przykład:

```text
Wybierz termin

<   Wrzesień 2026   >

[Pon 7]
[Wt  8]
[Śr  9]
[Czw 10]
[Pią 11]

Dostępne godziny

09:00
10:30
12:00
14:00
16:30

────────────────────────

12 września · 14:00
60 minut · online

[ Zarezerwuj ten termin ]
```

---

## 5. Model rezerwacji terminu

Backend jest źródłem prawdy dla dostępności terminów.

Frontend nigdy nie może samodzielnie uznawać terminu za ostatecznie zarezerwowany.

### Podstawowe statusy

```text
available
    ↓
held
    ↓
confirmed
```

Dodatkowa ścieżka:

```text
held
    ↓
expired
    ↓
available
```

### Tymczasowa blokada terminu

Po wybraniu terminu slot powinien zostać tymczasowo zablokowany.

Założenie MVP:

- czas blokady: **15 minut**,
- blokada zapisywana po stronie backendu,
- blokada posiada `expires_at`,
- użytkownik widzi countdown,
- inny użytkownik nie może w tym czasie kupić tego samego slotu,
- po wygaśnięciu blokady termin wraca do puli dostępnych terminów,
- rezerwacja staje się ostateczna dopiero po potwierdzeniu płatności.

Przykład komunikatu:

```text
Termin jest zarezerwowany dla Ciebie przez 14:32
```

### Ważne

Mechanizm musi być odporny na:

- dwa równoczesne żądania dotyczące tego samego terminu,
- ponowione webhooki,
- przerwane checkouty,
- zamknięcie przeglądarki po dokonaniu płatności,
- opóźnienia w komunikacji ze Stripe.

---

## 6. Stripe

Stripe odpowiada za płatność.

### Zasady

- nie uznajemy płatności wyłącznie na podstawie przekierowania użytkownika z checkoutu,
- źródłem prawdy o płatności jest zweryfikowany webhook Stripe,
- webhook musi być idempotentny,
- dopiero potwierdzona płatność zmienia rezerwację na `confirmed`.

Docelowy flow:

```text
held booking
    ↓
Stripe Checkout / Payment
    ↓
Stripe webhook
    ↓
verify event
    ↓
payment confirmed
    ↓
booking.confirmed
```

### Dokument sprzedaży / potwierdzenie płatności

Na MVP preferowane jest wykorzystanie funkcjonalności Stripe do wysyłki dokumentów / potwierdzeń płatności, zamiast budowania własnego systemu fakturowania od zera.

Szczegółowy model faktur i kwestie księgowe wymagają jeszcze doprecyzowania.

---

## 7. Komunikacja e-mail

Mailing transakcyjny należy oddzielić od marketingowego.

### Transakcyjne e-maile aplikacji

Preferowany provider: **MailerSend**.

MailerSend będzie wykorzystywany przez backend przez API.

Przykładowe eventy:

| Event | Wiadomość | MVP |
|---|---|---:|
| `booking.confirmed` | potwierdzenie konsultacji | tak |
| `booking.reminder_24h` | przypomnienie 24 h przed | tak |
| `booking.reminder_1h` | przypomnienie przed spotkaniem | tak / do decyzji |
| `booking.rescheduled` | potwierdzenie zmiany terminu | tak |
| `booking.cancelled` | potwierdzenie anulowania | tak |
| `payment.failed` | informacja o problemie z płatnością | tak |
| `payment.refunded` | informacja o zwrocie | tak |
| `booking.completed` | podziękowanie / follow-up | później |
| `booking.no_show` | informacja po nieobecności | później |
| `checkout.abandoned` | odzyskanie niedokończonego zakupu | później |

Nie wysyłamy e-maila w momencie samego utworzenia krótkiego `hold`, ponieważ użytkownik pozostaje wtedy aktywny w flow checkoutu.

### Mailing marketingowy

Marketing nie powinien znajdować się na krytycznej ścieżce zakupu.

Potencjalne rozwiązanie później:

- MailerLite,
- newsletter,
- automatyzacje marketingowe,
- kampanie,
- follow-up sprzedażowy.

Dane klienta mogą trafić do systemu marketingowego wyłącznie zgodnie z odpowiednią podstawą prawną i po uzyskaniu wymaganej zgody.

---

## 8. Architektura wysyłki e-mail

Wysłanie wiadomości nie może blokować finalizacji rezerwacji.

Nie:

```text
Stripe webhook
    ↓
sendEmail()
    ↓
confirm booking
```

Preferowane:

```text
Stripe webhook
    ↓
verify payment
    ↓
confirm booking
    ↓
create booking.confirmed event
    ↓
email worker / job
    ↓
MailerSend
```

Dzięki temu awaria MailerSend nie wpływa na stan opłaconej rezerwacji.

### Minimalny model e-mail events

Przykład:

```text
email_events

id
type
booking_id
recipient
status
provider_message_id
attempts
scheduled_at
sent_at
created_at
```

Statusy:

```text
pending
processing
sent
failed
cancelled
```

Mechanizm powinien wspierać:

- retry,
- idempotencję,
- historię wysyłki,
- możliwość diagnostyki problemów.

---

## 9. RODO i prywatność

Projekt jest kierowany przede wszystkim do klientów w Polsce, dlatego należy uwzględnić RODO oraz polskie przepisy dotyczące usług świadczonych online i komunikacji elektronicznej.

### Podstawowe założenia

MailerSend może być używany jako procesor danych w rozwiązaniu zgodnym z RODO, przy odpowiednim skonfigurowaniu całego procesu.

W projekcie należy stosować zasadę:

> privacy by design i minimalizacja danych.

### Nie stosujemy checkboxa „zgoda na RODO”

Dane potrzebne do:

- rezerwacji,
- realizacji konsultacji,
- obsługi płatności,
- potwierdzenia terminu,
- przypomnień związanych z realizacją usługi

nie powinny być uzależniane od dodatkowej „zgody na RODO”, jeśli podstawą przetwarzania jest zawarcie lub wykonanie umowy.

### Marketing

Zgoda marketingowa musi być:

- oddzielna,
- dobrowolna,
- niezaznaczona domyślnie,
- niezależna od możliwości zakupu konsultacji.

Mail transakcyjny nie powinien być wykorzystywany do obchodzenia wymagań dotyczących marketingu.

---

## 10. Dokumenty prawne

Przed produkcyjnym uruchomieniem aplikacji potrzebujemy co najmniej:

### Regulamin konsultacji / serwisu

Powinien określać m.in.:

- dane usługodawcy,
- zakres konsultacji,
- czas konsultacji,
- cenę,
- sposób płatności,
- sposób zawarcia umowy,
- rezerwację terminu,
- tymczasową blokadę terminu,
- zmianę terminu,
- anulowanie,
- spóźnienie,
- no-show,
- zwroty,
- reklamacje,
- wymagania techniczne,
- zasady dla konsumentów,
- prawo odstąpienia od umowy.

### Polityka prywatności

Powinna określać m.in.:

- administratora danych,
- zakres zbieranych danych,
- cele przetwarzania,
- podstawy prawne,
- czas przechowywania,
- procesorów / odbiorców danych,
- transfery poza EOG,
- prawa osób, których dane dotyczą,
- dane związane z MailerSend,
- Stripe,
- hostingiem,
- kalendarzem / wideokonferencją,
- ewentualnym MailerLite.

### Cookies

Na MVP preferowane jest ograniczenie technologii do niezbędnych.

Na pierwszym etapie unikamy, jeśli nie są potrzebne:

- Meta Pixel,
- Google Ads,
- Hotjar,
- rozbudowanego marketing trackingu.

Po dodaniu narzędzi analitycznych / marketingowych wymagających zgody należy wdrożyć odpowiedni mechanizm zarządzania zgodami.

---

## 11. Klient B2C / prawa konsumenta

Jeżeli konsultację może kupić osoba będąca konsumentem, checkout i regulamin muszą uwzględniać odpowiednie prawa konsumenta.

Do szczegółowego opracowania pozostają m.in.:

- 14-dniowe prawo odstąpienia,
- konsultacje realizowane przed upływem 14 dni,
- żądanie wcześniejszego rozpoczęcia realizacji usługi,
- skutki pełnego wykonania usługi,
- sposób anulowania,
- zwroty.

### Finalny ekran przed płatnością

Użytkownik powinien jasno widzieć:

```text
Podsumowanie

Konsultacja indywidualna online
12 września 2026
14:00–15:00

Cena: 499,00 zł brutto

[ wymagane oświadczenia ]

[ Rezerwuję i płacę 499 zł ]
```

Finalne CTA powinno jednoznacznie komunikować obowiązek zapłaty.

---

## 12. Formularz klienta

Minimalizujemy zakres danych.

Przykład MVP:

```text
Imię
E-mail

[dane dodatkowe wymagane do płatności / faktury]

☐ Akceptuję Regulamin.

Potwierdzam zapoznanie się z Polityką prywatności.

☐ [jeśli ma zastosowanie]
Żądam rozpoczęcia świadczenia przed upływem
terminu na odstąpienie i przyjmuję do wiadomości
związane z tym konsekwencje.

☐ Chcę otrzymywać newsletter i informacje o ofertach.
  (opcjonalne)
```

Dokładna treść oświadczeń prawnych powinna zostać zweryfikowana przed uruchomieniem produkcyjnym.

---

## 13. Wstępna architektura systemu

```text
                    ┌────────────────────┐
                    │   Mobile Web App   │
                    │    Landing + UX    │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │      Backend       │
                    │                    │
                    │ availability       │
                    │ holds              │
                    │ bookings           │
                    │ payments           │
                    │ events             │
                    └───┬──────┬─────┬───┘
                        │      │     │
             ┌──────────┘      │     └──────────┐
             ▼                 ▼                ▼
          Stripe          MailerSend       Calendar /
          payment          transaction      meeting
          webhooks         e-mails          provider
```

W przyszłości:

```text
booking / marketing consent
          ↓
      MailerLite
          ↓
newsletter / marketing automation
```

---

## 14. Wstępne encje backendowe

Do dalszego opracowania:

### `slots`

- `id`
- `starts_at`
- `ends_at`
- `status`

### `booking_holds`

- `id`
- `slot_id`
- `token`
- `expires_at`
- `created_at`

### `bookings`

- `id`
- `slot_id`
- `customer_id`
- `status`
- `price`
- `currency`
- `created_at`
- `confirmed_at`
- `cancelled_at`

### `customers`

- `id`
- `name`
- `email`
- minimalne pozostałe dane wymagane przez proces

### `payments`

- `id`
- `booking_id`
- `provider`
- `provider_payment_id`
- `status`
- `amount`
- `currency`
- `created_at`
- `confirmed_at`

### `email_events`

- `id`
- `booking_id`
- `type`
- `recipient`
- `status`
- `attempts`
- `provider_message_id`
- `scheduled_at`
- `sent_at`

Dokładny model bazy będzie ustalany przy wyborze stacku i implementacji backendu.

---

## 15. Najważniejsze zasady techniczne

1. Backend jest źródłem prawdy dla dostępności.
2. Slot jest blokowany atomowo.
3. Rezerwacja jest potwierdzana dopiero po potwierdzonej płatności.
4. Webhooki są idempotentne.
5. Wysyłka e-mail jest asynchroniczna względem finalizacji rezerwacji.
6. Awaria mailingu nie może cofnąć poprawnej płatności.
7. Frontend nie przechowuje krytycznego stanu jako jedynego źródła prawdy.
8. Dane osobowe są minimalizowane.
9. Marketing jest odseparowany od komunikacji transakcyjnej.
10. UX jest projektowany Mobile First.
11. Finalny design jest osobnym późniejszym etapem.
12. Logika prawna i biznesowa powinna być spójna z backendem.

---

## 16. Decyzje podjęte

- aplikacja Mobile First,
- prosty interfejs MVP w tonacji navy blue,
- najpierw UX / funkcjonalność / backend, później finalny design,
- Stripe jako provider płatności,
- tymczasowa blokada slotu przed płatnością,
- założenie blokady: 15 minut,
- MailerSend jako preferowany provider e-maili transakcyjnych,
- oddzielenie mailingu transakcyjnego od marketingowego,
- MailerLite rozważany później jako system marketingowy,
- brak „zgody na RODO” dla danych niezbędnych do realizacji umowy,
- przygotowanie Regulaminu i Polityki prywatności przed produkcyjnym startem,
- ograniczenie trackingu w MVP.

---

## 17. Otwarte decyzje

Do ustalenia przed / w trakcie implementacji:

### Produkt

- długość konsultacji,
- cena,
- waluta,
- czy istnieje jeden czy kilka typów konsultacji,
- minimalne wyprzedzenie rezerwacji,
- maksymalne wyprzedzenie rezerwacji,
- zasady zmiany terminu,
- limit zmian terminu,
- zasady anulowania,
- zasady zwrotów,
- zasady spóźnienia,
- zasady no-show.

### Kalendarz

- źródło dostępności,
- integracja z Google Calendar / innym systemem,
- tworzenie wydarzenia,
- automatyczne generowanie linku do spotkania,
- obsługa stref czasowych.

### Płatności i dokumenty

- Stripe Checkout vs własny Payment Element,
- faktury vs receipts,
- dane wymagane do faktury,
- B2C / B2B,
- obsługa VAT,
- zwroty pełne i częściowe.

### Mailing

- ostateczna lista eventów,
- reminder 1 h przed konsultacją,
- abandoned checkout,
- szablony,
- domena wysyłkowa,
- SPF / DKIM / DMARC.

### Technologia

- frontend framework,
- backend,
- baza danych,
- hosting,
- scheduler / jobs,
- observability,
- system logów,
- środowiska staging / production.

### Prawo

- finalny regulamin,
- finalna polityka prywatności,
- cookies,
- prawo odstąpienia,
- treść checkboxów / oświadczeń,
- retencja danych,
- zasady marketingu.

---

## 18. Kolejność implementacji MVP

Proponowana kolejność:

1. ustalenie stacku,
2. model danych,
3. system slotów i dostępności,
4. mechanizm tymczasowego `hold`,
5. mobilny flow wyboru terminu,
6. formularz klienta,
7. podsumowanie rezerwacji,
8. Stripe,
9. webhooki i finalizacja rezerwacji,
10. MailerSend,
11. przypomnienia,
12. kalendarz / link do spotkania,
13. ekrany błędów i edge case'y,
14. dokumenty prawne i finalny checkout compliance,
15. testy E2E,
16. dopiero później finalny design / branding.

---

## 19. Definicja MVP

MVP jest gotowe, kiedy klient na telefonie może:

1. wejść na landing page,
2. zobaczyć podstawową ofertę,
3. wybrać realnie dostępny termin,
4. zablokować go na czas checkoutu,
5. podać minimalne wymagane dane,
6. zobaczyć jasne podsumowanie ceny i terminu,
7. zapłacić przez Stripe,
8. mieć rezerwację potwierdzoną na podstawie webhooka,
9. dostać e-mail z potwierdzeniem,
10. dostać późniejsze przypomnienie,
11. nie dopuścić do podwójnej sprzedaży tego samego slotu,
12. przejść poprawnie przez flow także przy błędach płatności lub wygaśnięciu blokady.

---

## 20. Uwagi

Ten dokument opisuje wymagania produktowe i techniczne oraz robocze założenia compliance. Nie stanowi porady prawnej.

Treści Regulaminu, Polityki prywatności oraz oświadczeń konsumenckich powinny przed wdrożeniem produkcyjnym zostać zweryfikowane pod kątem konkretnego modelu działalności i aktualnego prawa.
