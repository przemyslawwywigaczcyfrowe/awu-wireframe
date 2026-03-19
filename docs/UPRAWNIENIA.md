# Role i uprawnienia — AWU

> Matryca uprawnień, opis ról i zasady widoczności danych.

---

## Role w systemie

### Operator
**Kto:** Pracownik salonu
**Gdzie:** Przypisany do jednego salonu
**Widzi:** Tylko dane ze swojej lokalizacji

Podstawowa rola operacyjna. Obsługuje klientów, weryfikuje produkty, generuje umowy. Nie może ręcznie korygować cen — musi przekazać produkt do wyceny przez osobę z odpowiednimi uprawnieniami.

### Senior Operator
**Kto:** Doświadczony pracownik z wiedzą cenową
**Gdzie:** Przypisany do jednego salonu lub centrali
**Widzi:** Tylko dane ze swojej lokalizacji + dodatkowe funkcje

Może ręcznie wyceniać produkty, które nie mają ceny w bazie lub są oznaczone jako "wycena indywidualna". Ma wiedzę ekspercką pozwalającą na korektę cen.

### Admin
**Kto:** Paweł Ostęp, Bartek, Ewelina, Ania Kudra
**Gdzie:** Centrala Gdańsk (ale widzi wszystko)
**Widzi:** Dane ze wszystkich lokalizacji

Pełny dostęp do systemu. Zarządza użytkownikami, ma dostęp do PCC, analityki, może anulować umowy.

---

## Zasada widoczności cen i marży

> **Cena sprzedaży i marża widoczna TYLKO dla Senior Operator i Admin.**
> Operator w salonie widzi TYLKO cenę odkupu (ile może zaproponować klientowi).
> Dotyczy to zarówno szybkiej wyceny salonowej, jak i wszystkich innych widoków z cenami.

---

## Matryca uprawnień

### Część I: Odkup

| Akcja | Operator | Senior Operator | Admin |
|-------|----------|-----------------|-------|
| Przeglądanie listy wycen | ✅ (swój salon) | ✅ (swój salon) | ✅ (wszystkie) |
| Otwieranie szczegółów wyceny | ✅ (swój salon) | ✅ (swój salon) | ✅ (wszystkie) |
| Tworzenie wyceny salonowej | ✅ | ✅ | ✅ |
| Weryfikacja produktu (stan, akcesoria) | ✅ | ✅ | ✅ |
| Wycena produktu z bazy (cena automatyczna) | ✅ | ✅ | ✅ |
| Wycena produktu ręczna (spoza bazy / indywidualna) | ❌ | ✅ | ✅ |
| Korekta ceny produktu | ❌ | ✅ | ✅ |
| Zmiana statusu wyceny | ✅ (ograniczone*) | ✅ | ✅ |
| Przekazanie produktu do centrali (do wyceny eksperta) | ✅ | ✅ | ✅ |
| Podgląd ceny sprzedaży / marży | ❌ | ✅ | ✅ |
| Generowanie umowy | ✅ | ✅ | ✅ |
| Anulowanie umowy | ❌ | ❌ | ✅ |
| Wysyłanie maili do klienta | ✅ | ✅ | ✅ |

*Ograniczenia operatora przy zmianie statusu:*
- Może: Nowa → W trakcie weryfikacji → Zweryfikowana
- Nie może: Anulowanie, Realizacja finansowa, Zakończona (wymaga Admin)

### Część II: Kanban (produkty)

| Akcja | Operator | Senior Operator | Admin |
|-------|----------|-----------------|-------|
| Przeglądanie Kanbanu | ✅ | ✅ | ✅ |
| Przenoszenie produktów (drag & drop) | ✅ (wybrane kroki) | ✅ | ✅ |
| Tworzenie indeksu Verto | ❌ | ✅ | ✅ |
| Ręczne dodawanie do Kanbanu (B2B) | ❌ | ❌ | ✅ |
| Zatwierdzanie PZ (przyjęcie do magazynu) | ❌ | ✅ | ✅ |
| Aktywacja produktu na froncie | ❌ | ✅ | ✅ |

### Moduły wspólne

| Akcja | Operator | Senior Operator | Admin |
|-------|----------|-----------------|-------|
| Dashboard — widok | ✅ (swój salon) | ✅ (swój salon) | ✅ (pełny) |
| Serwis — rejestracja zlecenia | ✅ | ✅ | ✅ |
| Serwis — koszty | ❌ | ✅ (podgląd) | ✅ (pełny) |
| PCC — przegląd i raporty | ❌ | ❌ | ✅ |
| Analityka — podstawowa | ❌ | ✅ | ✅ |
| Analityka — pełna (marże, koszty) | ❌ | ❌ | ✅ |
| Zarządzanie użytkownikami | ❌ | ❌ | ✅ |
| Ustawienia — własny profil | ✅ | ✅ | ✅ |
| Ustawienia — lokalizacje | ❌ | ❌ | ✅ |

---

## Kluczowa zasada: produkt może być obsługiwany przez różne osoby

W ramach jednej wyceny (np. klient przynosi 4 produkty):
- **Produkt A** — ma cenę w bazie → operator salonu może sam potwierdzić cenę
- **Produkt B** — nie ma w bazie / oznaczony jako "wycena indywidualna" → wymaga Senior Operatora lub Admina
- **Produkt C** — wymaga serwisu → idzie do diagnostyki, obsługa przechodzi na inną osobę
- **Produkt D** — przekazany do centrali → wycena przez eksperta w centrali

**Każdy produkt w wycenie może mieć innego operatora na każdym etapie.**

---

## Widoczność danych — zasady

| Rola | Widzi wyceny | Widzi produkty w Kanbanie | Widzi PCC | Widzi analitykę |
|------|-------------|--------------------------|-----------|-----------------|
| Operator | Swój salon | Swoje / przypisane | ❌ | ❌ |
| Senior Operator | Swój salon | Swoje / przypisane | ❌ | Podstawowa |
| Admin | Wszystkie | Wszystkie | ✅ | Pełna |

---

## Kolejka e-podpis (dostęp ograniczony)

Podpisywanie umów elektronicznie (e-podpis) jest dostępne tylko dla wybranych osób:
- Paweł Ostęp
- Bartek
- Ewelina
- Ania Kudra

Inni operatorzy mogą generować umowy, ale podpis elektroniczny wymaga osoby z tej listy.

---

*Dokument aktualizowany: 19 marca 2026*
