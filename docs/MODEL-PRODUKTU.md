# Model produktu — oś całego systemu AWU

> Produkt jest najważniejszą encją w systemie. Nie wycena, nie umowa — PRODUKT.
> Bo to produkt przechodzi z rąk do rąk, zmienia stan, cenę, właściciela.

---

## Dlaczego produkt, a nie wycena?

Klient może przynieść 4 produkty w jednej wycenie. Co może się z nimi stać:

```
Wycena #2345 (4 produkty)
│
├── Produkt A (Canon EOS R5)
│   ├── Ma cenę w bazie → operator potwierdza → idzie na umowę
│   └── ✅ Standardowa ścieżka
│
├── Produkt B (Obiektyw Nikon 14-24mm)
│   ├── Nie ma w bazie → oznaczony "wycena indywidualna"
│   ├── Przekazany do centrali → Senior Operator wycenia ręcznie
│   ├── ⏳ Inna osoba, inny czas
│   └── 💡 Przekazanie do centrali dotyczy TEGO produktu, nie całej wyceny
│
├── Produkt C (Lampa Godox V1)
│   ├── Wykryto usterkę podczas weryfikacji
│   ├── → Serwis (diagnostyka 69 zł)
│   ├── → Wraca z kosztorysem naprawy
│   ├── → Nowa wycena, nowa umowa na ten 1 produkt
│   └── 🔧 Zupełnie inna ścieżka
│
└── Produkt D (Adapter Canon EF-EOS R)
    ├── Klient zadeklarował 8/10, operator stwierdza 5/10
    ├── Nowa cena: z 400 zł na 180 zł
    ├── Klient odrzuca → zwrot produktu
    └── ❌ Odrzucony
```

**Wycena** to tylko "koperta" grupująca produkty na wejściu. Po weryfikacji każdy produkt żyje własnym życiem.

---

## Cykl życia produktu

```
WYCENA (wejście)
    │
    ▼
WERYFIKACJA (stan, akcesoria, cena)
    │
    ├── Produkt z ceną w bazie → Operator potwierdza
    ├── Produkt bez ceny / indywidualny → Senior Operator / Admin wycenia ręcznie
    └── Produkt z usterką → Serwis
    │
    ▼
NEGOCJACJA (jeśli rozbieżność)
    │
    ├── Klient akceptuje → UMOWA
    └── Klient odrzuca → ZWROT
    │
    ▼
UMOWA (kontrakt na wybrane produkty)
    │
    ▼
KANBAN (przygotowanie do sprzedaży)
    │
    Nowy → Regał → Indeks → KGM → Sesja → Karta → PZ → Front → Allegro
    │
    ▼
SPRZEDANY
```

---

## Wersjonowanie — pełna historia zmian

Każdy produkt przechowuje **pełną historię** wszystkich zmian. Nic nie jest nadpisywane — każda edycja tworzy nową wersję.

### Co jest wersjonowane:

#### Ceny
```
Wersja 1: Cena z bazy         → 4500 zł (przelew) / 4900 zł (karta)     [automat]
Wersja 2: Po weryfikacji       → 3800 zł / 4200 zł                       [operator: Jan Kowalski]
Wersja 3: Korekta Sr. Operatora → 4000 zł / 4400 zł                      [senior: Ewelina Kostro]
Wersja 4: Finalna (Verto)      → 3950 zł                                 [system: import z Verto]
```

#### Stan / ocena
```
Wersja 1: Deklaracja klienta    → 8/10, akcesoria: pudełko, ładowarka, pasek
Wersja 2: Weryfikacja operatora → 6/10, brak paska, rysa na obudowie
```

#### Status
```
2026-03-15 10:30 → Nowa                    [system: wycena online]
2026-03-16 09:15 → W trakcie weryfikacji   [Jan Kowalski, Salon Kraków]
2026-03-16 11:45 → Zweryfikowana           [Jan Kowalski]
2026-03-16 11:46 → Oczekuje na decyzję     [system: mail wysłany]
2026-03-18 14:20 → Zaakceptowana           [system: klient kliknął "akceptuję"]
2026-03-18 14:25 → Umowa podpisana         [Ewelina Kostro]
```

#### Przypisanie operatora
```
2026-03-16 09:15 → Przypisany: Jan Kowalski (Salon Kraków)
2026-03-16 12:00 → Przekazany: Centrala (produkt wymaga wyceny indywidualnej)
2026-03-17 08:30 → Przypisany: Paweł Ostęp (Centrala Gdańsk)
```

### Kto może edytować cenę?

| Sytuacja | Kto może wycenić |
|----------|-----------------|
| Produkt ma cenę w bazie (standardowy) | Operator — potwierdza cenę z bazy |
| Produkt ma cenę w bazie, ale trzeba skorygować | Senior Operator / Admin — ręczna korekta |
| Produkt nie ma ceny w bazie | Senior Operator / Admin — wycena od zera |
| Produkt oznaczony "wycena indywidualna" | Tylko Senior Operator / Admin |
| Cena finalna (po serwisie, po negocjacjach) | Senior Operator / Admin |

---

## Model danych — Product (encja główna)

```
Product {
  id                    : UUID
  appraisalId           : FK → Appraisal         // wycena źródłowa (koperta)
  contractId            : FK → Contract | null    // umowa (jeśli zakontraktowany)

  // Identyfikacja
  catalogProductId      : FK → CatalogProduct | null  // z katalogu (jeśli znany)
  virtualProductName    : string | null               // ręczna nazwa (jeśli spoza katalogu)
  vertoIndex            : string | null               // indeks Verto
  serialNumber          : string | null
  cyfrologNumber        : string | null
  eanCode               : string | null

  // Aktualny stan (najnowsza wersja)
  currentVersion        : FK → ProductVersion         // wskaźnik na aktualną wersję

  // Status w procesie
  appraisalProductStatus : enum                       // status w ramach wyceny
  kanbanStep             : KanbanStep | null           // krok w Kanbanie (po umowie)
  serviceTicketId        : FK → ServiceTicket | null   // jeśli w serwisie

  // Przypisanie
  currentOperatorId      : FK → User | null
  locationId             : FK → Location

  // Flagi
  requiresExpertPricing  : boolean                     // czy wymaga wyceny eksperta
  isInService            : boolean
  hasDiscrepancy         : boolean

  createdAt              : datetime
  updatedAt              : datetime
}
```

### ProductVersion (wersja — każda zmiana tworzy nowy wpis)

```
ProductVersion {
  id                    : UUID
  productId             : FK → Product
  versionNumber         : int                         // 1, 2, 3...

  // Stan
  rating                : int (5-10)
  accessories           : AccessoryItem[]
  accessoryComment      : string | null

  // Ceny
  priceTransfer         : decimal | null
  priceGiftCard         : decimal | null
  priceResale           : decimal | null
  priceAllegro          : decimal | null

  // Kto i dlaczego
  changedBy             : FK → User
  changeReason          : string                      // "Weryfikacja operatora", "Korekta ceny", "Import z Verto"
  changeType            : enum (DEKLARACJA_KLIENTA, WERYFIKACJA, KOREKTA_CENY, WYCENA_INDYWIDUALNA, IMPORT_VERTO, SERWIS)

  // Notatki
  internalNote          : string | null

  createdAt             : datetime
}
```

### AuditEntry (log operacji)

```
AuditEntry {
  id                    : UUID
  entityType            : enum (PRODUCT, APPRAISAL, CONTRACT)
  entityId              : UUID

  action                : string                      // "status_change", "price_edit", "operator_assign", "email_sent"
  previousValue         : string | null
  newValue              : string | null

  performedBy           : FK → User
  performedAt           : datetime

  description           : string                      // "Jan Kowalski zmienił ocenę z 8/10 na 6/10"
}
```

---

## Kluczowe zasady

1. **Produkt jest osią** — wycena to koperta, umowa to formalizacja, ale produkt jest tym, co się weryfikuje, wycenia, przygotowuje i sprzedaje

2. **Każda zmiana tworzy nową wersję** — cena, stan, akcesoria — nic nie jest nadpisywane, zawsze jest historia

3. **Różni operatorzy na różnych etapach** — produkt może przechodzić przez wiele rąk, każda osoba zostawia ślad w historii

4. **Wycena indywidualna** — produkty bez ceny w bazie lub oznaczone jako wymagające eksperta muszą być wycenione przez Senior Operatora / Admina

5. **Pełny audit log** — każdy kontakt z klientem, każda zmiana statusu, każda edycja ceny — wszystko zapisane z kto, kiedy, dlaczego

6. **Przekazanie do centrali = per produkt** — w wycenie z 4 produktami, 1 może zostać przekazany do centrali, a 3 pozostałe idą dalej normalnie. Eskalacja nigdy nie blokuje całej wyceny.

---

*Dokument aktualizowany: 19 marca 2026*
*Kluczowe źródło: ustalenia z Product Ownerem, 19 marca 2026*
