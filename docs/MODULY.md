# Mapa modułów — AWU

> Opis każdego modułu systemu z zakresem funkcjonalności, zależnościami i priorytetem.

---

## Przegląd architektury

```
┌─────────────────────────────────────────────────────────────────┐
│                          AWU ADMIN                               │
├───────────────────────────┬─────────────────────────────────────┤
│                           │                                     │
│  CZĘŚĆ I: ODKUP           │  CZĘŚĆ II: PRZYGOTOWANIE            │
│                           │  DO SPRZEDAŻY                       │
│  ┌─────────────────────┐  │  ┌───────────────────────────────┐  │
│  │ 1. Lista wycen      │  │  │ 6. Kanban produktów           │  │
│  │ 2. Szczegóły wyceny │  │  │    (Nowy → Regał → Indeks     │  │
│  │ 3. Weryfikacja      │  │  │     → KGM → Sesja → Karta     │  │
│  │ 4. Umowy            │  │  │     → PZ → Front → Allegro)  │  │
│  │ 5. Salon Quick      │  │  │                               │  │
│  └─────────────────────┘  │  └───────────────────────────────┘  │
│                           │                                     │
├───────────────────────────┴─────────────────────────────────────┤
│                       MODUŁY WSPÓLNE                             │
│  ┌────────────┐ ┌──────────┐ ┌──────────────┐ ┌─────────────┐  │
│  │ 7. Serwis  │ │ 8. PCC   │ │ 9. Komunik.  │ │10. Dashboard│  │
│  └────────────┘ └──────────┘ └──────────────┘ └─────────────┘  │
│  ┌──────────────┐ ┌───────────────┐ ┌────────────────────────┐  │
│  │11. Analityka │ │12. Użytkownicy│ │13. Ustawienia          │  │
│  └──────────────┘ └───────────────┘ └────────────────────────┘  │
├─────────────────────────────────────────────────────────────────┤
│                       INTEGRACJE                                 │
│  Verto (ERP) │ Sylius (sklep) │ Baselinker │ Thulium │ Cyfrolog │
└─────────────────────────────────────────────────────────────────┘
```

---

## CZĘŚĆ I: Odkup (wycena → umowa)

### Moduł 1: Lista wycen
**Priorytet:** Krytyczny (MVP)
**Ścieżka:** `/wyceny`

**Zakres:**
- Tabela wszystkich wycen z paginacją
- Widok kompaktowy: nr wyceny, data, klient, status, kwota
- Filtrowanie: status + zakres dat (priorytetowe), lokalizacja, operator, nr przesyłki (rozszerzone)
- Wyszukiwanie: nr wyceny, nazwisko, email
- Nawigacja do szczegółów (klik na wiersz)
- Przycisk "+ Nowa wycena (salon)"

**Reguły:**
- Operator widzi tylko swój salon
- Admin widzi wszystkie lokalizacje
- Domyślne sortowanie: od najnowszych
- "Przekaż do centrali" w liście dotyczy **konkretnego produktu**, nie całej wyceny

**Zależności:** Moduł 12 (Użytkownicy — role i lokalizacje)

---

### Moduł 2: Szczegóły wyceny
**Priorytet:** Krytyczny (MVP)
**Ścieżka:** `/wyceny/:id`

**Zakres:**
- Nagłówek: nr wyceny, status (badge), dane klienta, przyciski akcji
- Przebieg statusów (timeline wizualny)
- Zakładki: Produkty, Weryfikacja, Komunikacja, Umowy, Wysyłka, Historia

**Zakładka Produkty:**
- Tabela produktów: nazwa, indeks Verto, S/N, stan deklarowany, stan zweryfikowany, akcesoria, ceny
- Rozbieżności podświetlone (żółty/czerwony)
- Podsumowanie kwot

**Zakładka Historia:**
- Audit log: kto, kiedy, co zmienił
- Zmiany statusów z timestampami

**Reguły:**
- Przyciski akcji zależne od aktualnego statusu i roli
- "Zmień status" waliduje warunki przejścia
- "Przekaż do centrali" działa **per produkt** — nie blokuje pozostałych produktów w wycenie

**Zależności:** Moduł 1 (nawigacja), Moduł 3 (weryfikacja), Moduł 4 (umowy)

---

### Moduł 3: Weryfikacja produktu
**Priorytet:** Krytyczny (MVP)
**Widok:** Zakładka w szczegółach wyceny

**Zakres:**
- Formularz weryfikacji per produkt:
  - Ocena stanu (dropdown 5-10/10 z opisami)
  - Numer seryjny (pole tekstowe)
  - Akcesoria (checkboxy — lista z katalogu produktu)
  - Notatka wewnętrzna
- Porównanie: deklarowane vs zweryfikowane (side-by-side)
- Automatyczne przeliczanie ceny po zmianie
- Wykrywanie rozbieżności z wizualnym alertem
- Oznaczenie "Wymaga serwisu" z opisem problemu
- "Zapisz roboczo" / "Zakończ weryfikację"

**Reguły:**
- Akcesoria ODEJMOWANE od ceny (nie dodawane)
- Zakończenie weryfikacji wymaga: ocena + S/N + przynajmniej 1 akcesorium
- Rozbieżność = różnica > 1 punkt w ocenie

**Zależności:** Katalog produktów (lista akcesoriów per model)

---

### Moduł 4: Umowy
**Priorytet:** Krytyczny (MVP)
**Widok:** Zakładka w szczegółach wyceny

**Zakres:**
- Wybór produktów na umowę (checkboxy, produkty w serwisie wykluczone)
- Typ umowy: kupno os. fizyczna / kupno os. fiz. VAT / faktura VAT
- Forma płatności: przelew / karta / gotówka / przelew odwrotny / karta z opieką
- Metoda podpisu: salon / e-podpis / kurier
- Generowanie umowy → podgląd → potwierdzenie
- Śledzenie statusu: Wygenerowana → Wysłana → Podpisana → Anulowana

**Reguły:**
- Numer umowy generowany automatycznie
- Umowa > 1000 zł → automatyczne oznaczenie PCC
- Anulowanie umowy wymaga podania powodu
- Cena na umowie vs cena finalna w Verto mogą się różnić

**Zależności:** Moduł 3 (weryfikacja zakończona), Moduł 8 (PCC)

---

### Moduł 5: Szybka wycena salonowa
**Priorytet:** Krytyczny (MVP)
**Ścieżka:** `/salon/szybka-wycena`

**Zakres (5-krokowy stepper):**

1. **Produkty (wiele!)** — wyszukiwanie z katalogu / skan kodu / ręczne wpisanie. Ocena stanu (5-10), zaznaczenie akcesoriów. Można dodać **wiele produktów** w jednej sesji.

2. **Cena** — cena odkupu: przelew vs karta podarunkowa. Widoczność:
   - Operator: widzi TYLKO cenę odkupu (ile zaproponować klientowi)
   - Senior Operator / Admin: widzi cenę odkupu + cenę sprzedaży + marżę
   - Senior Operator / Admin: może edytować cenę

3. **Decyzja klienta** — 3 scenariusze:
   - A) Klient niezainteresowany → KONIEC
   - B) Klient chce umowę od razu → przejdź do danych klienta
   - C) Klient zostawia sprzęt do ekspertyzy → przejdź do danych klienta

4. **Dane klienta**
   - Osoba fizyczna: imię, nazwisko, email, telefon, PESEL
   - Firma: TYLKO NIP → dane firmy auto-uzupełnienie z API (GUS/CEIDG)
   - Konto bankowe: wymagane TYLKO gdy płatność = przelew ORAZ klient = osoba fizyczna

5. **Finalizacja** (zależy od scenariusza z kroku 3):
   - A) Umowa na miejscu (os. fizyczna): numery seryjne TERAZ → generuj umowę → podpis → rozliczenie
   - B) Protokół pozostawienia sprzętu: numery seryjne TERAZ → generuj protokół (klient dostaje kopię) → sprzęt do ekspertyzy
   - C) Firma — protokół przyjęcia: numery seryjne TERAZ → generuj protokół → czekamy na fakturę od klienta

**Kluczowe reguły:**
- Salon i operator uzupełniane automatycznie z sesji
- Numer seryjny wymagany DOPIERO przy finalizacji (gdy sprzęt zostaje), NIE podczas wstępnej oceny
- Typ dokumentu ustalany automatycznie: osoba fizyczna → umowa, firma → czekamy na ich fakturę
- Korekta ceny wymaga roli Senior Operator / Admin
- Cena z bazy Verto (jeśli jest w katalogu) lub ręczna (jeśli poza katalogiem)
- Konto bankowe NIE wymagane dla karty podarunkowej ani dla firm

**Zależności:** Katalog produktów, Moduł 4 (umowy), API GUS/CEIDG (auto-uzupełnienie danych firmy)

---

## CZĘŚĆ II: Przygotowanie do sprzedaży

### Moduł 6: Kanban produktów
**Priorytet:** Krytyczny (MVP)
**Ścieżka:** `/kanban`

**Zakres:**
- Horyzontalnie przewijana tablica (Trello-style) z 9 kolumnami: Nowy → Regał → Indeks → KGM → Sesja → Karta → PZ → Front → Allegro
- Karty produktów: numer CYF, nazwa, numer umowy, tydzień, operator
- Kliknięcie karty → modal szczegółów produktu z 3 zakładkami:
  - **Info** — dane produktu (nazwa, S/N, cena, klient, umowa), link do powiązanej wyceny
  - **Historia** — pełny audit log (kto, kiedy, co zmienił, przejścia między kolumnami)
  - **Serwis** — widoczna tylko gdy produkt jest/był w serwisie: szczegóły zgłoszenia, serwisant, koszty, daty, logi serwisowe
- Bezpośredni link z karty produktu do jego wyceny
- Drag & drop między kolumnami z potwierdzeniem (system wyświetla checklistę wymaganych warunków)
- Wizualne blokery: Indeks (BLOKER #1), PZ (BLOKER #2)
- Produkty w serwisie wizualnie wyróżnione (pomarańczowa ramka, badge "Serwis")
- Filtry: operator, tydzień, lokalizacja
- Łączna liczba produktów w pipeline

**Reguły:**
- Indeks blokuje: KGM, Sesja, Karta
- PZ blokuje: Front
- Front wymaga ZARÓWNO karty produktu JAK I PZ
- Ręczne dodawanie produktów (B2B/faktury bez CSV)
- Śledzenie pracy tygodniówkami (numer tygodnia)

**Wymagania przejść między kolumnami:**

| Przejście | Wymagane potwierdzenia |
|-----------|----------------------|
| Nowy → Regał | Produkt fizycznie odebrany, Umowa/faktura zweryfikowana |
| Regał → Indeks | Produkt wyczyszczony i zresetowany, Akcesoria uzupełnione, Odłożony na regał |
| Indeks → KGM | Indeks Verto utworzony, Numer indeksu wpisany |
| KGM → Sesja | Karta gwarancyjna wydrukowana, Naklejka EAN wydrukowana, Naklejki fizycznie dołożone |
| Sesja → Karta | Zdjęcia wykonane, Zdjęcia usterek (jeśli są), Produkt spakowany |
| Karta → PZ | Karta w Sylius uzupełniona, Cena sprzedaży ustawiona |
| PZ → Front | Dokument PZ wprowadzony do Verto, Numer PZ zapisany |
| Front → Allegro | MMK wykonane, Produkt aktywowany na cyfrowe.pl |

**Zależności:** Moduł 4 (umowa podpisana → produkt wchodzi na Kanban), Moduł 7 (dane serwisowe w zakładce Serwis)

---

## MODUŁY WSPÓLNE

### Moduł 7: Serwis
**Priorytet:** Wysoki
**Ścieżka:** `/serwis`

**Zakres:**
- Lista aktywnych zleceń serwisowych
- 5 zakładek: Aktywne, Przed zakupem, Po zakupie, Gwarancja, Zakończone
- Tabela: produkt, typ, serwisant, status, koszt, czas
- 3 scenariusze: przed zakupem (klient płaci), po zakupie (Cyfrowe płaci), gwarancja (Cyfrowe płaci)
- Dashboard kosztów serwisowych (per miesiąc, per rok)
- Statusy: Diagnostyka → Wycena naprawy → Oczekuje decyzji → W naprawie → Zakończone

**Reguły:**
- Scenariusz "przed zakupem": po naprawie tworzona NOWA wycena + NOWA umowa na 1 produkt
- Koszt naprawy odejmowany od ceny odkupu (scenariusz 1) lub ponoszony przez Cyfrowe (scenariusze 2-3)

**Zależności:** Moduł 2/3 (powiązanie z produktem i wycenią)

---

### Moduł 8: PCC (Podatek)
**Priorytet:** Wysoki
**Ścieżka:** `/pcc`

**Zakres:**
- Podsumowanie bieżącego miesiąca: liczba umów, wartość, PCC (2%)
- Lista umów podlegających PCC (> 1000 zł)
- Porównanie: kwota AWU vs kwota Verto
- Statusy: OK / Brak w Verto / Rozbieżność
- Do wyjaśnienia: tabela problemów do rozwiązania
- Eksport raportu PCC (CSV/Excel)
- Generowanie raportu PCC

**Reguły:**
- PCC = 2% od finalnej wartości z Verto (nie z umowy!)
- Termin: do 7. dnia następnego miesiąca
- Anulowane umowy nie podlegają PCC
- Źródło prawdy: cena z Verto

**Zależności:** Moduł 4 (umowy), integracja Verto (ceny finalne)

---

### Moduł 9: Komunikacja
**Priorytet:** Średni
**Widok:** Zakładka w szczegółach wyceny + integracja z Thulium

**Zakres:**
- Historia komunikacji per wycena (timeline)
- Typy: email wysłany/odebrany, SMS, notatka wewnętrzna, zdarzenie systemowe
- 6 szablonów mailowych
- Wysyłanie maila/SMS z poziomu wyceny
- Automatyczne powiadomienia (zmiana statusu, przypomnienia)

**Zależności:** Integracja z Thulium

---

### Moduł 10: Dashboard
**Priorytet:** Średni
**Ścieżka:** `/` (strona główna)

**Zakres:**
- KPI Odkup: nowe wyceny, w weryfikacji, oczekujące, umowy do podpisu, zakończone
- KPI Kanban: produkty na każdym kroku, blokery
- Poranny rytuał: tabela problemów do sprawdzenia (duplikaty, brakujące dane, produkty bez cen)
- Ostatnia aktywność (timeline)
- Serwis: aktywne zlecenia
- Widok dostosowany do roli

---

### Moduł 11: Analityka
**Priorytet:** Niski (po MVP)
**Ścieżka:** `/analityka`

**Zakres:**
- KPI: produkty sprzedane, refurby, średnia marża, dodatkowe koszty
- Wolumen odkupów — trend miesięczny (wykres)
- Marżowość per kategoria (wykres)
- Lokalizacje — rozbicie (tabela)
- Koszty dodatkowe — rozbicie: Allegro, PCC, serwis, akcesoria
- Eksport danych

---

### Moduł 12: Użytkownicy
**Priorytet:** Średni
**Ścieżka:** `/uzytkownicy`

**Zakres:**
- Lista użytkowników z filtrami (rola, lokalizacja)
- Dodawanie / edycja / dezaktywacja
- Role: Operator, Senior Operator, Admin
- Przypisanie do lokalizacji

---

### Moduł 13: Ustawienia
**Priorytet:** Niski
**Ścieżka:** `/ustawienia`

**Zakres:**
- Profil użytkownika
- Preferencje powiadomień
- Lista lokalizacji (podgląd)
- Status integracji (Verto, Sylius, Baselinker, Thulium, Cyfrolog)

---

## Kolejność implementacji (rekomendowana)

| Faza | Moduły | Uzasadnienie |
|------|--------|-------------|
| **Faza 1 (MVP)** | 1, 2, 3, 4, 5, 6, 12 | Pełny workflow odkupu + Kanban + role |
| **Faza 2** | 7, 8, 10 | Serwis, PCC, Dashboard |
| **Faza 3** | 9, 11, 13 | Komunikacja, Analityka, Ustawienia |

---

*Dokument aktualizowany: 19 marca 2026*
