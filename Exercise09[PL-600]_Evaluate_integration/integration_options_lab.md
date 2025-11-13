# Evaluate Integration Options – Integration Lab

## Scenariusz 1 – Property Environmental Hazard data

### Wymaganie
Klient ma licencjonowane dane o zagrożeniach środowiskowych dla nieruchomości.  
W aplikacji na każdej nieruchomości musi być widoczne:
- czy istnieje hazard,
- jaki to hazard,
- kiedy ostatnio był raportowany.

### Fakty
- 555 000 nieruchomości w danych.
- Około 1000 rekordów zmienionych lub nowych tygodniowo.
- Aktualizacja 1x w tygodniu – plik JSON do pobrania z URL.
- Klucz: `County + ParcelID`.
- 20 kolumn danych.

### Proponowana architektura integracji (inbound)

#### Model danych
- Tabela **Property**:
  - `County`, `ParcelID`,
  - `HasHazard` (bool),
  - `HazardType` (tekst/option set),
  - `HazardLastReportedOn` (datetime).
- Tabela pomocnicza (opcjonalnie) **HazardImport**:
  - `County`,
  - `ParcelId`,
  - `HazardType`,
  - `ReportedOn`,
  - `Status` (Pending/Processed/Error/NotFound),
  - `Checksum` (opcjonalnie).

#### Integracja – Opcja A: Power Automate (batch ETL)
- **Scheduled cloud flow** (1x w tygodniu):
  - `HTTP` – pobierz plik JSON z URL.
  - `Parse JSON` – rozbij na rekordy.
  - Dla każdego elementu:
    - znajdź `Property` po `County + ParcelID`,
    - jeśli istnieje:
      - zaktualizuj `HasHazard`, `HazardType`, `HazardLastReportedOn`,
    - jeśli nie istnieje:
      - opcjonalnie utwórz rekord w `HazardImport` lub w `Property` (w zależności od wymagań).
- Przy potrzebie lepszej skalowalności:
  - Flow 1 ładuje dane do `HazardImport`,
  - Flow 2 (batch processor) aktualizuje `Property` w paczkach (np. 1000 rekordów na run).

#### Integracja – Opcja B: Dataflows (Power Query)
- Dataflow:
  - źródło: JSON z URL,
  - transformacja w Power Query,
  - load do:
    - `HazardImport` albo
    - bezpośrednio do `Property` (po kluczu `County + ParcelID`).
- Harmonogram: 1x w tygodniu.

#### Dodatkowe elementy
- Logowanie importu:
  - tabela `HazardImportLog`: data importu, liczba rekordów, błędy.
- Idempotencja:
  - pełny snapshot co tydzień – prostota procesu ważniejsza niż różnicowe aktualizacje.

---

## Scenariusz 2 – Access to Property Tax Authority

### Wymaganie
W model-driven app na formularzu nieruchomości użytkownik musi zobaczyć:
- bieżącą kwotę podatku,
- kwotę ostatniej płatności.

### Ograniczenia
- Dane mogą być przechowywane lokalnie max **24h**.
- Dane dostępne przez **REST API z OAuth**.
- API ma okno niedostępności w weekendy (2h).
- Dane aktualizują się w sposób ciągły.
- Aplikacja jest **model-driven**.

### Proponowana architektura

#### Model danych
- Tabela **Property**.
- Tabela pomocnicza **PropertyTaxCache**:
  - `Property` (lookup),
  - `CurrentTaxAmount`,
  - `LastPaymentAmount`,
  - `LastSyncedOn` (datetime),
  - `Status` (OK/Error).

#### Integracja – cache 24h

1. **Custom connector**:
   - Definiuje request do API podatkowego + obsługę OAuth.

2. **On-demand odświeżanie z poziomu formularza:**
   - Na formularzu Property:
     - PCF control lub komanda na wstążce → wywołuje flow/akcję.
   - Logika:
     - jeśli `LastSyncedOn` < 24h:
       - pokaż dane z `PropertyTaxCache`,
     - jeśli brak danych lub `LastSyncedOn >= 24h`:
       - wywołaj Power Automate / Dataverse action:
         - użyj custom connectora, pobierz dane z API,
         - zaktualizuj rekord `PropertyTaxCache`,
         - odśwież formularz.

3. **Opcjonalny prefetch (np. nocny):**
   - Scheduled flow:
     - pobiera listę nieruchomości „ważnych” na dzisiaj,
     - aktualizuje ich cache poza godzinami szczytu.

#### Obsługa błędów i downtime
- Jeśli API jest niedostępne:
  - zapis `Status = Error` w `PropertyTaxCache`,
  - zapis do tabeli `IntegrationErrorLog`.
- Flow monitorujący błędy:
  - jeśli pojawi się błąd integracji → powiadomienie dla managera (Teams/email).

---

## Scenariusz 3 – Outsourced customer support (webhook outbound)

### Wymaganie
Nowi i zmienieni klienci mają być wysyłani do firmy zewnętrznej:
- wyjściowe dane w JSON (schema dostarczona),
- endpoint: webhook URL (HTTP POST),
- błędy muszą być zgłaszane managerowi.

### Proponowana architektura (outbound push)

#### Model danych
- Tabela **Customer** (Contact/Account).
- Tabela **CustomerSyncLog**:
  - `Customer` (lookup),
  - `Direction` (Outbound),
  - `Status` (Pending/Sent/Error),
  - `Attempts`,
  - `LastAttemptOn`,
  - `ErrorMessage`.

#### Flow – „Sync Customer to Outsourced Support”
1. **Trigger:**
   - Dataverse: `When a row is added, modified` (Customer),
   - warunek: np. tylko `SyncToSupport = Yes`.

2. **Akcje:**
   - Mapowanie pól → JSON wg schema.
   - `HTTP POST` do webhook URL.
   - Jeśli sukces (2xx):
     - zapis `CustomerSyncLog` z `Status = Sent`.
   - Jeśli błąd (4xx/5xx/timeout):
     - zapis `CustomerSyncLog` z `Status = Error`, `ErrorMessage`.

3. **Retry / niezawodność:**
   - Drugi flow „Retry Customer Sync” (Recurrence):
     - wybiera `CustomerSyncLog` z `Status = Error` i `Attempts < N`,
     - próbuje ponownie.
   - Po przekroczeniu limitu prób:
     - zachowuje `Status = Error`,
     - nie próbuje dalej (trzeba interwencji człowieka).

4. **Powiadomienie managera:**
   - Flow monitorujący:
     - jeśli w ostatniej godzinie powstało X nowych rekordów z `Status = Error`,
     - wysyła powiadomienie (Teams/email) do managera.

---

## Scenariusz 4 – Customer Referral (Woodgrove → Contoso)

### Wymaganie
Inna dywizja (Woodgrove) chce przesyłać referral’e klientów przez API:

Proces:
1. Utwórz Account, jeśli jeszcze nie istnieje (po nazwie).
2. Utwórz Opportunity.
3. Przypisz Opportunity do zespołu sprzedaży wg regionu.
4. Utwórz Task dla zespołu Account Research.

Dane:
- Wszyscy użytkownicy w jednej sieci.
- Około 25 leadów tygodniowo (mały wolumen).

### Proponowana architektura

#### Opcja A – Azure API + Dataverse Web API

1. **Endpoint API** (np. Azure Function / API Management):
   - `POST /referrals`
   - Payload JSON: dane klienta, region, dodatkowe informacje.

2. Logika w funkcji:
   - Zapytanie Dataverse:
     - szukaj `Account` po nazwie (i ewentualnie NIP/TaxID).
   - Jeśli konto nie istnieje:
     - `Create Account`.
   - `Create Opportunity`:
     - powiąż z Account.
   - Na podstawie `Region`:
     - przypisz Owner:
       - do właściwego Owner Team / sales team.
   - `Create Task`:
     - `Regarding` = Account lub Opportunity,
     - przypisany do zespołu Account Research.

3. Autoryzacja:
   - Woodgrove → API (klucz/OAuth).
   - API → Dataverse (Service Principal).

#### Opcja B – Power Automate HTTP Request trigger

1. **Power Automate flow** z triggerem:
   - `When a HTTP request is received`.
2. Flow:
   - `Parse JSON` z payloadu.
   - `List rows` Accounts – znajdź istniejące konto.
   - `Create row` Account, jeśli brak.
   - `Create row` Opportunity.
   - Zielone przypisanie wg regionu:
     - lookup do tabeli Region→Team,
     - `Assign` na odpowiedni team/user.
   - `Create row` Task (account research).

Przy 25 leadach tygodniowo to jest całkowicie wystarczające i proste w utrzymaniu.

---

## Podsumowanie – wzorce integracji

- **Scenariusz 1 (hazard JSON)**  
  - Batch pull (weekly) z pliku → aktualizacja Dataverse (Dataflows / Power Automate).

- **Scenariusz 2 (tax API)**  
  - Pull on demand z cache TTL 24h + custom connector; monitorowanie błędów i downtime.

- **Scenariusz 3 (outsourced support)**  
  - Outbound push z Dataverse trigger → webhook POST, z logowaniem, retry i alertami.

- **Scenariusz 4 (referrals API)**  
  - Inbound push: wystawienie API (Azure/Flow HTTP trigger), które tworzy Account, Opportunity, Task i przypisuje je wg regionu.

Te cztery scenariusze razem pokazują sensowny miks:
- ETL/batch (Scenariusz 1),
- near-real-time pull z cache (Scenariusz 2),
- event-driven push na zewnątrz (Scenariusz 3),
- event-driven inbound API (Scenariusz 4).
