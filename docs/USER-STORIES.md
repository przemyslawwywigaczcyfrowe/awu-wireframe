# User Stories — AWU

> Kompletny zestaw user stories z kryteriami akceptacji.
> Format: Jako [rola], chcę [akcja], aby [cel].

---

## Spis treści

1. [Odkup — Lista wycen](#1-odkup--lista-wycen)
2. [Odkup — Weryfikacja produktu](#2-odkup--weryfikacja-produktu)
3. [Odkup — Negocjacje i komunikacja](#3-odkup--negocjacje-i-komunikacja)
4. [Odkup — Umowy](#4-odkup--umowy)
5. [Odkup — Szybka wycena salonowa](#5-odkup--szybka-wycena-salonowa)
6. [Kanban — Przygotowanie do sprzedaży](#6-kanban--przygotowanie-do-sprzedaży)
7. [Serwis](#7-serwis)
8. [PCC](#8-pcc)
9. [Komunikacja](#9-komunikacja)
10. [Dashboard i analityka](#10-dashboard-i-analityka)
11. [Użytkownicy i uprawnienia](#11-użytkownicy-i-uprawnienia)

---

## 1. Odkup — Lista wycen

### US-001: Przeglądanie listy wycen
**Jako** operator, **chcę** widzieć listę wycen przypisanych do mojego salonu, **aby** wiedzieć co mam do zrobienia.

**Kryteria akceptacji:**
- Operator widzi tylko wyceny ze swojej lokalizacji
- Admin widzi wyceny ze wszystkich lokalizacji
- Lista domyślnie posortowana od najnowszych
- Widok kompaktowy: nr wyceny, data, klient, status, kwota

### US-002: Filtrowanie wycen
**Jako** operator, **chcę** filtrować wyceny po statusie i dacie, **aby** szybko znaleźć te wymagające mojej uwagi.

**Kryteria akceptacji:**
- Filtr po statusie (dropdown z 11 statusami)
- Filtr po zakresie dat (od–do)
- Filtry można łączyć
- Przycisk "Wyczyść" resetuje wszystkie filtry
- Wyniki aktualizują się natychmiast

### US-003: Wyszukiwanie wycen
**Jako** operator, **chcę** wyszukać wycenę po numerze, nazwisku klienta, emailu lub numerze przesyłki, **aby** szybko znaleźć konkretną sprawę.

**Kryteria akceptacji:**
- Pole wyszukiwania w górnej części listy
- Wyszukiwanie po: nr wyceny, nazwisko, email, nr przesyłki
- Wyniki pojawiają się podczas wpisywania (debounce 300ms)

### US-004: Otwarcie szczegółów wyceny
**Jako** operator, **chcę** kliknąć na wycenę w liście i zobaczyć jej pełne szczegóły, **aby** podjąć działania.

**Kryteria akceptacji:**
- Klik na wiersz otwiera widok szczegółowy
- Widok szczegółowy zawiera: dane klienta, produkty, statusy, historię
- Przycisk "Wróć" przenosi do listy z zachowaniem filtrów

---

## 2. Odkup — Weryfikacja produktu

### US-010: Porównanie stanu deklarowanego i rzeczywistego
**Jako** operator, **chcę** porównać stan zadeklarowany przez klienta ze stanem rzeczywistym, **aby** skorygować cenę jeśli trzeba.

**Kryteria akceptacji:**
- Widok side-by-side: deklarowane vs zweryfikowane
- Ocena na skali 5-10/10 z opisem każdego poziomu
- Jeśli jest rozbieżność → wizualne ostrzeżenie (żółty/czerwony)
- System automatycznie oblicza nową cenę

### US-011: Weryfikacja akcesoriów
**Jako** operator, **chcę** zaznaczyć które akcesoria faktycznie są, a których brakuje, **aby** system przeliczył cenę.

**Kryteria akceptacji:**
- Lista akcesoriów z checkboxami (obecne/brakujące)
- Akcesoria z katalogu produktu (dedykowane per model)
- Możliwość dodania niestandardowego akcesorium
- Brakujące akcesoria ODEJMUJĄ wartość od ceny bazowej
- Cena aktualizuje się na żywo

### US-012: Automatyczne przeliczanie ceny
**Jako** operator, **chcę** zobaczyć automatycznie obliczoną nową cenę po zmianie stanu lub akcesoriów, **aby** nie liczyć ręcznie.

**Kryteria akceptacji:**
- Cena przelew i cena karta podarunkowa aktualizują się automatycznie
- Widoczna różnica między ceną pierwotną a nową
- Wyraźne oznaczenie "cena spadła" / "cena bez zmian"

### US-013: Oznaczenie produktu do serwisu
**Jako** operator, **chcę** oznaczyć produkt jako „wymaga serwisu" podczas weryfikacji, **aby** rozpocząć proces diagnostyki.

**Kryteria akceptacji:**
- Przycisk "Wymaga serwisu" w formularzu weryfikacji
- Pole na opis problemu (wymagane)
- Produkt przechodzi do statusu "Serwis — diagnostyka"
- Pozostałe produkty z wyceny mogą iść dalej normalnie

### US-014: Zapisanie weryfikacji
**Jako** operator, **chcę** zapisać wyniki weryfikacji roboczo lub finalnie, **aby** nie tracić postępu pracy.

**Kryteria akceptacji:**
- "Zapisz roboczo" — zachowuje dane bez zmiany statusu
- "Zakończ weryfikację" — zmienia status na "Zweryfikowana"
- Wymagane pola: ocena stanu, numer seryjny, przynajmniej 1 akcesorium zaznaczone
- Ostrzeżenie jeśli jest rozbieżność i nie została potwierdzona

---

## 3. Odkup — Negocjacje i komunikacja

### US-020: Wysłanie nowej propozycji cenowej
**Jako** operator, **chcę** wysłać klientowi mail z nową propozycją cenową jednym kliknięciem, **aby** przyspieszyć proces.

**Kryteria akceptacji:**
- Przycisk "Wyślij nową ofertę" dostępny po zakończeniu weryfikacji z rozbieżnością
- Mail generowany z szablonu z uzupełnionymi danymi (produkty, stara cena, nowa cena)
- Operator może edytować treść przed wysłaniem
- Status zmienia się na "Oczekuje na decyzję"

### US-021: Śledzenie decyzji klienta
**Jako** operator, **chcę** widzieć czy klient odpowiedział na ofertę, **aby** podjąć kolejne kroki.

**Kryteria akceptacji:**
- Widoczne: data wysłania oferty, czy klient odpowiedział
- Przycisk "Klient akceptuje" / "Klient odrzuca"
- Po akceptacji → przejście do generowania umowy
- Po odrzuceniu → przejście do zwrotu produktu
- Automatyczne przypomnienie po 6 dniach bez odpowiedzi

### US-022: Kontakt telefoniczny
**Jako** operator, **chcę** odnotować rozmowę telefoniczną z klientem, **aby** zachować historię kontaktu.

**Kryteria akceptacji:**
- Przycisk "Dodaj notatkę z rozmowy"
- Pole: data, kto dzwonił, podsumowanie
- Notatka widoczna w historii komunikacji

---

## 4. Odkup — Umowy

### US-030: Wybór produktów na umowę
**Jako** operator, **chcę** wybrać które produkty z wyceny idą na umowę, **aby** obsłużyć częściowe odkupy.

**Kryteria akceptacji:**
- Checkboxy przy każdym produkcie
- Produkty w serwisie automatycznie wyłączone
- Suma umowy oblicza się dynamicznie
- Przynajmniej 1 produkt wymagany

### US-031: Generowanie umowy
**Jako** operator, **chcę** wygenerować umowę z wybranymi produktami i formą rozliczenia, **aby** sfinalizować transakcję.

**Kryteria akceptacji:**
- Wybór typu umowy (kupno os. fizyczna, kupno os. fiz. VAT, faktura VAT)
- Wybór formy płatności (przelew, karta, gotówka, przelew odwrotny, karta z opieką)
- Podgląd umowy przed generowaniem
- Numer umowy generowany automatycznie
- Status zmienia się na "Umowa podpisana" po podpisaniu

### US-032: Śledzenie podpisu umowy
**Jako** admin, **chcę** śledzić status podpisu umowy, **aby** wiedzieć co jeszcze nie jest zamknięte.

**Kryteria akceptacji:**
- Statusy: Wygenerowana → Wysłana → Podpisana → Zwrócona / Anulowana
- Filtr w liście wycen po statusie umowy
- Informacja o metodzie podpisu (salon / e-podpis / kurier)

### US-033: Anulowanie umowy
**Jako** admin, **chcę** oznaczyć umowę jako anulowaną, **aby** nie uczestniczyła w PCC i raportach.

**Kryteria akceptacji:**
- Przycisk "Anuluj umowę" z polem na powód
- Umowa nie bierze udziału w PCC po anulowaniu
- Status zmienia się na "Anulowana"
- Wpis w audit logu

---

## 5. Odkup — Szybka wycena salonowa

### US-040: Szybka identyfikacja produktu
**Jako** pracownik salonu, **chcę** szybko znaleźć produkt w katalogu, **aby** podać klientowi cenę bez szukania w Excelu.

**Kryteria akceptacji:**
- Pole wyszukiwania z autouzupełnianiem
- Wyniki z katalogu produktów (nazwa, marka, wariant)
- Opcja skanowania kodu kreskowego
- Jeśli produktu nie ma w katalogu → ręczne wpisanie nazwy

### US-041: Szybka wycena i pokazanie ceny
**Jako** pracownik salonu, **chcę** wybrać stan produktu i zobaczyć natychmiastowo proponowaną cenę, **aby** powiedzieć klientowi ile dostanie.

**Kryteria akceptacji:**
- Wybór stanu (5-10/10) jednym kliknięciem
- Zaznaczenie posiadanych akcesoriów
- Cena wyświetla się natychmiast (przelew vs karta)
- Widoczna informacja o marży (dla senior operatorów)

### US-042: Wprowadzenie danych klienta
**Jako** pracownik salonu, **chcę** wprowadzić dane klienta i wygenerować umowę w jednym flow, **aby** nie przeskakiwać między systemami.

**Kryteria akceptacji:**
- Formularz: imię, nazwisko, email, telefon, PESEL, nr konta
- Typ klienta: osoba fizyczna / firma
- Jeśli firma: NIP, nazwa firmy
- Adres (opcjonalny — wymagany przy wysyłce kurierem)
- Walidacja danych przed generowaniem umowy

### US-043: Automatyczna lokalizacja i operator
**Jako** pracownik salonu, **chcę** żeby system wiedział w jakim jestem salonie, **aby** nie wpisywać tego ręcznie.

**Kryteria akceptacji:**
- Salon i operator uzupełniane automatycznie z profilu użytkownika
- Brak potrzeby ręcznego wybierania lokalizacji

---

## 6. Kanban — Przygotowanie do sprzedaży

### US-050: Widok tablicy Kanban
**Jako** operator, **chcę** widzieć tablicę Kanban z produktami na każdym kroku, **aby** wiedzieć co jest do zrobienia.

**Kryteria akceptacji:**
- 9 kolumn: Nowy → Regał → Indeks → KGM → Sesja → Karta → PZ → Front → Allegro
- Każda kolumna pokazuje liczbę produktów
- Karty produktów z: numer CYF, nazwa produktu, numer umowy, tydzień, operator

### US-051: Wizualne blokery
**Jako** operator, **chcę** widzieć blokery wizualnie, **aby** wiedzieć co blokuje postęp.

**Kryteria akceptacji:**
- Kolumna "Indeks" oznaczona jako BLOKER #1 (czerwona)
- Kolumna "PZ" oznaczona jako BLOKER #2 (czerwona)
- Produkty bez indeksu nie mogą przejść do KGM, Sesja, Karta
- Produkty bez PZ nie mogą przejść do Front

### US-052: Przenoszenie produktów
**Jako** operator, **chcę** przenosić produkty między krokami Kanbanu, **aby** odzwierciedlić postęp pracy.

**Kryteria akceptacji:**
- Drag & drop między kolumnami
- Walidacja: nie można przenieść dalej jeśli bloker aktywny
- Zapis automatyczny po przeniesieniu
- Wpis w historii produktu

### US-053: Filtrowanie Kanbanu
**Jako** operator, **chcę** filtrować Kanban po operatorze, tygodniu lub lokalizacji, **aby** widzieć tylko swoje produkty.

**Kryteria akceptacji:**
- Filtr po: operatorze, numerze tygodnia, lokalizacji
- Łączna liczba produktów w pipeline
- Wyszukiwanie po: numerze CYF, nazwie produktu, numerze umowy

### US-054: Ręczne dodawanie do Kanbanu (B2B/faktury)
**Jako** admin, **chcę** ręcznie dodać produkty do Kanbanu, **aby** obsłużyć zakupy na fakturę, które nie generują CSV.

**Kryteria akceptacji:**
- Formularz: nazwa produktu, numer umowy/faktury, dane dostawcy
- Możliwość dodania wielu produktów naraz (bulk)
- Produkty pojawiają się w kolumnie "Nowy"

---

## 7. Serwis

### US-060: Rejestracja zlecenia serwisowego
**Jako** operator, **chcę** zarejestrować zlecenie serwisowe powiązane z produktem, **aby** śledzić naprawę.

**Kryteria akceptacji:**
- Wybór scenariusza: przed zakupem / po zakupie / gwarancja
- Pole: opis problemu, serwisant, koszt diagnostyki
- Powiązanie z produktem i wycenią

### US-061: Śledzenie statusu naprawy
**Jako** operator, **chcę** śledzić status naprawy i jej koszty, **aby** informować klienta o postępie.

**Kryteria akceptacji:**
- Statusy: Diagnostyka → Wycena naprawy → Oczekuje na decyzję → W naprawie → Zakończone
- Widoczny koszt diagnostyki i naprawy
- Informacja kto ponosi koszt (klient / Cyfrowe.pl)

### US-062: Dashboard kosztów serwisowych
**Jako** admin, **chcę** widzieć sumaryczne koszty serwisowe per miesiąc/rok, **aby** kontrolować ukryte koszty.

**Kryteria akceptacji:**
- Podsumowanie: koszty per miesiąc i per rok
- Rozbicie na scenariusze (przed zakupem, po zakupie, gwarancja)
- Rozbicie na serwisantów

---

## 8. PCC

### US-070: Lista umów podlegających PCC
**Jako** admin, **chcę** widzieć listę umów podlegających PCC z podziałem na miesiące, **aby** przygotować raport dla księgowości.

**Kryteria akceptacji:**
- Lista umów > 1000 zł z datą, klientem, kwotą
- Podział na miesiące rozliczeniowe
- Podsumowanie: liczba umów, łączna wartość, kwota PCC (2%)

### US-071: Porównanie kwot umowa vs Verto
**Jako** admin, **chcę** porównać kwotę z umowy z kwotą z Verto, **aby** wykryć rozbieżności.

**Kryteria akceptacji:**
- Kolumny: kwota AWU, kwota Verto, różnica
- Wizualne oznaczenie rozbieżności (czerwone)
- Status: OK / Brak w Verto / Rozbieżność

### US-072: Eksport raportu PCC
**Jako** admin, **chcę** wyeksportować raport PCC do CSV/Excel, **aby** wysłać go do księgowości.

**Kryteria akceptacji:**
- Przycisk "Eksportuj do CSV"
- Plik zawiera: numer umowy, datę, klienta, kwotę, PCC
- Gotowy do wysłania do księgowości

---

## 9. Komunikacja

### US-080: Wysyłanie maila z szablonu
**Jako** operator, **chcę** wysłać mail do klienta z szablonu jednym kliknięciem, **aby** przyspieszyć komunikację.

**Kryteria akceptacji:**
- 6 szablonów: potwierdzenie odbioru, wynik weryfikacji (zgodny), wynik weryfikacji (rozbieżność), korygowana cena, przypomnienie (6 dni), informacja o zwrocie
- Szablon uzupełniony danymi wyceny/produktów
- Operator może edytować przed wysłaniem

### US-081: Notatka wewnętrzna
**Jako** operator, **chcę** dodać notatkę wewnętrzną do wyceny, **aby** przekazać informacje kolegom bez wysyłania do klienta.

**Kryteria akceptacji:**
- Pole notatki wyraźnie oznaczone jako "wewnętrzna"
- Notatka widoczna tylko dla operatorów, NIE dla klienta
- Widoczna w historii komunikacji z oznaczeniem "wewnętrzna"

### US-082: Historia komunikacji
**Jako** operator, **chcę** widzieć pełną historię komunikacji z klientem w jednym miejscu, **aby** nie szukać w różnych systemach.

**Kryteria akceptacji:**
- Timeline: maile, SMS, notatki wewnętrzne, zdarzenia systemowe
- Chronologicznie od najnowszych
- Integracja z Thulium (jeśli dostępna)

---

## 10. Dashboard i analityka

### US-090: Dashboard operatora
**Jako** operator, **chcę** widzieć podsumowanie mojej pracy na dashboardzie, **aby** wiedzieć co jest do zrobienia.

**Kryteria akceptacji:**
- KPI: nowe wyceny, w trakcie weryfikacji, oczekujące na decyzję, umowy do podpisu
- Ostatnia aktywność (timeline)
- Szybkie akcje: nowa wycena, skanuj kod

### US-091: Dashboard Kanban
**Jako** admin, **chcę** widzieć ile produktów czeka na każdym kroku Kanbanu, **aby** zarządzać zasobami.

**Kryteria akceptacji:**
- KPI per krok Kanbanu
- Wizualne blokery (ile produktów zablokowanych na Indeksie i PZ)
- Poranny rytuał: lista problemów do sprawdzenia

### US-092: Analityka marżowości
**Jako** admin, **chcę** widzieć rzeczywistą marżę z uwzględnieniem ukrytych kosztów, **aby** podejmować trafne decyzje biznesowe.

**Kryteria akceptacji:**
- Marża per produkt/kategoria/lokalizacja
- Koszty ukryte: serwis, PCC, prowizje Allegro, faktury na akcesoria
- Trendy czasowe (miesiąc do miesiąca)
- Eksport danych

---

## 11. Użytkownicy i uprawnienia

### US-100: Zarządzanie użytkownikami
**Jako** admin, **chcę** dodawać, edytować i dezaktywować użytkowników, **aby** kontrolować dostęp do systemu.

**Kryteria akceptacji:**
- Formularz: imię, nazwisko, email, rola, lokalizacja
- Role: Operator, Senior Operator, Admin
- Aktywacja/dezaktywacja bez usuwania

### US-101: Kontrola widoczności
**Jako** system, **chcę** ograniczać widoczność danych na podstawie roli i lokalizacji, **aby** chronić dane.

**Kryteria akceptacji:**
- Operator widzi tylko dane ze swojej lokalizacji
- Senior Operator widzi tylko dane ze swojej lokalizacji + korekta cen
- Admin widzi dane ze wszystkich lokalizacji

---

*Dokument aktualizowany: 19 marca 2026*
*Wersja: 1.0*
*Liczba user stories: 35*
