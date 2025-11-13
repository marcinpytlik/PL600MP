# Evaluate Requirements for First-Party Apps

## Dynamics 365 Sales

### 1. Sales managers need to see parent account details while viewing a contact

**Rozwiązanie:**
- Na formularzu **Contact** dodać **Quick View Form** encji **Account**, oparte na relacji Parent Account.
- Quick View pokaże kluczowe pola konta bez opuszczania formularza kontaktu.

**Typ:** OOTB konfiguracja formularza (Quick View Form).

---

### 2. Preferred pricing tylko na określony czas, potem powrót do standardu

**Rozwiązanie:**
- Użyć:
  - standardowego **Product Catalog** z cennikami (Price Lists),
  - lub dodatkowej encji **Pricing Agreement** z polami: `StartDate`, `EndDate`, `PreferredPrice`, `IsActive`.
- Logika:
  - Business rule / Power Automate / plug-in:
    - przy tworzeniu/aktualizacji Opportunity/Quote:
      - jeśli bieżąca data w zakresie umowy → stosuj preferred pricing,
      - jeśli nie → używaj standardowego cennika.

**Typ:** OOTB (price lists) + lekka logika (workflow/flow/plug-in).

---

### 3. Product configurator dla produktów (kolor, rozmiar, komponenty) + istniejąca WPF .NET

**Rozwiązanie – podejście docelowe:**
- Wykorzystać **Product Catalog** w D365 Sales:
  - Product families,
  - Product bundles,
  - Product relationships (upsell/cross-sell),
  - ewentualnie wbudowane lub third-party **CPQ**.
- Zastąpić WPF konfigurator rozwiązaniem opartym na standardowych produktach.

**Rozwiązanie – podejście pragmatyczne:**
- Zintegrować istniejącą WPF:
  - wywołanie konfiguratora z poziomu formy (link/przycisk),
  - zapis wybranej konfiguracji do D365 przez Web API / Custom Action.

**Typ:** kombinacja pierwszo-stronnych funkcji (Product Catalog / CPQ) + ewentualna integracja legacy WPF.

---

### 4. Użytkownicy oczekują, że formularz Opportunity będzie pokazywał/ukrywał kolumny na podstawie ich wyborów kolumn w widoku

**Komentarz:**
- Widok (grid) i formularz to dwa różne artefakty.
- OOTB nie ma mechanizmu: „jeśli użytkownik dodał kolumnę X do widoku, pokaż ją na formularzu”.

**Rozwiązanie:**
- Edukacja użytkowników: formy konfigurujemy centralnie.
- Użyć:
  - **różnych formularzy** dla różnych ról/scenariuszy (Form Designer),
  - lub **Business Rules** / JavaScript do show/hide sekcji/pól na podstawie:
    - wartości pól,
    - roli użytkownika.

**Typ:** konfiguracja formularza + edukacja; brak OOTB powiązania „view → form”.

---

### 5. Wyświetlanie w czasie rzeczywistym informacji o „top current events” konta na formularzu Account

**Jeśli events są w Dataverse (np. Appointments, Tasks, custom „AccountEvent”):**
- Dodać:
  - **timeline** lub
  - **sub-grid** powiązanej encji na Account.

**Jeśli events są z zewnętrznego systemu:**
- Opcje:
  - **Virtual table** dla „AccountEvent” → sub-grid na formie,
  - **PCF control** pobierające dane z API,
  - lub kafelek **Power BI** osadzony na formularzu Account.

**Typ:** OOTB (timeline/sub-grid) lub integracja (virtual table/PCF/Power BI).

---

### 6. Opportunity > $1M → zmiana właściciela na zespół + powiadomienie

**Rozwiązanie:**
- **Real-time workflow / Power Automate / plug-in**:
  - Trigger: on create Opportunity (lub on update Estimated Revenue).
  - Warunek: `EstimatedRevenue > 1,000,000`.
  - Akcje:
    - `Assign` Opportunity do odpowiedniego **Owner Team** (np. high-value team).
    - Wyślij email/Teams do pierwotnego ownera z informacją o zmianie.

**Typ:** automatyzacja (workflow/flow) na bazie OOTB funkcji.

---

### 7. Wyświetlanie bieżącego etapu (stage) leada w gridzie Lead

**Rozwiązanie:**
- Użyć **Business Process Flow** (Lead-to-Opportunity).
- W encji Lead dostępne jest pole „Process Stage” powiązane z BPF.
- Dodać kolumnę BPF Stage do widoku Lead (grid).

**Jeśli etap jest niestandardowy:**
- Dodać pole „CurrentStageName” i aktualizować je workflow/flow, ale najczęściej wystarczy OOTB.

**Typ:** OOTB + konfiguracja widoków.

---

## Dynamics 365 Customer Service

### 1. Brak pre-purchased tickets → agent sprzedaje support bez wychodzenia z Case

**Rozwiązanie:**
- Użyć **Entitlements** (umowy wsparcia / pulle incydentów).
- Na formularzu **Case**:
  - dodać przycisk lub **quick create form** do stworzenia nowego Entitlement,
  - ewentualnie użyć embedded **canvas app** na formie Case („sprzedaj pakiet wsparcia”).
- Agent pracuje cały czas na tym samym Case, tworząc powiązany Entitlement.

**Typ:** OOTB (Entitlements + quick create) + ewentualny embedded canvas.

---

### 2. Gdy preferred customer skończył pulę incydentów, oferta sprzedaży kolejnych

**Rozwiązanie:**
- **Power Automate / workflow**:
  - Trigger: liczba pozostałych incydentów w Entitlement spada do 0 (lub poniżej progu).
  - Akcje:
    - utwórz Task dla account managera,
    - wyślij email / powiadomienie,
    - opcjonalnie: aktywuj journey w Marketing/Customer Insights do kampanii odnowienia.

**Typ:** automatyzacja (flow/workflow).

---

### 3. Użytkownicy nie mogą korzystać z aplikacji na urządzeniach mobilnych

**Rozwiązanie (poza samym D365):**
- **Azure AD Conditional Access / MDM**:
  - blokować logowanie do D365 z urządzeń mobilnych,
  - lub wymagać zgodnych urządzeń zarządzanych (compliant).
- W samym D365 nie ma opcji „disable mobile” per app, więc robimy to na poziomie tożsamości/urządzenia.

**Typ:** Security / Conditional Access, nie funkcja samej aplikacji.

---

### 4. Case on-site → dispatch do kwalifikowanego field resource

**Rozwiązanie:**
- Integracja **Customer Service + Field Service**:
  - przy Case typu „on-site”:
    - przycisk/automatyzacja tworzy **Work Order** w Field Service,
    - przenosi: klienta, lokalizację, opis problemu.
- W Field Service:
  - używamy **Resource Scheduling / Schedule Board**:
    - dopasowanie kwalifikacji (skills),
    - wybór dostępnego technika.

**Typ:** pierwszo-stronna integracja z D365 Field Service (OOTB).

---

### 5. Personal chart top agenta → udostępnić wszystkim, by widzieli własne open cases od początku tygodnia

**Rozwiązanie:**
- Administrator lub customizer:
  - otwiera personal chart agenta,
  - robi „Save As” → **system chart**,
  - ustawia filtr `Owner = Current User`.
- Następnie:
  - przypisuje system chart do odpowiednich ról (Support Agent).

**Typ:** OOTB – konwersja personal chart → system chart.

---

### 6. Wyświetlanie bieżącego etapu Case w gridzie Case

**Rozwiązanie:**
- Użyć **Case Business Process Flow**.
- Dodać kolumnę procesu/etapu (BPF Stage) do widoku Case.

**Typ:** OOTB + konfiguracja widoku.

---

### 7. Case form – show/hide sections jak w Opportunity form

**Komentarz:**
- Nie istnieje automatyczne dziedziczenie ustawień między formami Case i Opportunity.
- Możemy replikować **zachowanie** (nie konfigurację).

**Rozwiązanie:**
- Na Case:
  - użyć **Business Rules** do show/hide sekcji/pól na podstawie:
    - typu sprawy,
    - priorytetu,
    - statusu,
    - ról użytkownika.
  - dla bardziej zaawansowanej logiki – JavaScript lub PCF.

**Typ:** konfiguracja formularza + ewentualny kod JS/PCF.

---

## Dynamics 365 Field Service

### 1. Po nowym szkoleniu zasoby mają automatycznie aktualizowane skills

**Rozwiązanie:**
- Model:
  - użyć **Resource Characteristics** (skills) w Field Service.
- Integracja z systemem szkoleń (LMS):
  - gdy użytkownik zalicza kurs:
    - LMS wywołuje webhook / API (lub zapis do tabeli w Dataverse),
    - **Power Automate / Azure Function**:
      - aktualizuje Resource Characteristics (dodaje/zmienia skill, poziom).

**Typ:** integracja (flow/integracja z LMS) + OOTB Resource Skills.

---

### 2. Wszystkie ekrany muszą renderować się < 5.25 s

**Rozwiązanie:**
- To jest **NFR (Non-Functional Requirement)**, nie jedna funkcja.
- Działania:
  - ograniczyć ciężkie customizacje (JS, PCF, duże formularze),
  - optymalizować ilość sub-gridów i danych ładowanych na start,
  - monitorować wydajność (Diagnostics, Monitor, Application Insights),
  - testy wydajności.

**Typ:** wymaganie niefunkcjonalne – realizowane przez projektowanie i testy, nie jeden checkbox w UI.

---

### 3. Zasoby muszą mieć certyfikację przed zamknięciem zlecenia bez supervisor’a

**Rozwiązanie:**
- Model:
  - flaga `IsCertified` lub skill „Certified Technician” na Resource.
- Logika:
  - **Business rule / plug-in**:
    - przy zmianie statusu Work Order / Booking na „Completed”:
      - jeśli `IsCertified = false`:
        - wymagany `SupervisorApproval` lub blokuj zmianę (exception, validation).
- Dodatkowo:
  - wykorzystać **Booking Status transitions**, aby wymusić krok supervisor review dla niecertyfikowanych.

**Typ:** OOTB konfiguracja + logika (business rule/plug-in).

---

### 4. Każdy Work Order wymaga Work Order Type

**Rozwiązanie:**
- `Work Order Type` to standardowe pole w Field Service.
- Ustawić:
  - pole jako **Required** na formularzu,
  - albo wymusić przez BPF / business rule (bez typu – brak implementacji/zapisu).

**Typ:** OOTB – konfiguracja wymagalności pola.

---

### 5. Dispatchers robią 70% bookingu, technicy self-booking via mobile

**Rozwiązanie:**
- Dla dispatcherów:
  - używamy **Schedule Board** – centrum zarządzania bookingiem.
- Dla techników:
  - mobilna aplikacja Field Service:
    - włączona funkcja **self-booking** / „Accept job” / „Request next job”.
- Uprawnienia:
  - role Field Service tak, by:
    - dispatcher może bookować dla wszystkich,
    - technik bookuje tylko dla siebie (lub z ograniczeniem).

**Typ:** OOTB Field Service – konfiguracja ról i doświadczenia mobilnego.

---

### 6. Klienci często źle opisują problem, chcemy lepiej wykorzystywać czas techników i wysyłać ich tylko, gdy trzeba

**Rozwiązanie – podejście procesowo-techniczne:**

1. **Remote triage / pre-qualification**
   - Użyć Customer Service / Virtual Agent:
     - seria pytań diagnostycznych,
     - KB articles, checklisty dla pierwszej linii.
   - Tylko gdy problem spełnia określone kryteria → tworzymy Work Order w Field Service.

2. **Connected Field Service / IoT**
   - Jeśli urządzenia (roboty) wysyłają telemetry:
     - integracja z Azure IoT + Connected Field Service,
     - reguły alertów (np. prąd, temperatury, błędy),
     - Work Order tworzony tylko, gdy rzeczywiście jest usterka.

3. **Analiza historycznych danych**
   - Analityka (Power BI, AI) nad danymi:
     - typ zgłoszeń vs faktyczne usterki,
     - budowa modelu scoringowego „prawdopodobieństwo realnej awarii”.

**Typ:** kombinacja:
- OOTB (Connected Field Service, Virtual Agent, KB),
- zmian procesowych,
- ewentualnie AI/analityka.

---

To zestawienie pokazuje, które wymagania załatwiamy:
- czystą konfiguracją (Quick View, BPF, Required fields, Charts),
- automatyzacją (workflows/Power Automate),
- integracjami (LMS, zewnętrzne API),
- a które są stricte NFR i wymagają dobrego projektowania i governance.

