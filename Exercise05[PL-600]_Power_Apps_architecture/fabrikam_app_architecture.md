# Fabrikam Robotics – Strategia aplikacji (App Architecture)

## 1. Grupy użytkowników aplikacji

### Sales – small deals
- Zespół pracujący wspólnie.
- Potrzebują dostępu do leadów, opportunities, wizyt i danych z tourów.

### Sales – large deals / field sales
- Często w terenie, słaby zasięg.
- Potrzebują trybu offline i podglądu danych tourów.

### Sales Managers
- Zatwierdzają rabaty.
- Nie mogą zatwierdzić rabatu sami dla siebie.

### Reception Staff
- Check-in odwiedzających.
- Robienie zdjęć, podpisywanie waiverów.
- Brak dostępu do danych sprzedażowych.

### Customers
- Składają request o wizytę na stronie WWW.
- Otrzymują SMS reminder 24h przed wizytą.

---

## 2. Proponowane Power Apps

### Sales (small & large deals) – Model-driven App
- Oparta na Dataverse, wspiera proces sprzedażowy.
- Łatwe sub-gridy, reguły biznesowe, PCF.
- Obsługa offline dla pracy terenowej.
- Umożliwia podpięcie danych tour poprzez custom connector.

### Reception – Canvas App (tablet/desktop)
- Duże przyciski, UI „kioskowe”.
- Lista wizyt i rezerwacji.
- Check-in, zdjęcia, podpisy.
- Brak danych sprzedażowych dzięki rolom bezpieczeństwa.

### Customers – Power Pages
- Formularz requestu wizyty.
- Zapis do Dataverse.
- Integracja z Flow do wysyłki SMS 24h przed wizytą.

### Opcjonalnie – custom WWW
- Jeśli firma ma już stronę, możliwe użycie Power Automate + API + Dataverse.

---

## 3. Obsługa pracy offline dla Sales

- Włączone **offline profiles** w model-driven app.
- Synchronizowane:
  - Opportunities,
  - Accounts,
  - Visits,
  - Visit summaries,
  - podstawowe dane z tourów.
- UI nie polega na ciągłym wywołaniu API.
- Tour data agregowane i zapisane do Dataverse → dostępne offline.

---

## 4. Custom Connector – użycie

### Zakres
API urządzeń trackingowych → dostęp do:
- `GetVisitTrackingSummary`
- `GetVisitTrackingTimeline`

### Integracja
- W model-driven app:
  - **Virtual table** lub
  - **PCF component** renderujący mapę/tracking.
- W Power Automate:
  - Sync wsadowy danych trackingowych co określony czas.

### W UI sprzedażowym
- Na formularzu Opportunity → sub-grid Visits.
- Na formularzu Visit → PCF / mapa / timeline z tour data.

---

## 5. Power Apps Component Framework – gdzie użyć?

- **Wizualizacja tour data** (mapa, timeline).
- **Kontrolka rabatu** (discount + approver, walidacja).
- **Zaawansowany viewer zdjęć** (opcjonalnie).
- **Reusable header** na wielu formularzach.

---

## 6. Canvas App – jak wspierać wielu twórców?

- **Component Libraries** – wspólne nagłówki, layouty, walidacje.
- **Podział pracy per ekran** – check-in, waiver, visitor history.
- **Solution management** – trzymanie w solution + repo.
- Naming conventions (`txtName`, `btnSave`, `galVisits`).

---

## 7. Co delegujemy do Power Automate?

### SMS reminder 24h przed wizytą
- Flow cykliczny:
  - znajduje wizyty za 24h,
  - wysyła SMS (Twilio / Azure Communication Services).

### Discount approval
- Flow wysyła notyfikację do managera.
- Po akceptacji aktualizuje status rabatu.

### Visitor request z WWW / Power Pages
- Flow tworzy rekord wizyty / requestu.
- Potwierdzenie mailowe/SMS.

### Integracje trackingowe
- Cykl flow:
  - pobiera dane z API (custom connector),
  - zapisuje do Dataverse jako agregaty lub zdarzenia,
  - zapewnia dostęp offline.

---

# Podsumowanie

Proponowane rozwiązanie obejmuje:
- model-driven app dla Sales (z offline i custom connector),
- canvas app dla recepcji (zdjęcia, check-in, podpisy),
- Power Pages dla klientów (request wizyt + SMS reminder),
- PCF dla wizualizacji danych trackingowych,
- Power Automate do procesów automatyzacji.

Wszystkie wymagania są zaadresowane zgodnie z najlepszymi praktykami architektury Power Platform.
