# Power Automate Architecture – Contoso Smart Beds

## Scenariusz 1 – Autoryzacja wymiany łóżka

### Opis
Support zgłasza potrzebę wymiany łóżka producentowi. Producent zatwierdza lub odrzuca. Po zatwierdzeniu support informuje klienta.

### Architektura Power Automate
**Flow 1 – Start replacement approval**
- Trigger: Dataverse on create `Replacement Request` (Status = Requested)
- Utworzenie Approval → ManufacturerContact
- Wynik:
  - Approved → update Replacement Request + Help Request, notyfikacja do support
  - Rejected → update + powiadomienie

**Flow 2 – Notify customer**
- Trigger: Replacement Request status = Approved
- Mail/SMS do klienta

**Dlaczego Flow?**
Naturalny proces akceptacji, udział osób zewnętrznych, brak potrzeby transakcji synchronicznej.

---

## Scenariusz 2 – Sprawdzenie planu wsparcia i odjęcie allowance

### Wymóg
System ma:
- sprawdzić plan wsparcia,
- odjąć koszt naprawy od allowance,
- ustawić priorytet Help Request,
- działać natychmiast przy tworzeniu rekordu.

### Ocena plug-in vs Power Automate
- Logika finansowa musi być **synchroniczna**, transakcyjna.
- Flow jest asynchroniczny → opóźnienie, race conditions.

### Rekomendacja
- **Synchronous plug-in**:
  - wylicza allowance,
  - aktualizuje plan, ustawia priorytet,
  - natychmiast dostępne w UI.
- Power Automate:
  - powiadomienia,
  - zadania follow-up.

---

## Scenariusz 3 – Recall dla 200k+ łóżek

### Wymóg
Plik z serialami → oznaczyć aktywne łóżka.  
Musi być restartowalne, bez duplikacji, skalowalne.

### Projekt

#### Tabele
- **Bed** – serial, recall tag
- **BedRecall** (opcjonalnie) – wiele recalli
- **RecallImport** – SerialNumber, RecallName, Status (Pending/Processed/Error)

#### Flow A – Import pliku
- Trigger: File created (SharePoint/OneDrive)
- Parsowanie CSV
- Tworzenie rekordów `RecallImport (Pending)`

#### Flow B – Batch processor
- Trigger: Recurrence co X minut
- Pobierz np. 1000 Pending
- Dla każdego:
  - Znajdź Bed
  - Jeśli istnieje → update BedRecall → Status=Processed
  - Jeśli brak → Status=NotFound
  - Błąd → Status=Error, Attempts++

**Idempotencja:**
- Przetwarzamy tylko Pending
- Unikatowe `(RecallName + SerialNumber)`
- Można dodać unikalny key w BedRecall

**Skalowalność:**
Batch 500–2000 rekordów/run.  
Opcjonalnie: Dataflows / ADF dla masowej integracji.

---

## Scenariusz 4 – Deductible przy tworzeniu Help Request

### Wymóg
Deductible ma być widoczny **natychmiast** przy tworzeniu rekordu.  
Zależny od wieku łóżka i planu wsparcia.

### Ocena Flow
- Cloud flow działa po zapisie → opóźnienie.
- Nie jest częścią transakcji → nie gwarantuje spójności.

### Rekomendacja
**Nie Power Automate** do obliczenia.  
Lepiej:
1. **Calculated field** – jeśli prosta formuła.
2. **Business rule** – logika UI.
3. **Synchronous plug-in** – pełna logika, natychmiastowy wynik.

Power Automate może służyć do:
- notyfikacji,
- audytu,
- działań follow-up.

---

# Podsumowanie

| Scenariusz | Najlepsze rozwiązanie | Rola Power Automate |
|------------|-----------------------|----------------------|
| 1. Replacement approval | Power Automate approvals | Główna logika |
| 2. Warranty allowance | Plug-in | Powiadomienia |
| 3. Recall (200k+) | Batch Power Automate / Dataflows | Orkiestracja, integracja |
| 4. Deductible | Plug-in / Calculated / Business rule | Powiadomienia |

To podejście łączy asynchroniczne procesy Power Automate z tymi, które wymagają pełnej transakcyjności w plug-inach.
