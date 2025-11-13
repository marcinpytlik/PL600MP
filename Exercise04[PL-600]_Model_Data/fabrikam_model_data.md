# Fabrikam Robotics – model danych (propozycja)

## Założenia

Budujemy rozwiązanie w oparciu o **Dataverse / Dynamics 365**. Chcemy:

- śledzić **wizyty** w showroomie i części produkcyjnej,
- łączyć je z **procesem sprzedażowym** (tam, gdzie ma to sens),
- obsłużyć **gości**, **zdjęcia**, **zgody/waivery**, **tracking urządzeń**,
- dać **marketingowi** raporty konwersji po wizycie.

---

## Proponowany model danych

### Główne tabele / encje

1. **Contact** (Common Data Model)
   - Reprezentuje osobę – potencjalnego klienta, partnera itp.
   - Dla „primary visitor” korzystamy z *Contact*, nie tworzymy nowego bytu.

2. **Account** (CDM, opcjonalnie)
   - Firma / organizacja powiązana z wizytą i procesem sprzedaży.

3. **Opportunity** (CDM)
   - Standardowa encja sprzedażowa:
     - `Topic`, `EstimatedRevenue`, `CloseDate`, `Status` itd.
   - Każda wizyta sprzedażowa może być:
     - powiązana 1-do-1 z Opportunity, albo
     - powiązana 1-do-wielu (jedna szansa sprzedaży, wiele wizyt).

4. **Visit** (nowa tabela)
   - Reprezentuje **pojedynczą wizytę** w showroomie/zakładzie.
   - Kluczowe pola:
     - `VisitId`
     - `PrimaryVisitor` (lookup do **Contact**)
     - `VisitDateTimeStart`, `VisitDateTimeEnd`
     - `VisitType` (option set: *Sales*, *Tourist/Fun*)
     - `ReservationNumber`
     - `RelatedOpportunity` (lookup do **Opportunity**, opcjonalny – tylko dla wizyt sprzedażowych)
     - `Location` (np. Showroom / Manufacturing / Both)

5. **VisitGuest** (nowa tabela)
   - Reprezentuje **gościa** powiązanego z wizytą.
   - Pola:
     - `VisitGuestId`
     - `Visit` (lookup do **Visit**)
     - `Name`, `Email`, `Phone` (opcjonalnie)
   - Alternatywnie goście mogą być osobnymi Contact, ale na potrzeby ćwiczenia prosta tabela w zupełności wystarczy.

6. **VisitorPhoto** (nowa tabela lub kolumna Image/File w Visit)
   - Każda wizyta → przynajmniej jedno zdjęcie primary visitor.
   - Pola (wariant tabeli):
     - `VisitorPhotoId`
     - `Visit` (lookup)
     - `Photo` (kolumna typu **Image** albo **File**)
     - `TakenOn` (datetime)

7. **WaiverAcceptance** (nowa tabela)
   - Każdorazowe podpisanie zrzeczenia odpowiedzialności.
   - Pola:
     - `WaiverAcceptanceId`
     - `Visit` (lookup do **Visit**)
     - `Visitor` (lookup do **Contact**)
     - `AcceptedOn` (datetime)
     - `Signature` (File/Image – np. rysunek z podpisem lub PDF)
     - `WaiverVersion` (tekst / numer dokumentu z treścią zgody)

8. **TrackingDevice** (nowa tabela)
   - Fizyczne urządzenie przypisywane do głównego odwiedzającego.
   - Pola:
     - `TrackingDeviceId`
     - `DeviceIdentifier` (np. numer seryjny)
     - `IsActive` (bool)

9. **VisitTrackingAssignment** (nowa tabela)
   - Łączy **Visit** z **TrackingDevice**.
   - Pola:
     - `VisitTrackingAssignmentId`
     - `Visit` (lookup)
     - `TrackingDevice` (lookup)
     - `AssignedOn`, `ReturnedOn`

10. **TrackingEvent** (nowa tabela lub tabela wirtualna)
    - Pojedynczy odczyt lokalizacji z urządzenia.
    - Pola:
      - `TrackingEventId`
      - `VisitTrackingAssignment` (lookup)
      - `Timestamp`
      - `LocationZone` i/lub `Coordinates`
    - Technicznie może to być:
      - fizyczna tabela zasilana wsadem/bulk importem, lub
      - **Virtual Table** wskazująca bezpośrednio na chmurę producenta urządzeń.

---

## Odpowiedzi na pytania z ćwiczenia

### 1. Jak przechowuję zdjęcia odwiedzających?

Wymóg mówi: „Each visitor must have a photo taken upon arrival and associated with their visit” – czyli **zdjęcie per wizyta**, nie jedno globalne na kontakt.

Dwa warianty:

**Wariant A – prostszy (na lab):**

- W tabeli **Visit** dodaję kolumnę typu **Image** – `PrimaryVisitorPhoto`.
- Dodaję też `PhotoTakenOn` (datetime).
- Wystarczy, jeśli zakładamy jedno zdjęcie na wizytę.

**Wariant B – bardziej elastyczny:**

- Tworzę osobną tabelę **VisitorPhoto**:
  - `Visit` (lookup),
  - `Photo` (Image/File),
  - `TakenOn`.
- Umożliwia trzymanie wielu zdjęć powiązanych z jedną wizytą (np. powtórka zdjęcia).

**Uzasadnienie:**

> Zdjęcie jest powiązane z konkretną wizytą (kontekst bezpieczeństwa i audytu), więc przechowujemy je na poziomie Visit lub osobnej tabeli powiązanej z Visit, a nie tylko na encji Contact.

---

### 2. Jak przechowuję waiver i podpis?

Wymóg: „Each visitor must sign a waiver of liability **each time they visit**, and you must store their signature and date time of acceptance.”

Rozwiązanie:

- Tworzę tabelę **WaiverAcceptance**:
  - `Visit` – konkretna wizyta,
  - `Visitor` – kto podpisał (Contact),
  - `AcceptedOn` – data i czas akceptacji,
  - `Signature` – kolumna File (skan/obraz/PDF),
  - `WaiverVersion` – pozwala udowodnić, jaką treść ktoś akceptował.
- Dzięki temu mamy pełny **ślad audytowy**: kto, kiedy, jaką wersję dokumentu podpisał.

---

### 3. Jak umożliwiłem oglądanie danych trackingowych w interfejsie sprzedaży?

Wymóg: „You must allow for the tracking data to be viewed in the sales process user interface used by the sales staff.”

Założenie: handlowiec pracuje głównie na **Opportunity** i powiązanych wizytach.

**Kroki:**

1. **Relacje danych**
   - **Visit** ma lookup `RelatedOpportunity` do **Opportunity**.
   - **VisitTrackingAssignment** łączy `Visit` ↔ `TrackingDevice`.
   - **TrackingEvent** jest powiązany z `VisitTrackingAssignment`.

2. **UI dla sprzedaży**
   - Na formularzu **Opportunity**:
     - Sub-grid z listą powiązanych **Visits**.
   - Na formularzu **Visit**:
     - sub-grid z **TrackingEvents**, lub
     - osadzony raport **Power BI** z wizualizacją ścieżki/heatmapy.

3. **Dostęp do danych z chmury urządzeń**
   - Jeśli dane trackingowe pozostają w zewnętrznej chmurze:
     - używam **Virtual Tables** dla `TrackingEvent`, lub
     - embeduję **Power BI report** oparty bezpośrednio na API urządzeń.

**Podsumowanie:**

> Dane trackingowe są dostępne dla handlowca przez powiązaną encję Visit, widoczną z poziomu Opportunity, z osadzonym sub-gridem lub raportem Power BI.

---

### 4. Czy użyłem czegoś z Common Data Model?

Tak, wykorzystuję standardowe encje CDM:

- **Contact** – główny byt osoby odwiedzającej.
- **Account** – powiązana firma (dla wizyt biznesowych).
- **Opportunity** – proces sprzedaży, do którego przypinamy wizyty.

To pozwala:

- korzystać z gotowych procesów sprzedażowych,
- raportować spójnie w całej organizacji,
- unikać „wynajdywania koła” dla podstawowych bytów typu klient/szansa sprzedaży.

---

### 5. Jak wsparłem potrzeby marketingu?

Wymóg:  
„Marketing has asked to be able to view visits by day/month/quarter along with statistics on closing of sales after a visit.”

Czyli marketing chce:

- liczbę wizyt w czasie (dzień/miesiąc/kwartał),
- statystyki **konwersji** – ile sprzedaży po wizycie,
- powiązanie wizyty z wynikiem sprzedaży.

**Model danych:**

- Tabela **Visit** – zawiera datę rozpoczęcia wizyty (`VisitDateTimeStart`) i typ wizyty.
- Tabela **Opportunity** – zawiera status, kwotę i datę zamknięcia.
- **Relacja:** `Visit.RelatedOpportunity` → `Opportunity.OpportunityId`.

**Raportowanie (Power BI):**

- Tworzę model Power BI nad Dataverse:
  - tabele: `Visit`, `Opportunity`, ewentualnie kalendarz `Date`.
- Miary:
  - `VisitsCount` – liczba wizyt,
  - `ClosedOpportunitiesAfterVisit` – liczba szans wygranych po wizycie,
  - `ConversionRate = ClosedOpportunitiesAfterVisit / VisitsCount`,
  - segmentacja po `VisitType`, dacie (dzień/miesiąc/kwartał).

**Wizualizacje dla marketingu:**

- liczba wizyt per dzień/miesiąc/kwartał,
- współczynnik konwersji po wizycie,
- porównanie wizyt typu *Sales* vs *Fun*,
- ewentualnie heatmapa „które miesiące/tury wizyt konwertują najlepiej”.

---

## Podsumowanie

W zaproponowanym modelu:

- każda **wizyta** ma jednoznaczny kontekst:
  - kto przyszedł (Contact),
  - kiedy,
  - z jakim celem (Sales / Fun),
  - jaki waiver podpisał,
  - jakie zdjęcie wykonano,
  - jakim urządzeniem była śledzona,
  - czy powiązała się z **Opportunity**.
- dane trackingowe są dostępne w interfejsie sprzedaży,
- marketing otrzymuje pełen obraz: od wizyty, przez tracking, po domknięcie sprzedaży.

