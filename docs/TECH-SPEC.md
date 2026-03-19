# AWU — Specyfikacja techniczna
## Dokument dla programistów | Cyfrowe.pl

> Ten dokument opisuje strukturę danych, moduły systemu, przepływy informacji
> i wymagania integracyjne systemu AWU do zarządzania skupem produktów używanych.

---

## Spis treści

1. [Architektura modułów](#1-architektura-modułów)
2. [Model danych](#2-model-danych)
3. [Statusy i maszyny stanów](#3-statusy-i-maszyny-stanów)
4. [Moduły systemu](#4-moduły-systemu)
5. [Integracje zewnętrzne](#5-integracje-zewnętrzne)
6. [Uprawnienia i role](#6-uprawnienia-i-role)
7. [User Stories](#7-user-stories)

---

## 1. Architektura modułów

```
┌─────────────────────────────────────────────────────────────┐
│                        AWU ADMIN                            │
├─────────────────────┬───────────────────────────────────────┤
│                     │                                       │
│  CZĘŚĆ I: ODKUP     │  CZĘŚĆ II: PRZYGOTOWANIE DO SPRZEDAŻY │
│                     │                                       │
│  ┌───────────────┐  │  ┌─────────────────────────────────┐  │
│  │ Lista wycen   │  │  │ Kanban produktów                │  │
│  │ Szczegóły     │  │  │ (Nowy → Regał → Indeks → KGM   │  │
│  │ Weryfikacja   │  │  │  → Sesja → Karta → PZ         │  │
│  │ Negocjacje    │  │  │  → Front → Allegro)             │  │
│  │ Umowy         │  │  │                                 │  │
│  │ Salon Quick   │  │  │                                 │  │
│  └───────────────┘  │  └─────────────────────────────────┘  │
│                     │                                       │
├─────────────────────┴───────────────────────────────────────┤
│                    MODUŁY WSPÓLNE                            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌───────────────┐  │
│  │ Serwis   │ │ PCC      │ │Komunikacja│ │ Dashboard     │  │
│  │ /naprawy │ │ /podatek │ │ /Thulium  │ │ /analityka    │  │
│  └──────────┘ └──────────┘ └──────────┘ └───────────────┘  │
│  ┌──────────────┐ ┌───────────────────┐                     │
│  │ Użytkownicy  │ │ Ustawienia        │                     │
│  └──────────────┘ └───────────────────┘                     │
├─────────────────────────────────────────────────────────────┤
│                    INTEGRACJE                                │
│  Verto (ERP) │ Sylius (sklep) │ Baselinker │ Thulium │ Cyfrolog │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. Model danych

### Zasada #1: PRODUKT jest główną encją

Wycena grupuje produkty, ale po weryfikacji każdy produkt idzie własną ścieżką.
Eskalacja ("Przekaż do centrali") działa **per produkt** — w jednej wycenie z 4 produktami, 1 produkt może być przekazany do centrali, a 3 pozostałe kontynuują normalnie.
Moment przejścia z "wyceny" na "produkt": **zatwierdzenie umowy**.

### Encje

#### Product (główna encja)
```
Product {
  id                    : UUID
  appraisalId           : FK → Appraisal         // wycena, z której pochodzi
  contractId            : FK → Contract | null    // umowa, na której kupiony

  // Identyfikacja
  catalogProductId      : FK → CatalogProduct     // produkt z katalogu (np. "Canon EOS 2000D")
  vertoIndex            : string | null           // indeks Verto (nadawany w kroku INDEKS)
  cyfrologNumber        : string | null           // numer CYF
  eanCode               : string | null           // kod EAN
  serialNumber          : string | null

  // Stan deklarowany (przez klienta)
  declaredRating        : int (5-10)
  declaredAccessories   : AccessoryItem[]

  // Stan zweryfikowany (przez operatora)
  verifiedRating        : int (5-10) | null
  verifiedAccessories   : AccessoryItem[] | null
  hasDiscrepancy        : boolean                 // czy jest rozbieżność

  // Ceny
  priceTransfer         : decimal | null          // cena odkupu (przelew)
  priceGiftCard         : decimal | null          // cena odkupu (karta podarunkowa)
  priceResale           : decimal | null          // cena sprzedaży na froncie
  priceAllegro          : decimal | null          // cena na Allegro (wyższa o prowizje)
  priceVertoFinal       : decimal | null          // finalna cena z Verto (źródło prawdy)

  // Status w procesie
  appraisalStatus       : AppraisalProductStatus  // status w ramach wyceny
  kanbanStep            : KanbanStep | null       // krok w Kanbanie (po umowie)
  serviceStatus         : ServiceStatus | null    // status serwisowy (jeśli dotyczy)

  // Metadata
  locationId            : FK → Location
  operatorId            : FK → User | null
  notes                 : string
  internalNote          : string                  // notatka wewnętrzna (nie dla klienta)

  createdAt             : datetime
  updatedAt             : datetime
}
```

#### Appraisal (wycena — encja grupująca)
```
Appraisal {
  id                    : UUID
  number                : string                  // nr wyceny (np. "2331")

  // Klient
  clientId              : FK → Client

  // Kontekst
  source                : enum (ONLINE, SALON)    // skąd przyszła wycena
  locationId            : FK → Location
  assignedOperatorId    : FK → User | null

  // Warunki
  agreementType         : enum (UMOWA_OSOBY_FIZ, UMOWA_OSOBY_FIZ_VAT, FAKTURA_VAT)
  paymentType           : enum (PRZELEW, KARTA_PODARUNKOWA, GOTOWKA, PRZELEW_ODWROTNY, KARTA_PELNA_OPIEKA)
  collectionType        : enum (KURIER_DPD, PACZKOMAT_INPOST, WYSYLKA_WLASNA, SALON)

  // Status
  status                : AppraisalStatus

  // Wartości sumaryczne (obliczane z produktów)
  totalPriceTransfer    : decimal
  totalPriceGiftCard    : decimal

  // Daty
  createdAt             : datetime
  expiryDate            : datetime                // ważność wyceny (7 dni)
  verifiedAt            : datetime | null
  decidedAt             : datetime | null

  // Powiązania
  products              : Product[]
  contracts             : Contract[]
  communications        : Communication[]
  shipments             : Shipment[]
  auditLog              : AuditEntry[]
}
```

#### Client (klient)
```
Client {
  id                    : UUID
  name                  : string
  email                 : string
  phone                 : string

  // Adres
  street                : string | null
  buildingNumber        : string | null
  apartmentNumber       : string | null
  postalCode            : string | null
  city                  : string | null

  // Firma
  isCompany             : boolean
  companyName           : string | null
  nip                   : string | null           // NIP (dla firm)

  createdAt             : datetime
}
```

#### Contract (umowa)
```
Contract {
  id                    : UUID
  appraisalId           : FK → Appraisal

  type                  : enum (UMOWA_KUPNA, FAKTURA_VAT)
  subtype               : enum (OSOBA_FIZYCZNA, OSOBA_FIZYCZNA_VAT, FIRMA)
  number                : string                  // numer umowy

  // Kwoty
  totalAmount           : decimal                 // suma na umowie
  vertoFinalAmount      : decimal | null          // finalna kwota z Verto (źródło prawdy)

  // Status
  status                : enum (WYGENEROWANA, WYSLANA, PODPISANA, ZWROCONA, ANULOWANA)
  signingMethod         : enum (SALON, ONLINE_EPODPIS, KURIER)

  // Powiązane produkty
  products              : Product[]               // które produkty obejmuje umowa

  // PCC
  pccApplicable         : boolean                 // czy > 1000 zł
  pccMonth              : string | null           // miesiąc rozliczenia PCC (np. "2026-03")

  // Verto
  vertoDocNumber        : string | null           // nr dokumentu zakupu 03/RACH

  // Daty
  createdAt             : datetime
  signedAt              : datetime | null
  vertoEnteredAt        : datetime | null
}
```

#### Shipment (przesyłka)
```
Shipment {
  id                    : UUID
  appraisalId           : FK → Appraisal

  type                  : enum (KURIER_DPD, PACZKOMAT_INPOST, WYSYLKA_WLASNA, PRZYCHODZACA_SALON, ZWROT_KURIER, ZWROT_INPOST)
  carrier               : enum (DPD, INPOST, WLASNA, SALON)
  trackingNumber        : string | null

  status                : enum (ZAMOWIONA, ODEBRANA, W_TRANSPORCIE, DOSTARCZONA, ZWROCONA)

  address               : Address | null

  createdAt             : datetime
  updatedAt             : datetime
}
```

> **Metody dostawy przychodzące (3 typy):**
> - `KURIER_DPD` — numer listu przewozowego DPD powiązany z wycenią, wyszukiwalny ze skanerem na liście wycen
> - `PACZKOMAT_INPOST` — numer przesyłki InPost powiązany z wycenią, wyszukiwalny ze skanerem na liście wycen
> - `WYSYLKA_WLASNA` — klient wysyła własną metodą, BEZ numeru trackingowego powiązanego z wycenią. Identyfikacja ręczna po otwarciu przesyłki (wyszukanie po nazwisku/emailu/nr wyceny).

#### ServiceTicket (zlecenie serwisowe)
```
ServiceTicket {
  id                    : UUID
  productId             : FK → Product

  scenario              : enum (PRZED_ZAKUPEM, PO_ZAKUPIE, GWARANCJA)

  // Serwis
  serviceProvider       : string                  // np. "ProClub"
  diagnosticCost        : decimal                 // np. 69 zł
  repairCost            : decimal | null          // koszt naprawy
  costBearer            : enum (KLIENT, CYFROWE)  // kto płaci

  // Status
  status                : enum (DIAGNOSTYKA, WYCENA_NAPRAWY, OCZEKUJE_DECYZJI, W_NAPRAWIE, ZAKONCZONE, ANULOWANE)

  // Wynik
  resultDescription     : string | null
  newPriceAfterRepair   : decimal | null          // nowa cena po naprawie (scenariusz 1)

  createdAt             : datetime
  completedAt           : datetime | null
}
```

#### Communication (komunikacja)
```
Communication {
  id                    : UUID
  appraisalId           : FK → Appraisal | null
  productId             : FK → Product | null

  type                  : enum (EMAIL_WYSLANY, EMAIL_ODEBRANY, SMS_WYSLANY, SMS_ODEBRANY, NOTATKA_WEWNETRZNA, SYSTEMOWY)

  from                  : string
  to                    : string
  subject               : string | null
  body                  : string

  templateId            : FK → EmailTemplate | null
  thuliumTicketId       : string | null           // powiązanie z Thulium

  status                : enum (WYSLANA, DOSTARCZONA, NIEUDANA, PRZECZYTANA)

  createdAt             : datetime
}
```

#### Location (lokalizacja)
```
Location {
  id                    : UUID
  name                  : string                  // np. "Salon Kraków"
  type                  : enum (SALON, CENTRALA)

  street                : string
  city                  : string
  postalCode            : string

  openingHours          : string

  isActive              : boolean
}
```

---

## 3. Statusy i maszyny stanów

### 3.1 Status wyceny (AppraisalStatus)

```
                    ┌──────────────────┐
                    │      NOWA        │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  W TRAKCIE       │
                    │  WERYFIKACJI     │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
                    │  ZWERYFIKOWANA   │
                    └────────┬─────────┘
                             │
                    ┌────────▼─────────┐
              ┌─────│  OCZEKUJE NA     │─────┐
              │     │  DECYZJĘ         │     │
              │     └──────────────────┘     │
              │                              │
     ┌────────▼─────────┐        ┌──────────▼───────┐
     │  ZAAKCEPTOWANA   │        │   ODRZUCONA      │
     └────────┬─────────┘        └──────────┬───────┘
              │                              │
     ┌────────▼─────────┐        ┌──────────▼───────┐
     │  UMOWA PODPISANA │        │  ZWROT DO        │
     └────────┬─────────┘        │  KLIENTA         │
              │                  └──────────────────┘
     ┌────────▼─────────┐
     │  REALIZACJA      │
     │  FINANSOWA       │
     └────────┬─────────┘
              │
     ┌────────▼─────────┐
     │  ZAKOŃCZONA      │
     └──────────────────┘

Z dowolnego statusu (przed UMOWA PODPISANA):
  → PRZEKAZANA DO CENTRALI (dotyczy konkretnego PRODUKTU, nie całej wyceny)
  → ZWROT DO KLIENTA
```

> **Zasada projektowa:** Na obecnym etapie nie ograniczamy przejść między statusami.
> Produkt może przejść z dowolnego statusu na dowolny. Każda zmiana jest logowana
> (kto, kiedy, z czego na co). Ewentualne ograniczenia przejść zostaną dodane w przyszłości.
>
> Powyższy diagram przedstawia **typowy flow**, ale NIE jest wymuszany przez system.

### 3.2 Krok Kanban (KanbanStep)

```
NOWY → REGAL → INDEKS → KGM_WYDRUK → SESJA_PAKOWANIE → KARTA_PRODUKTU → PZ → FRONT → ALLEGRO → SPRZEDANY

Typowy flow (kolejność niezmuszona przez system):
- INDEKS zazwyczaj blokuje: KGM_WYDRUK, SESJA_PAKOWANIE, KARTA_PRODUKTU
- PZ zazwyczaj blokuje: FRONT
- KARTA_PRODUKTU + PZ → oba zalecane przed FRONT
- Z dowolnego kroku: → SERWIS (ścieżka boczna)

> Zasada projektowa: jak w statusach wyceny — brak wymuszonych ograniczeń przejść na obecnym etapie.
```

### 3.2.1 Wymagania przejść Kanban (TransitionRequirement)

Każde przejście między kolumnami wymaga potwierdzenia checklisty. Model danych:

```
TransitionRequirement {
  id        : UUID
  label     : string          // np. "Produkt fizycznie odebrany"
  checked   : boolean         // czy operator potwierdził
}
```

Każde przejście tworzy wpisy w logu:
- **Wpis przejścia**: kto, kiedy, z jakiej kolumny na jaką
- **Wpisy potwierdzeń**: po jednym dla każdego potwierdzonego wymagania (requirement label, operator, timestamp)

Produkty aktualnie w serwisie są oznaczane flagą:
```
Product {
  ...
  inService     : boolean       // czy produkt jest aktualnie w serwisie
  serviceNote   : string | null // notatka serwisowa (np. "Diagnostyka obiektywu")
}
```

Produkty z `inService: true` są wyświetlane na Kanbanie z pomarańczową ramką i badge "🔧 Serwis".

### 3.3 Status serwisowy (ServiceStatus)

```
DIAGNOSTYKA → WYCENA_NAPRAWY → OCZEKUJE_DECYZJI → W_NAPRAWIE → ZAKONCZONE
                                     │
                                     └──→ ANULOWANE
```

---

## 4. Moduły systemu

### Moduł 1: Dashboard
- KPI: ilość wycen per status, wartość skupu, średni czas obsługi
- KPI Kanban: ile produktów na każdym kroku, blokery
- Szybkie akcje: nowa wycena, skanuj kod
- Widok dostosowany do roli (operator widzi swój salon)

### Moduł 2: Lista wycen
- Tabela: nr wyceny, data, klient, status, kwota (kompaktowy widok)
- Filtry priorytetowe: status + zakres dat
- Filtry rozszerzone: lokalizacja, operator, nr przesyłki, email klienta
- Wyszukiwarka: nr wyceny, nazwisko, email
- Widoczność: operator → tylko swój salon, admin → wszystko

### Moduł 3: Szczegóły wyceny
- Informacje o kliencie
- Lista produktów z deklarowanym i zweryfikowanym stanem
- Porównanie: deklarowane vs zweryfikowane (wykrywanie rozbieżności)
- Zmiana statusu z walidacją (kto może, kiedy, jakie warunki)
- Timeline zmian (audit log)
- Zakładki: produkty, umowy, wysyłka, komunikacja, historia

### Moduł 4: Weryfikacja produktu
- Checklist: stan wizualny, mechaniczny, numer seryjny, firmware
- Ocena 5-10/10 z opisem każdego poziomu
- Zarządzanie akcesoriami (zaznacz obecne/brakujące)
- Automatyczne przeliczanie ceny po zmianie stanu/akcesoriów
- Akcesoria ODEJMOWANE od ceny bazowej (nie dodawane)

### Moduł 5: Umowy
- Generowanie umowy (3 typy × 3 metody podpisu)
- Wybór produktów na umowę (nie wszystkie muszą iść)
- Status umowy: wygenerowana → wysłana → podpisana → zwrócona
- Integracja z e-podpisem (do ustalenia)
- Śledzenie: czy umowa w Verto, czy PCC naliczone

### Moduł 6: Kanban produktów
- Widok tablicowy: kolumny per krok
- Blokery widoczne wizualnie (INDEKS, PZ)
- Drag & drop między krokami
- Filtry: lokalizacja, data, operator
- Podgląd szczegółów produktu z poziomu karty

### Moduł 7: Szybka wycena salonowa (5-krokowy stepper)
- **Krok 1: Produkty** — wyszukaj produkt, oceń stan, zaznacz akcesoria. Obsługa **wielu produktów** w jednej sesji.
- **Krok 2: Cena** — cena odkupu (przelew vs karta). Senior/Admin widzi też cenę sprzedaży i marżę. Senior/Admin może edytować cenę. Operator widzi TYLKO cenę odkupu.
- **Krok 3: Decyzja klienta** — A) niezainteresowany → KONIEC, B) umowa od razu → dalej, C) zostawia do ekspertyzy → dalej
- **Krok 4: Dane klienta** — osoba fizyczna (imię, nazwisko, email, tel, PESEL) lub firma (TYLKO NIP → auto-uzupełnienie z GUS/CEIDG). Konto bankowe wymagane TYLKO przy przelewie + osoba fizyczna.
- **Krok 5: Finalizacja** — numery seryjne wpisywane TERAZ. Generowanie: umowy (os. fiz.) / protokołu pozostawienia / protokołu przyjęcia (firma). Firma: czekamy na ich fakturę.
- Cel: **jak najszybciej**, bez zbędnych kroków

### Moduł 8: Serwis
- Rejestracja zlecenia serwisowego (3 scenariusze)
- Śledzenie statusu naprawy
- Koszty: diagnostyka, naprawa, kto ponosi
- Powiązanie z produktem i wycenią
- Dashboard kosztów serwisowych

### Moduł 9: PCC
- Lista umów podlegających PCC (> 1000 zł)
- Automatyczne oznaczanie miesiąca PCC
- Porównanie: kwota z umowy vs kwota z Verto
- Eksport raportu do księgowości
- Obsługa korekt

### Moduł 10: Komunikacja
- Historia email/SMS per wycena
- Szablony wiadomości (6 typów)
- Integracja z Thulium
- Notatki wewnętrzne (niewidoczne dla klienta)
- Automatyczne powiadomienia (zmiana statusu, przypomnienia)

### Moduł 11: Analityka
- Marżowość per produkt/kategoria/lokalizacja
- Koszty ukryte: serwis, PCC, prowizje Allegro, akcesoria
- Trendy czasowe
- Efektywność operatorów
- Eksport danych

### Moduł 12: Użytkownicy
- CRUD użytkowników
- Przypisanie roli i lokalizacji
- Historia aktywności

### Moduł 13: Ustawienia
- Profil użytkownika
- Preferencje powiadomień
- Konfiguracja lokalizacji

---

## 5. Integracje zewnętrzne

| System | Kierunek | Dane | Priorytet |
|--------|----------|------|-----------|
| **Verto** | ← odczyt | Indeksy, ceny finalne, dokumenty zakupu, PZ, stany magazynowe | Wysoki |
| **Verto** | → zapis | Tworzenie indeksów, PZ, MMK | Wysoki |
| **Sylius** | → zapis | Karty produktów (opis, stan, zdjęcia) | Średni |
| **Baselinker** | → zapis | Aukcje Allegro (cena, opis, kategoria) | Średni |
| **Thulium** | ↔ dwukierunkowa | Wysyłanie/odbiór wiadomości, tickety | Wysoki |
| **Cyfrolog** | ← odczyt | Numery CYF, kody EAN | Niski |
| **DPD API** | → zapis | Zamawianie kurierów, etykiety | Średni |
| **InPost API** | → zapis | Zamawianie przesyłek zwrotnych | Niski |
| **e-Podpis** | → zapis | Weryfikacja tożsamości, podpis umowy | Do ustalenia |

---

## 6. Uprawnienia i role

### Matryca uprawnień

| Akcja | Operator | Senior Operator | Admin |
|-------|----------|-----------------|-------|
| Tworzenie wyceny | ✅ (swój salon) | ✅ (swój salon) | ✅ (wszystkie) |
| Weryfikacja produktu | ✅ | ✅ | ✅ |
| Zmiana statusu wyceny | ✅ (ograniczone) | ✅ | ✅ |
| Ręczna korekta ceny | ❌ | ✅ | ✅ |
| Generowanie umowy | ✅ | ✅ | ✅ |
| Dostęp do PCC | ❌ | ❌ | ✅ |
| Kanban (przesuwanie) | ✅ (wybrane kroki) | ✅ | ✅ |
| Tworzenie indeksu Verto | ❌ | ✅ | ✅ |
| Zarządzanie użytkownikami | ❌ | ❌ | ✅ |
| Analityka / raporty | ❌ | ✅ (podstawowa) | ✅ (pełna) |
| Widoczność lokalizacji | Swój salon | Swój salon | Wszystkie |

---

## 7. User Stories

### CZĘŚĆ I: Odkup

#### Wycena online
- **US-001**: Jako operator, chcę widzieć listę wycen przypisanych do mojego salonu, abym wiedział co mam do zrobienia.
- **US-002**: Jako operator, chcę filtrować wyceny po statusie i dacie, abym szybko znalazł te wymagające mojej uwagi.
- **US-003**: Jako operator, chcę otworzyć szczegóły wyceny i zobaczyć dane klienta, produkty, i aktualny status.

#### Weryfikacja
- **US-010**: Jako operator, chcę porównać stan zadeklarowany przez klienta ze stanem rzeczywistym, abym mógł skorygować cenę.
- **US-011**: Jako operator, chcę zaznaczyć które akcesoria faktycznie są, a których brakuje, aby system przeliczył cenę.
- **US-012**: Jako operator, chcę zobaczyć automatycznie obliczoną nową cenę po zmianie stanu/akcesoriów.
- **US-013**: Jako operator, chcę wysłać klientowi mail z nową propozycją cenową jednym kliknięciem.

#### Umowa
- **US-020**: Jako operator, chcę wybrać które produkty z wyceny idą na umowę (nie wszystkie muszą), abym mógł obsłużyć częściowe odkupy.
- **US-021**: Jako operator, chcę wygenerować umowę z wybranymi produktami i wybraną formą rozliczenia.
- **US-022**: Jako admin, chcę śledzić status podpisu umowy (wygenerowana → wysłana → podpisana).

#### Salon — szybka wycena
- **US-030**: Jako pracownik salonu, chcę szybko wycenić produkt klienta „z ulicy" — wyszukać produkt, ocenić stan, pokazać cenę.
- **US-031**: Jako pracownik salonu, chcę wprowadzić dane klienta i wygenerować umowę w jednym flow, bez przeskakiwania między systemami.
- **US-032**: Jako pracownik salonu, chcę żeby system wiedział w jakim jestem salonie i jak się nazywam, bez ręcznego wpisywania.

### CZĘŚĆ II: Kanban

- **US-040**: Jako operator, chcę widzieć tablicę Kanban z produktami na każdym kroku przygotowania do sprzedaży.
- **US-041**: Jako operator, chcę przenosić produkty między krokami Kanbanu (drag & drop).
- **US-042**: Jako operator, chcę widzieć blokery (Indeks, PZ) wizualnie — żebym wiedział co blokuje postęp.
- **US-043**: Jako admin, chcę widzieć ile produktów czeka na każdym kroku, abym mógł zarządzać zasobami.

### CZĘŚĆ III: Serwis

- **US-050**: Jako operator, chcę oznaczyć produkt jako „wymaga serwisu" podczas weryfikacji, z opisem problemu.
- **US-051**: Jako operator, chcę śledzić status naprawy serwisowej i jej koszty.
- **US-052**: Jako admin, chcę widzieć sumaryczne koszty serwisowe per miesiąc/rok, abym wiedział o ukrytych kosztach.

### PCC

- **US-060**: Jako admin, chcę widzieć listę umów podlegających PCC (> 1000 zł) z podziałem na miesiące.
- **US-061**: Jako admin, chcę eksportować raport PCC do wysłania do księgowości.
- **US-062**: Jako admin, chcę porównać kwotę z umowy z kwotą z Verto, aby wykryć rozbieżności.

### Komunikacja

- **US-070**: Jako operator, chcę wysłać mail do klienta z szablonu (np. „wynik weryfikacji") jednym kliknięciem.
- **US-071**: Jako operator, chcę dodać notatkę wewnętrzną do wyceny, niewidoczną dla klienta.
- **US-072**: Jako operator, chcę widzieć pełną historię komunikacji z klientem w jednym miejscu.

---

*Dokument przygotowany: 19 marca 2026*
*Wersja: 1.0*
