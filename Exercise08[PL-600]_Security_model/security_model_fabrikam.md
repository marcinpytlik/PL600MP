# Fabrikam Robotics – Security model (Power Platform / Dataverse)

## 1. Kontekst rozwiązania

Istniejące aplikacje:
- **Sales Central** – model-driven app dla sprzedawców i menedżerów sprzedaży.
- **Canvas app – Reception** – check-in gości w showroomie.
- **Power Pages portal** – dla odwiedzających do składania requestów wizyt.

Cele bezpieczeństwa:
- Ochrona przed „dzikimi” konektorami w Power Automate.
- Separacja danych sprzedażowych od recepcji.
- Poprawne role bezpieczeństwa i Business Units w Dataverse.
- Ograniczenie widoczności danych w portalu do „moich” rekordów.
- Dopasowanie do licencjonowania (D365 Sales, Power Apps per app).

---

## 2. Rogue connectors – jak uspokoić security

### Działania techniczne

1. **DLP (Data Loss Prevention) policies**
   - Utworzenie dedykowanego środowiska (environment) dla rozwiązania Fabrikam.
   - DLP policy dla tego środowiska:
     - Grupa **Business**:
       - Dataverse, Office 365, Outlook, Teams, SharePoint, zaakceptowane LOB/API.
     - Grupa **Blocked**:
       - wszystkie niezatwierdzone 3rd party connectors (Twitter, Dropbox, Slack itd.).
   - Efekt: Flow nie może łączyć Dataverse z blokowanymi usługami.

2. **Kontrola tworzenia środowisk i flow**
   - Ograniczenie prawa do tworzenia **nowych environmentów**.
   - Ograniczenie self-service Power Automate:
     - tylko określone grupy AD mogą tworzyć flow w prodowym środowisku.

3. **Edukacja i governance**
   - Krótka polityka:
     - „Dane z Dataverse mogą być wysyłane tylko do zatwierdzonych systemów”.
   - Sesja szkoleniowa dla Sales:
     - jak zgłaszać potrzeby integracyjne,
     - czego nie robić w swoich flow.

---

## 3. Tylko sales managers mogą zatwierdzać i stosować rabaty

Wymóg:
- Pole `Discount` (currency),
- Pole `Approver` (lookup),
- Sprzedawca nie może sam sobie zatwierdzić rabatu.

### Podejście

1. **Security roles na poziomie pól**
   - Rola **Salesperson** (small/large):
     - `Discount` → Read, brak Write.
   - Rola **Sales Manager**:
     - `Discount` → Read + Write,
     - `Approver` → Read + Write.

2. **Walidacja `Approver` ≠ właściciel rekordu**
   - Logika (plug-in / business rule / flow):
     - jeśli `Approver = Owner` → błąd walidacji.
   - Dodatkowa walidacja:
     - `Approver` musi mieć rolę Sales Manager.

3. **Proces akceptacji rabatu (opcjonalnie)**
   - Pola:
     - `RequestedDiscount` – wypełnia sprzedawca,
     - `Discount` – finalna zaakceptowana wartość.
   - Flow:
     - trigger: zmiana `RequestedDiscount`,
     - wyślij approval do `Approver`,
     - po akceptacji:
       - przepisz `RequestedDiscount` → `Discount`,
       - ustaw `DiscountApprovedBy` / `Approver`.

---

## 4. Kontrola dostępu do Power Apps

### Sales Central (model-driven)

- Udostępniamy tylko użytkownikom z licencją **Dynamics 365 Sales**.
- Role:
  - `Salesperson Small Deals`,
  - `Salesperson Large Deals`,
  - `Sales Manager Small Deals`,
  - `Sales Manager Large Deals`.
- Brak roli = brak dostępu do danych i aplikacji.

### Canvas app – Reception

- Udostępniona tylko grupie AD `Showroom Reception`.
- Rola Dataverse `Receptionist`:
  - dostęp do tabel: Visit, Parking, ograniczony Contact, Building (Read),
  - brak dostępu do Lead, Opportunity, Discount.
- Licencja: **Power Apps per app** – przypisana do tej aplikacji.

### Power Pages portal

- Dostęp wyłącznie dla zewnętrznych odwiedzających:
  - Web Role: `Visitor`,
  - Table Permissions (szczegóły w sekcji portalowej).

---

## 5. Licencje Power Apps per app dla recepcji – co to zmienia?

- Recepcja:
  - ma licencję tylko na **konkretną aplikację canvas**.
- Technicznie:
  - nie share’ujemy im żadnych innych aplikacji w tym environment,
  - rola `Receptionist` ma wąski zakres uprawnień.
- Efekt:
  - nawet gdyby ktoś podał link do Sales Central, brak:
    - licencji D365,
    - share’u aplikacji,
    - odpowiedniej roli bezpieczeństwa.

---

## 6. Jakie role bezpieczeństwa w Dataverse?

Propozycja ról:

1. **Salesperson Small Deals**
   - Lead, Opportunity:
     - Read/Write na poziomie **Business Unit**.
   - Visit (jeśli potrzebne): własne + BU.
   - Brak Write do `Discount`.

2. **Salesperson Large Deals**
   - Lead, Opportunity:
     - głównie **own + team**.
   - Brak Write do `Discount`.

3. **Sales Manager Small Deals**
   - Read/Write Lead/Opportunity na poziomie BU.
   - Full control nad `Discount` i `Approver`.
   - Dostęp do raportów.

4. **Sales Manager Large Deals**
   - Jak wyżej, ale dla BU Large Deals.

5. **Receptionist**
   - Read/Write:
     - Visit,
     - Parking,
     - Contact (tylko wybrane pola + ograniczenia rekordowe),
     - Building (Read).
   - Brak dostępu do danych sprzedażowych (Lead, Opportunity, Discount).

6. **Marketing**
   - Głównie Read do danych sprzedażowych i wizyt (zakres do ustalenia).

7. **Operations / Showroom Admin**
   - Zarządzanie budynkami, slotami, widoczność wszystkich Visit.

8. **Portal Visitor Web Role** (w Power Pages)
   - Konfiguracja Table Permissions dla Visit/VisitRequest.

---

## 7. Jakie Business Units?

Struktura organizacyjna:

- Sales
  - Small Deals
  - Large Deals
  - Wholesale
- Marketing
- Operations
  - Manufacturing
  - Showroom
  - Shipping
- Customer Service

### Proponowana struktura BU:

**Root BU – Fabrikam**

- **Sales BU**
  - **Sales – Small Deals BU**
  - **Sales – Large Deals BU**
  - **Sales – Wholesale BU**
- **Marketing BU**
- **Operations BU**
  - Showroom (opcjonalnie osobny BU)
  - Manufacturing
  - Shipping
- **Customer Service BU**

Dla naszego scenariusza kluczowe:

- **Sales – Small Deals BU**:
  - wszyscy mają BU-level access do danych sprzedażowych.
- **Sales – Large Deals BU**:
  - domyślnie own + team,
  - manager z widocznością całego BU.
- **Showroom / Reception**:
  - osobny BU lub w ramach Operations BU,
  - wyraźnie odseparowany od Sales BU.

---

## 8. Portal – widoczność tylko własnych wizyt

Chcemy, by odwiedzający widział tylko **swoje** wizyty / requesty.

### Konfiguracja Power Pages

1. **Relacja danych**
   - Tabela `Visit` (lub `VisitRequest`) ma lookup do:
     - `Contact` reprezentującego użytkownika portalu.

2. **Table Permission**
   - Tworzymy Table Permission dla `Visit`:
     - Scope: **Contact**,
     - Parent table: `Contact`,
     - Relationship: `Contact–Visit`.
   - Przypisujemy je do Web Role `Visitor`.

Efekt:

- Po zalogowaniu użytkownik portalu widzi tylko te rekordy `Visit`, które są powiązane z jego `Contact`.

---

## 9. Widoczność danych – Small vs Large Deals

Wymóg:
- Small sales department: widzi **wszystkie** dane small.
- Large deals: widzą **tylko swoje** dane, chyba że są w zespole dla dużego deala.

### Small Deals

- BU: **Sales – Small Deals BU**.
- Rola `Salesperson Small Deals`:
  - Lead/Opportunity – uprawnienia na poziomie **BU** (Read/Write BU).
- Efekt:
  - każdy sprzedawca small widzi i może pracować na danych całego zespołu.

### Large Deals

- BU: **Sales – Large Deals BU**.
- Rola `Salesperson Large Deals`:
  - Lead/Opportunity – uprawnienia:
    - Read: Own + Team,
    - Write: Own + Team.
- **Access Teams / Owner Teams**:
  - Dla bardzo dużych deali:
    - tworzymy Access Team `Deal_X_Team`,
    - dodajemy do niego wybranych sprzedawców,
    - nadajemy wspólny dostęp do konkretnego Opportunity/Lead.
  - Alternatywnie: Owner Team jako właściciel rekordu.

### Managerowie

- Rola `Sales Manager Large Deals`:
  - Read/Write na poziomie BU,
  - widoczność wszystkich deali w Large Deals,
  - możliwość modyfikacji discountów.

### Hierarchiczny model bezpieczeństwa (opcjonalnie)

- Można włączyć **Manager hierarchy security**:
  - manager widzi rekordy swoich podwładnych,
  - ale handlowcy między sobą nie mają wzajemnej pełnej widoczności, jeśli nie są w jednym BU lub zespole.

---

## 10. Podsumowanie

Proponowany model bezpieczeństwa:

- Chroni przed „rogue connectors”:
  - DLP policies,
  - kontrola środowisk i uprawnień do Power Automate.
- Gwarantuje, że:
  - **tylko** Sales Managers zatwierdzają rabaty,
  - recepcja nie ma dostępu do danych sprzedażowych,
  - użytkownicy portalu widzą tylko swoje rekordy.
- Wykorzystuje:
  - sensownie zdefiniowane **security roles**,
  - **Business Units** zgodne ze strukturą organizacyjną,
  - **Access Teams** do elastycznego dzielenia się dużymi dealami.
- Jest spójny z licencjonowaniem:
  - D365 Sales dla Sales Central,
  - Power Apps per app dla aplikacji recepcji.

Ten model jest łatwo rozszerzalny o kolejne moduły (np. inspekcje, incydenty) bez łamania zasady najmniejszych możliwych uprawnień.
