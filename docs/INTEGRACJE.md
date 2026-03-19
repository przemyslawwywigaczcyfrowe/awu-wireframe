# Integracje zewnętrzne — AWU

> Opis każdej integracji z systemem zewnętrznym: co, jak, kiedy i w jakim kierunku.

---

## Mapa integracji

```
                         ┌─────────────┐
                    ┌───▶│   Verto     │◀───┐
                    │    │   (ERP)     │    │
                    │    └─────────────┘    │
                    │                       │
              ┌─────┴─────┐          ┌──────┴──────┐
              │   AWU     │          │  Cyfrolog   │
              │  ADMIN    │          │  (numery)   │
              └─────┬─────┘          └─────────────┘
                    │
         ┌──────────┼──────────┐
         │          │          │
    ┌────▼────┐ ┌───▼────┐ ┌──▼───────┐
    │ Sylius  │ │Thulium │ │Baselinker│
    │ (sklep) │ │ (mail) │ │(Allegro) │
    └─────────┘ └────────┘ └──────────┘
```

---

## 1. Verto (ERP)

**Priorytet:** Wysoki — bez Verto system nie będzie działać w pełni

### Kierunek: AWU ← Verto (odczyt)

| Dane | Częstotliwość | Opis |
|------|---------------|------|
| Indeksy produktów | Na żądanie | Wyszukiwanie istniejących indeksów do klonowania |
| Ceny finalne | Dziennie / na żądanie | Cena z dokumentu zakupu (źródło prawdy dla PCC) |
| Stany magazynowe | Na żądanie | Czy produkt jest na stanie, w jakim magazynie |
| Dokumenty zakupu PZ | Na żądanie | Status: czy produkt został formalnie przyjęty |

### Kierunek: AWU → Verto (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Nowy indeks produktu | Krok "Indeks" w Kanbanie | Tworzenie indeksu na podstawie klonowania istniejącego |
| Dokument PZ | Krok "PZ" w Kanbanie | Formalne przyjęcie towaru na stan (seria 03) |
| Przesunięcie MMK | Krok "Front" w Kanbanie | Z magazynu "używany" na magazyn główny |

### Uwagi techniczne
- Dziś dane z Verto importowane do Sheets raz na tydzień — system musi mieć dostęp w czasie rzeczywistym
- Verto ma API (do zbadania zakres i limity)
- Kluczowe: seria numeracji 03/RACH dla produktów używanych
- Indeks to klonowanie istniejącego indeksu z 4-6 zmianami pól

---

## 2. Sylius (sklep internetowy)

**Priorytet:** Średni

### Kierunek: AWU → Sylius (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Karta produktu | Krok "Karta produktu" w Kanbanie | Uzupełnienie pustej karty: stan, przebieg, opis, zdjęcia |
| Aktywacja produktu | Krok "Front" w Kanbanie | Włączenie widoczności produktu na stronie |
| Cena frontowa | Krok "Front" | Ustawienie ceny sprzedaży na cyfrowe.pl |

### Uwagi techniczne
- Indeks Verto automatycznie tworzy pustą kartę w Sylius — AWU musi ją uzupełnić
- Sylius API dostępne (REST/GraphQL — do zbadania)
- Zdjęcia z sesji zdjęciowej muszą być uploadowane do Sylius
- Opis stanu i przebieg mogą być generowane automatycznie z danych weryfikacji

---

## 3. Baselinker → Allegro

**Priorytet:** Średni

### Kierunek: AWU → Baselinker (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Aukcja Allegro | Krok "Allegro" w Kanbanie | Tworzenie aukcji z danymi produktu |
| Cena Allegro | Krok "Allegro" | Cena wyższa niż frontowa (prowizje Allegro) |
| Opis i zdjęcia | Krok "Allegro" | Z Sylius + ręczne poprawki |

### Uwagi techniczne
- Baselinker służy jako most do Allegro — nie bezpośrednia integracja z Allegro
- Szablon aukcji: część danych pobierana z Sylius, część ręcznie
- Cena Allegro = cena frontowa + marża na prowizje (do konfiguracji)
- OLX zarządzane osobno (Kamila) — nie wymaga integracji w MVP

---

## 4. Thulium (komunikacja z klientem)

**Priorytet:** Wysoki

### Kierunek: AWU ↔ Thulium (dwukierunkowa)

| Dane | Kierunek | Opis |
|------|----------|------|
| Wysyłanie maili | AWU → Thulium | Szablony mailowe z poziomu wyceny |
| Wysyłanie SMS | AWU → Thulium | Powiadomienia SMS |
| Odbieranie odpowiedzi | Thulium → AWU | Odpowiedzi klientów widoczne w historii |
| Tickety | AWU ↔ Thulium | Powiązanie ticketu Thulium z wycenią |

### Uwagi techniczne
- Thulium zastępuje OTRS (wdrożenie w toku)
- Docelowo: komunikacja zarządzana z poziomu AWU, Thulium jako silnik wysyłki
- Kolejki AWU: AWU, Indywidualna, Sprzęt w drodze, Przypomnienia, Odkupione salon, Serwis, Umowa e-podpis
- Notatki wewnętrzne NIE wysyłane do Thulium
- Automatyczne powiadomienia: zmiana statusu, przypomnienie po 6 dniach

---

## 5. Cyfrolog (numeracja CYF/EAN)

**Priorytet:** Niski

### Kierunek: AWU ← Cyfrolog (odczyt)

| Dane | Kiedy | Opis |
|------|-------|------|
| Numer CYF | Krok "KGM" w Kanbanie | Unikalny numer produktu |
| Kod EAN | Krok "KGM" w Kanbanie | Kod kreskowy do naklejki |

### Uwagi techniczne
- Dziś każdy produkt wprowadzany do Cyfrolog pojedynczo
- Bulk processing byłby pożądany ale API może nie wspierać
- Alternatywa: wbudowane generowanie numerów w AWU (jeśli Cyfrolog nie ma API)

---

## 6. DPD API (kurier)

**Priorytet:** Średni

### Kierunek: AWU → DPD (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Zamówienie kuriera | Wycena online — klient wybiera kuriera | Etykieta zwrotna |
| Śledzenie przesyłki | Po zamówieniu | Status dostawy w czasie rzeczywistym |

### Uwagi techniczne
- DPD API dla etykiet zwrotnych
- Tracking number powiązany z wycenią
- Statusy: Zamówiona → Odebrana → W transporcie → Dostarczona

---

## 7. InPost API (paczkomaty)

**Priorytet:** Niski

### Kierunek: AWU → InPost (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Zamówienie przesyłki zwrotnej | Zwrot produktu do klienta (InPost) | Zwrot po odrzuceniu oferty |

### Uwagi techniczne
- Tylko dla zwrotów (klient → InPost paczkomat)
- Opcjonalnie: przesyłka od klienta przez paczkomat

---

## 8. E-podpis (do ustalenia)

**Priorytet:** Do ustalenia
**Kandydaci:** Autenti, mSzafir, DocuSign, SimplySign

### Kierunek: AWU → Serwis e-podpisu (zapis)

| Dane | Kiedy | Opis |
|------|-------|------|
| Dokument umowy | Generowanie umowy online | Wysłanie do podpisu |
| Status podpisu | Po wysłaniu | Czy klient podpisał |
| Podpisany dokument | Po podpisaniu | PDF z podpisem elektronicznym |

### Uwagi techniczne
- Weryfikacja tożsamości klienta (KYC)
- Opcja dla klientów, którzy nie chcą podpisywać przy kurierze
- Wymaga analizy kosztów i regulacji prawnych

---

## Priorytetyzacja integracji

| Faza | Integracja | Uzasadnienie |
|------|-----------|-------------|
| **MVP** | Verto (odczyt cen, indeksów) | Bez tego brak cennika i źródła prawdy |
| **MVP** | Thulium (wysyłka maili) | Komunikacja z klientem to core workflow |
| **Faza 2** | Verto (zapis: indeksy, PZ, MMK) | Automatyzacja Kanbanu |
| **Faza 2** | Sylius (karty produktów) | Automatyzacja frontu |
| **Faza 2** | DPD (etykiety, tracking) | Obsługa logistyki |
| **Faza 3** | Baselinker (Allegro) | Automatyzacja marketplace |
| **Faza 3** | Cyfrolog | Automatyzacja numeracji |
| **Faza 3** | InPost | Obsługa zwrotów |
| **Backlog** | E-podpis | Wymaga analizy i decyzji biznesowej |

---

*Dokument aktualizowany: 19 marca 2026*
