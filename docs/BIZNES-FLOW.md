# AWU — Opis procesu skupu produktów używanych
## Dokument dla biznesu | Cyfrowe.pl

> Ten dokument opisuje prostym językiem, jak działa cały proces skupu produktów używanych —
> od momentu, gdy klient chce nam sprzedać sprzęt, aż do wystawienia go na sprzedaż.
> Celem jest wspólne zrozumienie procesu, zanim zaczniemy budować system.

---

## Spis treści

1. [Przegląd procesu — mapa od A do Z](#1-przegląd-procesu)
2. [CZĘŚĆ I: Od wyceny do umowy](#2-część-i-od-wyceny-do-umowy)
3. [CZĘŚĆ II: Od umowy do sprzedaży (Kanban)](#3-część-ii-od-umowy-do-sprzedaży)
4. [CZĘŚĆ III: Pętla serwisowa](#4-część-iii-pętla-serwisowa)
5. [Podatek PCC](#5-podatek-pcc)
6. [Role w systemie](#6-role-w-systemie)
7. [Obecne narzędzia i co je zastąpi](#7-obecne-narzędzia-i-co-je-zastąpi)
8. [Słownik pojęć](#8-słownik-pojęć)

---

## 1. Przegląd procesu

Cały proces skupu dzieli się na **dwie główne części** i jedną **ścieżkę boczną**:

```
CZĘŚĆ I: ODKUP                    CZĘŚĆ II: PRZYGOTOWANIE DO SPRZEDAŻY
(wycena → ekspertyza → umowa)     (regał → indeks → zdjęcia → front → Allegro)

┌──────────────────────────┐      ┌──────────────────────────────────────┐
│                          │      │                                      │
│  Klient zgłasza sprzęt   │─────▶│  Produkt przygotowywany do sprzedaży │
│  → weryfikujemy stan     │      │  → indeks → zdjęcia → karta → PZK   │
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

### Najważniejsza zasada: PRODUKT jest główną osią

Jeden klient może przynieść 4 produkty. Z tych 4:
- 2 mogą iść na umowę od razu
- 1 może zostać odrzucony
- 1 może trafić do serwisu

**Każdy produkt ma własną, niezależną ścieżkę.** Wycena grupuje produkty, ale po weryfikacji każdy idzie swoją drogą.

---

## 2. CZĘŚĆ I: Od wyceny do umowy

### 2.1 Jak powstaje wycena?

Są **dwa scenariusze**:

#### Scenariusz A: Klient online
1. Klient wchodzi na cyfrowe.pl → wybiera produkt → ocenia stan (5-10/10) → zaznacza akcesoria
2. System pokazuje wstępną cenę (przelew vs karta podarunkowa)
3. Klient wybiera formę rozliczenia i sposób dostawy (kurier DPD / salon / InPost)
4. Wycena ważna 7 dni
5. Klient wysyła sprzęt lub przynosi do salonu

#### Scenariusz B: Klient "z ulicy" w salonie
1. Klient przychodzi do salonu z produktem — nie robił wyceny online
2. Pracownik salonu musi szybko: zidentyfikować produkt → ocenić stan → podać cenę
3. Dziś robi to w Excelu/cenniku papierowym — **to główna bolączka**
4. Jeśli klient zainteresowany → pracownik wprowadza dane → generuje umowę → klient podpisuje

### 2.2 Ekspertyza (weryfikacja)

Gdy sprzęt dotrze do nas (kurier/salon), operator sprawdza:
- **Stan wizualny** — czy odpowiada temu, co klient zadeklarował
- **Stan mechaniczny** — czy wszystko działa
- **Numer seryjny** — czy zgadza się z deklaracją
- **Akcesoria** — czy są wszystkie, które klient zaznaczył

#### Co jeśli stan się nie zgadza?

Częsty przypadek: klient zaznaczył 8/10, a po ekspertyzie jest 6/10.

**Wtedy:**
1. System oblicza nową cenę (niższą)
2. Automatyczny mail do klienta z nową propozycją
3. Operator może dodatkowo zadzwonić
4. Klient decyduje: akceptuje nową cenę lub odrzuca (zwrot sprzętu)

#### Jak działają akcesoria i cena?

Akcesoria są **odejmowane** od ceny bazowej — nie dodawane. Jeśli klient zaznaczy, że ma pudełko, dekielek i pasek, a po ekspertyzie brakuje paska → cena spada o wartość paska.

### 2.3 Statusy wyceny

Wycena przechodzi przez następujące etapy:

| Status | Co to znaczy |
|--------|-------------|
| **Nowa** | Wycena złożona (online lub salon) |
| **W trakcie weryfikacji** | Operator sprawdza sprzęt |
| **Zweryfikowana** | Sprawdzone, gotowe do wysłania oferty |
| **Oczekuje na decyzję** | Klient dostał ofertę, decyduje |
| **Zaakceptowana** | Klient przyjął ofertę |
| **Odrzucona** | Klient odrzucił ofertę |
| **Umowa podpisana** | Umowa zawarta |
| **Realizacja finansowa** | Przelew/karta w trakcie |
| **Zakończona** | Rozliczone, zamknięte |
| **Zwrot do klienta** | Sprzęt wraca do klienta |
| **Przekazana do centrali** | Eskalacja — salon nie radzi sobie |

### 2.4 Umowa

Trzy rodzaje umów:
- **Umowa kupna-sprzedaży** — osoba prywatna
- **Umowa kupna-sprzedaży z VAT** — osoba prywatna z VAT
- **Faktura VAT** — firma (klient wysyła swoją fakturę)

Trzy sposoby podpisania:
- **W salonie** — klient podpisuje przy biurku
- **Online** — weryfikacja tożsamości przez serwis elektroniczny (do ustalenia)
- **Kurierem** — dokumenty wysyłane do klienta, podpisuje przy kurierze, kurier odsyła

### 2.5 Formy płatności

| Forma | Opis |
|-------|------|
| Przelew bankowy | Na konto klienta |
| Karta podarunkowa | Do wykorzystania w sklepie (cena wyższa o ~5-10%) |
| Gotówka | W salonie |
| Przelew odwrotny | Klient kupuje nowy sprzęt, wysyła używany |
| Karta z pełną opieką | Karta podarunkowa + obsługa zamówienia |

---

## 3. CZĘŚĆ II: Od umowy do sprzedaży

Po podpisaniu umowy każdy produkt przechodzi przez **Kanban przygotowania do sprzedaży**. To 10 kroków:

### Przegląd kroków

```
NOWY → REGAŁ → INDEKS* → KGM → SESJA → KARTA → PZK* → FRONT → ALLEGRO
                  ⚠️                               ⚠️
               BLOKER #1                         BLOKER #2
```

### Szczegóły kroków

#### Krok 1: NOWY
Produkt pojawia się na liście po zatwierdzeniu umowy. Każdego ranka Paweł sprawdza co nowego wjechało, weryfikuje duplikaty, uzupełnia brakujące dane (który salon, czyje imię i nazwisko).

#### Krok 2: REGAŁ
Przygotowanie fizyczne produktu:
- Wyczyszczenie
- Reset do ustawień fabrycznych
- Uzupełnienie brakujących elementów (np. ładowarka)
- Odłożenie na odpowiednie miejsce na regale (podzielone na marki)
- Zasada: **fotograf ma nie myśleć** — produkt na regale musi być w 100% gotowy

#### Krok 3: INDEKS ⚠️ BLOKER #1
Utworzenie indeksu produktu w systemie Verto (ERP). **Bez indeksu nic dalej się nie wydarzy.**
- Otwieramy istniejący indeks produktu używanego → zmieniamy 4-6 pól → zapisujemy nowy indeks
- Ten krok blokuje: kartę gwarancyjną, sesję zdjęciową, kartę produktu

#### Krok 4: KGM WYDRUK
- Wydruk karty gwarancyjnej (6 miesięcy)
- Nadanie kodu EAN (numer Cyfrolog)
- Wydruk naklejki z pełną nazwą produktu (jeśli brak oryginalnego pudełka)
- Fizyczne dołożenie karty i naklejek do pudełka na regale

#### Krok 5: SESJA + PAKOWANIE
- Sfotografowanie produktu (w tym dodatkowe zdjęcia usterek, np. rysa na obudowie)
- Profesjonalne spakowanie, plomby, taśma, naklejki EAN
- Rozliczanie pracy: tygodniówki (numer tygodnia w roku)

#### Krok 6: KARTA PRODUKTU
- Uzupełnienie listingu w Sylius (sklep internetowy)
- Karta tworzy się automatycznie z indeksu, ale jest **pusta** — trzeba ręcznie wpisać: stan, przebieg, opis

#### Krok 7: PZK ⚠️ BLOKER #2
- Przyjęcie towaru na stan w Verto (PZ-ka)
- **Bez PZK nie można uruchomić produktu na froncie**
- Wymaga fizycznej umowy/faktury w centrali
- Numer wewnętrzny: 03/RACH (seria 03 = używane)

> **Ważne:** Karta produktu (krok 6) i PZK (krok 7) są od siebie **niezależne**. Mogą być robione w dowolnej kolejności, ale **oba** muszą być gotowe, żeby przejść dalej.

#### Krok 8: DO URUCHOMIENIA (FRONT)
- Oba warunki spełnione (PZK + karta produktu)
- MMK — przesunięcie towaru z magazynu "używany" na magazyn główny
- Aktywacja produktu na stronie cyfrowe.pl

#### Krok 9: ALLEGRO
- Wystawienie na Allegro przez Baselinker
- Cena na Allegro **wyższa** niż na stronie (prowizje Allegro)
- Ręczna zmiana: cena, marża, kategoria, szablon
- Jeśli produkt sprzeda się na stronie zanim trafi na Allegro → status "sprzedane"

#### Krok 10: OLX
- Zarządzane przez Kamilę
- Forma reklamy, nie główny kanał sprzedaży

---

## 4. CZĘŚĆ III: Pętla serwisowa

Serwis może pojawić się **w dowolnym momencie** życia produktu. Są 3 scenariusze:

### Scenariusz 1: Serwis podczas ekspertyzy

Ekspertyza wykrywa problem (np. luz na pierścieniu obiektywu).

1. Informujemy klienta: „3 produkty OK, 1 wymaga diagnostyki — koszt 69 zł. Zgadzasz się?"
2. Jeśli tak → umowa na 3 produkty, ten 1 w zawieszeniu
3. Produkt jedzie do serwisu
4. Serwis wycenia naprawę (np. 250 zł)
5. Nowa oferta dla klienta: „Zamiast 750 zł dostaniesz 500 zł" (koszt naprawy odliczony)
6. Klient decyduje: akceptacja lub rezygnacja
7. Jeśli akceptacja → nowa wycena → nowa umowa na ten 1 produkt

### Scenariusz 2: Serwis po zakupie (przed sprzedażą)

Podczas przygotowania do sprzedaży odkryto usterkę. Koszt naprawy ponosi Cyfrowe.pl.

### Scenariusz 3: Serwis po sprzedaży (gwarancja)

Klient kupił produkt i zgłasza reklamację. Gwarancja 6 miesięcy. Koszt ponosi Cyfrowe.pl.

### Dlaczego serwis jest ważny?

Koszty serwisowe to **ukryty koszt** — prowizje Allegro, PCC, naprawy, faktury na akcesoria.
W zeszłym roku było to **kilkaset tysięcy złotych**, których nikt nie widział w raportach marżowości.

---

## 5. Podatek PCC

### Zasady
- PCC naliczany od umów powyżej **1000 zł** (wartość finalna z Verto)
- Termin: do **7. dnia następnego miesiąca**
- Raport wysyłany do księgowości raz w miesiącu

### Problemy
- Cena na umowie może różnić się od ceny finalnej w Verto (np. odkup pracowniczy, zmiana formy płatności)
- Umowa może nie zdążyć dotrzeć do Verto przed terminem PCC
- Klient może zrezygnować po wygenerowaniu umowy — trzeba anulować PCC
- **Źródło prawdy**: cena z dokumentu zakupu w Verto, NIE z wygenerowanej umowy

---

## 6. Role w systemie

| Rola | Kto to jest | Co może |
|------|-------------|---------|
| **Operator** | Pracownik salonu | Tworzyć wyceny, weryfikować sprzęt, obsługiwać klientów. Widzi tylko swój salon. |
| **Senior Operator** | Doświadczony pracownik | Wszystko co Operator + ręczna korekta cen |
| **Admin** | Paweł, Bartek, Ewelina | Wszystko + zarządzanie użytkownikami, dostęp do wszystkich lokalizacji, PCC, analityka |

### Lokalizacje
- 7 salonów: Gdańsk, Katowice, Kraków, Łódź, Poznań, Warszawa Mokotów, Warszawa Wola
- 1 centrala: Gdańsk, ul. Łostowicka 25A

---

## 7. Obecne narzędzia i co je zastąpi

| Obecne narzędzie | Do czego służy | Co je zastąpi w AWU |
|------------------|----------------|---------------------|
| Google Sheets (ekspertyzy) | Wpisy operatorów salonów | → Moduł wycen + weryfikacja |
| Google Sheets (Kanban) | Przygotowanie do sprzedaży | → Moduł Kanban produktów |
| Google Sheets (umowy) | Rejestr umów | → Moduł umów |
| Google Sheets (cennik) | Wycena do odsprzedaży | → Automatyczny cennik w module produktów |
| Google Sheets (serwis) | Tracking napraw | → Moduł serwisowy |
| Google Sheets (PCC) | Obliczanie podatku PCC | → Moduł PCC |
| OTRS / Thulium | Komunikacja z klientem | → Moduł komunikacji (integracja z Thulium) |
| Ręczne arkusze | Wszystko powyżej | → Jeden system AWU |

**Integracje** (nie zastępujemy, ale łączymy się):
- **Verto** — indeksy, PZK, ceny, dokumenty zakupu
- **Sylius** — karty produktów na stronie
- **Baselinker** — aukcje Allegro
- **Cyfrolog** — numery CYF / kody EAN

---

## 8. Słownik pojęć

| Pojęcie | Znaczenie |
|---------|-----------|
| **AWU** | System do zarządzania skupem produktów używanych (ten który budujemy) |
| **Ekspertyza** | Fizyczna weryfikacja stanu sprzętu przez operatora |
| **Indeks** | Numer identyfikacyjny produktu w systemie Verto |
| **PZK** | Przyjęcie Zewnętrzne Kupno — formalne przyjęcie towaru na stan w Verto |
| **MMK** | Przesunięcie Magazynowe — zmiana magazynu z "używany" na główny |
| **KGM** | Karta Gwarancyjna — indywidualny dokument gwarancji dla każdego produktu |
| **PCC** | Podatek od Czynności Cywilnoprawnych — naliczany od umów > 1000 zł |
| **Verto** | System ERP Cyfrowe.pl — źródło prawdy o cenach i stanach magazynowych |
| **Sylius** | Platforma e-commerce — sklep internetowy cyfrowe.pl |
| **Baselinker** | Narzędzie do zarządzania aukcjami na Allegro |
| **Cyfrolog** | System nadawania unikalnych numerów CYF i kodów EAN produktom |
| **Thulium** | Nowy system komunikacji z klientem (zastępuje OTRS) |
| **CSV** | Plik z danymi ekspertyzy/umowy — generowany przez obecny system AWU |
| **RMA** | Return Merchandise Authorization — zlecenie serwisowe |
| **Refurb** | Produkt odnowiony — kupowany hurtowo od firm, nie od osób prywatnych |

---

*Dokument przygotowany: 19 marca 2026*
*Źródło: spotkanie Przemysław Wywigacz × Paweł Ostęp, 18 marca 2026 + analiza procesu*
