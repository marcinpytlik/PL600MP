# Contoso Robotics – Security System (Visitors, Access, Parking, Incidents)

## 1. Wymagane aplikacje i typy aplikacji

### Grupy użytkowników
- **Security Officers** – obsługa wejść, check-in/out, inspekcje.
- **Security Managers** – planowanie, raporty, kontrola dostępów, incydenty.
- **Employees** – rezerwacje wizyt, incydenty.
- **Visitors** – dostęp do informacji o wizycie i parkingu.
- **Security Contractors** – inspekcje, zgłoszenia.

---

## 2. Proponowane aplikacje

### Security Entrance App (Canvas, tablet/kiosk)
- Lista wizyt dzisiejszych.
- Skan QR → check-in/out.
- Dane gościa, hosta, plan czasowy.
- Widok parkingu.

### Employee Visit Booking App (Model-driven)
- Tworzenie Visit.
- Wybór budynku wg uprawnień.
- Rezerwacja parkingu.
- Generacja QR.
- Podgląd historii wizyt.

### Visitor Portal (Power Pages)
- Podgląd szczegółów wizyty.
- Rejestracja numeru auta.
- Informacja o parkingu.

### Security Manager App (Model-driven)
- Zarządzanie: Buildings, Access Keys, EmployeeAccess, Shifts, Parking.
- Dashboardy: obciążenie budynków, statystyki wizyt.
- Inspekcje, incydenty.
- Kontrola i synchronizacja swipe card.

### Inspection App (Canvas, mobile)
- Lista inspekcji.
- Checklista, zdjęcia, notatki.
- Aktualizacja statusu.

### Incident Reporting App (Canvas, Teams)
- Formularz incydentu.
- Zdjęcia, lokalizacja.
- Lista moich incydentów.
- Security Manager widzi i rozdziela zgłoszenia.

### Access Control Admin App (Model-driven)
- Zarządzanie dostępami pracowników i kontraktorów.
- Integracja z systemem kart (flow/connector/RPA).

---

## 3. Pokrycie wymagań

- Tracking visitors → Visit Management.
- Check-in/out, QR → Security Entrance App.
- Parking → Parking Pass + logic.
- Visitors booking parking → Portal.
- Auto-delete 90 dni → Power Automate.
- Incidents → Incident Reporting + Manager.
- Inspections → Scheduled flows + Inspection App.
- Security shifts → tabela + Manager App.
- Swipe card sync → Power Automate/RPA.

---

## 4. Segmentacja Solutions

### Warstwa Core – **Security Core Solution**
Zawiera:
- Tabele bazowe: Building, Room, Country, Visit, Parking Pass, Inspection, Access Key.
- Global Choices.
- Security roles.
- Connection roles.
- Minimalne formularze.

### Warstwy funkcjonalne

#### Visitor Management Solution
- BPF Visit.
- Model-driven dla pracowników.
- Canvas Security Entrance.
- QR flow.
- Auto-delete flow.

#### Parking Management Solution
- Logika parkingu (rules/workflows).
- Formularze i widoki parkingowe.

#### Inspection Management Solution
- Inspection App (canvas).
- Flow tworzący inspekcje.

#### Access Control Solution
- EmployeeBuildingAccess.
- SecurityShift.
- Sync z systemem kart (flow/RPA).
- Model-driven admin.

#### Incident Management Solution
- Incident tables.
- Canvas Teams app.
- Routing flows.

#### Portal Solution
- Power Pages.
- Formularze wizyt i parkingu dla gości.

---

## 5. Podsumowanie
Pełna architektura obejmuje zestaw aplikacji: canvas, model-driven, Power Pages oraz integracje i automatyzacje w Power Automate. Segmentacja solutions w warstwy core + feature pozwala zwiększyć skalowalność, bezpieczeństwo i łatwość wdrażania kolejnych etapów systemu.
