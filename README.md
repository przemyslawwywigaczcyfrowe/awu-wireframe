# AWU — Automatyczna Wycena Używanych

**Cyfrowe.pl** · Panel Operatora

---

## 🔗 Demo prototypu

**[▶ Otwórz interaktywny prototyp](https://przemyslawwywigaczcyfrowe.github.io/awu-wireframe/)**

Login testowy: `admin@cyfrowe.pl` / dowolne hasło

---

## O projekcie

AWU (Automatyczna Wycena Używanych) to wewnętrzne narzędzie do **automatycznej wyceny produktów używanych** na cyfrowe.pl — generuje wyceny, umowy i pliki CSV, obsługując pełny cykl od momentu, gdy klient chce nam sprzedać sprzęt, przez ekspertyzę, umowę, przygotowanie do sprzedaży, aż po wystawienie na stronie i Allegro.

### Problem, który rozwiązujemy

Dziś cały proces opiera się na **6+ arkuszach Google Sheets**, systemie ticketowym OTRS, ręcznym kopiowaniu danych między systemami i papierowych cennikach w salonach. System crashuje się, dane giną, a ukryte koszty (serwis, PCC, prowizje) nie są widoczne w raportach.

### Zakres systemu

```
CZĘŚĆ I: ODKUP                    CZĘŚĆ II: PRZYGOTOWANIE DO SPRZEDAŻY
(wycena → ekspertyza → umowa)     (regał → indeks → zdjęcia → front → Allegro)

┌──────────────────────────┐      ┌──────────────────────────────────────┐
│                          │      │                                      │
│  Klient zgłasza sprzęt   │─────▶│  Produkt przygotowywany do sprzedaży │
│  → weryfikujemy stan     │      │  → indeks → zdjęcia → karta → PZ   │
│  → proponujemy cenę      │      │  → aktywacja na stronie + Allegro    │
│  → podpisujemy umowę     │      │                                      │
│                          │      │                                      │
└──────────┬───────────────┘      └──────────────────────────────────────┘
           │
           │ (w dowolnym momencie)
           ▼
    ┌──────────────┐
    │   SERWIS     │
    │  (naprawa)   │
    └──────────────┘
```

---

## 📚 Dokumentacja

### Dla biznesu
| Dokument | Opis |
|----------|------|
| [**Opis procesu biznesowego**](docs/BIZNES-FLOW.md) | Pełny opis procesu skupu prostym językiem — od wyceny do sprzedaży |
| [**Model produktu — oś systemu**](docs/MODEL-PRODUKTU.md) | Dlaczego produkt (nie wycena) jest najważniejszy, wersjonowanie, historia zmian |
| [**Słownik pojęć**](docs/SLOWNIK.md) | Wszystkie skróty i terminy używane w systemie |
| [**Mapa obecnych narzędzi**](docs/NARZEDZIA-DZIS.md) | Co używamy dziś, jakie są problemy, co zastąpi AWU |

### Dla programistów
| Dokument | Opis |
|----------|------|
| [**Specyfikacja techniczna**](docs/TECH-SPEC.md) | Model danych, statusy, maszyny stanów, integracje |
| [**Model produktu (szczegółowy)**](docs/MODEL-PRODUKTU.md) | Encja Product, ProductVersion, AuditEntry, cykl życia |
| [**User Stories**](docs/USER-STORIES.md) | Kompletny zestaw 35 user stories z kryteriami akceptacji |
| [**Mapa modułów**](docs/MODULY.md) | 13 modułów systemu z zakresem, priorytetami i zależnościami |
| [**Integracje zewnętrzne**](docs/INTEGRACJE.md) | Verto, Sylius, Baselinker, Thulium, Cyfrolog — co, jak, kiedy |
| [**Uprawnienia i role**](docs/UPRAWNIENIA.md) | Matryca uprawnień, role, widoczność danych |

---

## Struktura prototypu

```
awu-wireframe/
├── docs/                    # Pełna dokumentacja
│   ├── BIZNES-FLOW.md       # Opis procesu (biznes)
│   ├── TECH-SPEC.md         # Specyfikacja techniczna
│   ├── USER-STORIES.md      # User stories
│   ├── MODULY.md            # Mapa modułów
│   ├── INTEGRACJE.md        # Integracje zewnętrzne
│   ├── UPRAWNIENIA.md       # Role i uprawnienia
│   ├── SLOWNIK.md           # Słownik pojęć
│   └── NARZEDZIA-DZIS.md    # Obecne narzędzia i bolączki
└── README.md                # Ten plik
```

---

## Role w systemie

| Rola | Kto | Widoczność |
|------|-----|-----------|
| **Operator** | Pracownik salonu | Swój salon |
| **Senior Operator** | Doświadczony pracownik | Swój salon + korekta cen |
| **Admin** | Paweł, Bartek, Ewelina | Wszystkie lokalizacje |

## Lokalizacje

7 salonów (Gdańsk, Katowice, Kraków, Łódź, Poznań, Warszawa Mokotów, Warszawa Wola) + Centrala (Gdańsk, ul. Łostowicka 25A)

---

## Status projektu

- [x] Prototyp interaktywny (wireframe)
- [x] Dokumentacja biznesowa
- [x] Dokumentacja techniczna
- [ ] Rewizja przez biznes (Paweł Ostęp)
- [ ] Rewizja przez programistów
- [ ] Implementacja backendu
- [ ] Integracje z Verto/Sylius/Baselinker

---

*Projekt: marzec 2026*
*Product Owner: Przemysław Wywigacz*
*Ekspert procesowy: Paweł Ostęp*
