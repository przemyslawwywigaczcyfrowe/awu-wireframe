# Obecne narzędzia i co je zastąpi — AWU

> Mapa wszystkich narzędzi używanych dziś w procesie skupu produktów używanych,
> ich problemy oraz co AWU ma zmienić.

---

## Przegląd: jak wygląda praca dziś

Proces skupu opiera się na **6+ arkuszach Google Sheets**, systemie ticketowym OTRS/Thulium, systemie ERP Verto i kilku dodatkowych narzędziach. Każdy z tych systemów działa osobno — dane są kopiowane ręcznie między nimi.

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Google Sheets │────▶│    OTRS      │────▶│    Verto     │
│ (6+ arkuszy) │     │  (tickety)   │     │   (ERP)      │
└──────┬───────┘     └──────────────┘     └──────┬───────┘
       │                                         │
       │         ┌──────────────┐                 │
       └────────▶│  Cyfrolog    │◀────────────────┘
                 │ (numery CYF) │
                 └──────┬───────┘
                        │
              ┌─────────▼────────┐
              │     Sylius       │───▶ Baselinker ───▶ Allegro
              │  (sklep online)  │
              └──────────────────┘
```

---

## 1. Google Sheets — serce obecnego procesu

### Arkusz 1: Lista umów (master)
**Do czego służy:** Centralny rejestr wszystkich umów. Kiedy AWU generuje umowę, CSV wpada mailem i jest wpisywany do tego arkusza.

**Co zawiera:** Numer umowy, data wygenerowania, numer wyceny, salon ekspertyzy, ilość produktów, dane klienta, status.

**Problemy:**
- Ciągłe przewijanie w prawo (dużo kolumn)
- Brak walidacji pól — ludzie zapominają wpisać dane lub wpisują źle
- Arkusz się crashuje przy dużej ilości danych
- Jeden wspólny plik — wszyscy edytują jednocześnie
- Brak wymaganych pól — nie ma mechanizmu, który wymusi uzupełnienie

**Co zastąpi AWU:** → Moduł listy wycen i umów z walidacją, wymaganymi polami i automatycznym importem danych

---

### Arkusz 2: Produkty po ekspertyzie
**Do czego służy:** Lista wszystkich produktów z umów z cenami odkupu i sprzedaży.

**Co zawiera:** Indeks produktu, cena odkupu, cena sprzedaży, marża, stan, akcesoria.

**Problemy:**
- CSV z AWU nie zawiera ceny sprzedaży — trzeba ją ustalić osobno
- Cennik produktów to oddzielny arkusz, trzeba ręcznie sprawdzać
- Akcesoria wpływają na cenę, ale obliczenia są ręczne

**Co zastąpi AWU:** → Moduł produktów z automatycznym cennikiem i kalkulacją marży

---

### Arkusz 3: Kanban przygotowania do sprzedaży
**Do czego służy:** Śledzenie, na jakim etapie jest każdy produkt (od „Nowy" do „Allegro").

**Co zawiera:** 10 kolumn/zakładek odpowiadających krokom Kanbanu. Każda osoba widzi tylko swoją zakładkę.

**Problemy:**
- Brak blokerów — system nie zatrzymuje postępu, jeśli warunek nie jest spełniony (np. brak indeksu)
- Ręczne przenoszenie między zakładkami
- Nie widać zależności (np. że PZ i karta produktu muszą być gotowe jednocześnie)
- Zakupy na fakturę (B2B) nie generują CSV — trzeba je wpisywać ręcznie

**Co zastąpi AWU:** → Moduł Kanban z drag & drop, wizualnymi blokerami i automatycznym importem

---

### Arkusz 4: Cennik odkupu
**Do czego służy:** Pracownicy salonów sprawdzają tutaj, ile mogą zaproponować klientowi za dany produkt.

**Problemy:**
- Papierowe wydruki w salonach — nieaktualne
- Pracownik musi ręcznie szukać produktu w Excelu
- Wolne, podatne na błędy
- **To jest główna bolączka operacyjna**

**Co zastąpi AWU:** → Moduł szybkiej wyceny salonowej z wyszukiwaniem produktów i natychmiastową ceną

---

### Arkusz 5: Rozliczenie PCC
**Do czego służy:** Obliczanie podatku PCC od umów > 1000 zł.

**Co zawiera:** Lista umów, kwota z umowy, kwota z Verto, czy podlega PCC, miesiąc rozliczenia.

**Problemy:**
- Cena na umowie może różnić się od ceny w Verto
- Dane z Verto importowane raz na tydzień — opóźnienia
- Konieczność ręcznego sprawdzania rozbieżności

**Co zastąpi AWU:** → Moduł PCC z automatycznym porównaniem kwot i eksportem raportów

---

### Arkusz 6: Plik serwisowy
**Do czego służy:** Śledzenie produktów w serwisie/naprawie.

**Problemy:**
- RMA w Verto nie są powiązane z działem produktów używanych
- Trudno wyliczyć sumaryczne koszty serwisowe
- Brak widoczności ukrytych kosztów w raportach marżowości

**Co zastąpi AWU:** → Moduł serwisowy z trackingiem kosztów i powiązaniem z produktami

---

## 2. OTRS / Thulium — komunikacja z klientem

**Do czego służy:** Centralny system komunikacji mailowej i SMS z klientami.

**Kolejki:** AWU (automatyczne), Indywidualna, Sprzęt w drodze, Przypomnienia, Odkupione salon, Serwis, Umowa e-podpis (restricted)

**Problemy:**
- Codzienne ręczne łączenie ticketów (wycena + umowa + podpis = jeden klient)
- Wyszukiwanie historii cenowej w korespondencji jest bardzo czasochłonne
- Dane osobowe (PESEL) w systemie ticketowym — problem bezpieczeństwa
- Każdy „zapisz" w ekspertyzie generuje nowy mail/CSV — ryzyko wysłania niekompletnych danych

**Co zastąpi AWU:** → Moduł komunikacji z integracją Thulium + automatyczne łączenie wątków

---

## 3. Verto (ERP)

**Do czego służy:** System ERP — źródło prawdy o cenach, stanach magazynowych, dokumentach zakupu.

**Kluczowe funkcje:**
- Indeksy produktów (tworzenie, edycja)
- Dokumenty zakupu PZ (seria 03 = używane)
- Ceny finalne (źródło prawdy, nie umowa!)
- Stany magazynowe i MMK (przesunięcia)
- RMA (zlecenia serwisowe)

**Problemy:**
- Import danych do Sheets raz na tydzień — opóźnienia
- Tworzenie indeksu to 4-6 ręcznych zmian na istniejącym indeksie
- PZ wymaga fizycznej umowy w centrali — opóźnienia z salonów
- RMA nie powiązane z działem używanych

**Co zastąpi AWU:** AWU nie zastąpi Verto — **zintegruje się z nim**. AWU będzie czytać ceny, tworzyć indeksy i PZ przez API.

---

## 4. Cyfrolog

**Do czego służy:** Nadawanie unikalnych numerów CYF i kodów EAN produktom.

**Problemy:**
- Każdy produkt musi być wprowadzony pojedynczo
- Bulk processing został wyceniony na zbyt wysoką kwotę 2.5 roku temu

**Co zastąpi AWU:** → Integracja z Cyfrolog API (jeśli dostępne) lub wbudowany moduł numeracji

---

## 5. Sylius (sklep internetowy)

**Do czego służy:** Platforma e-commerce — karty produktów na cyfrowe.pl.

**Problemy:**
- Indeks z Verto tworzy pustą kartę — wszystko trzeba uzupełnić ręcznie
- Brak automatycznego przenoszenia danych z ekspertyzy (stan, opis, zdjęcia)

**Co zastąpi AWU:** → Automatyczne tworzenie kart produktów na podstawie danych z ekspertyzy i sesji zdjęciowej

---

## 6. Baselinker → Allegro

**Do czego służy:** Most między Sylius a Allegro. Każda aukcja tworzona ręcznie.

**Problemy:**
- Cena na Allegro musi być wyższa (prowizje) — ręczna zmiana
- Stan, marża, kategoria — ręczna edycja
- Kamila robi cykliczne inwentaryzacje: czy produkty na stronie są też na Allegro

**Co zastąpi AWU:** → Automatyczne wystawianie na Allegro z odpowiednią ceną przez Baselinker API

---

## Podsumowanie: co zmieni AWU

| Dziś | Po wdrożeniu AWU |
|------|------------------|
| 6+ arkuszy Google Sheets | Jeden system z modułami |
| Ręczne kopiowanie danych | Automatyczny import i synchronizacja |
| Brak walidacji | Wymagane pola, walidacja na każdym kroku |
| Papierowy cennik w salonach | Szybka wycena salonowa z ceną na żywo |
| Ręczne łączenie ticketów | Automatyczne powiązanie komunikacji |
| Brak widoczności kosztów | Dashboard z marżą, PCC, serwisem, prowizjami |
| Crashujący arkusz | Stabilny system z bazą danych |

---

*Dokument aktualizowany: 19 marca 2026*
*Źródło: spotkanie Przemysław Wywigacz × Paweł Ostęp, 18 marca 2026*
