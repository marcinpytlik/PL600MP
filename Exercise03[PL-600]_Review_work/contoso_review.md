# Przegląd Workbooka Projektowego – Contoso (Review)

## 1. Diagram architektury wysokiego poziomu

### Trafność względem wymagań
Diagram ogólnie oddaje główne elementy rozwiązania:
- kanały komunikacji,
- rdzeń oparty na Dynamics 365,
- podstawową warstwę integracji,
- raportowanie Power BI.

Pasuje do wymagań dotyczących obsługi zgłoszeń, SLA i pracy techników terenowych.

### Braki / możliwości poprawy
- Brakuje wyraźnego pokazania:
  - proaktywnego utrzymania opartego na telemetrii (Req 7),
  - procesu nocnego feedu z producentów (Req 5),
  - portalu samoobsługowego i chatu (Req 3, 12, 16),
  - rozróżnienia Customer Service vs Field Service.
- Integracja z producentami jest pokazana zbyt ogólnie.
- Power BI jest przedstawione zbyt skrótowo względem wymagań menedżerskich (Req 18).

### Jak poprawić diagram
- Po lewej dodać „Kanały” → Omnichannel / Portal / Bot.
- W centrum rozdzielić moduły:
  - Dynamics 365 Customer Service,
  - Dynamics 365 Field Service,
  - Dataverse.
- Na dole: systemy producentów → feed nocny → integracja → Dataverse.
- Po prawej: dashboardy Power BI.

---

## 2. Dynamics 365 – ocena poprawności użytych aplikacji

### Poprawne wybory
- Dynamics 365 Customer Service – właściwy wybór dla obsługi spraw, SLA i KB.
- Dynamics 365 Field Service – właściwy wybór dla wizyt techników i harmonogramowania.

### Braki / niepełne decyzje
- Brakuje Omnichannel for Customer Service (chat, social, sentyment).
- Brakuje Power Pages dla portalu klienta (Req 3, 12, 16).
- Proaktywne maintenance nie jest oparte na Power BI/telemetrii w sposób wystarczająco szczegółowy.

### Podsumowanie
Większość wyborów poprawna, ale brakuje komponentów kanałowych i samoobsługowych.

---

## 3. Fit–Gap – ocena

### Potencjalne pomyłki / zaniżone szacunki

#### Req 3 – chat na stronie
- Ocenione jako OOTB/S, ale wymaga konfiguracji Omnichannel → powinno być M + Integration.

#### Req 7 – proaktywne utrzymanie
- Ocenione jako OOTB, a wymaga telemetrii, agregacji i reguł → powinno być Integration/Custom + M–L.

#### Req 16 – przejęcie własności
- Ocenione jako prosta konfiguracja, a wymaga walidacji, aktualizacji wielu relacji i audytu → Custom/Config + M.

### Podsumowanie
- Świetnie, że zespół korzysta głównie z OOTB,
- Ale część integracji i procesów została zbyt uproszczona.

---

## 4. Ocena ogólna

### Mocne strony zespołu
- Poprawne dopasowanie Customer Service / Field Service do wymagań.
- Spójna logika obsługi zgłoszeń, SLA i workflowów.
- Dobrze zorganizowany Fit–Gap.

### Co można poprawić
- Zbyt ogólne potraktowanie integracji z producentami.
- Brak wyraźnej warstwy kanałów (chat, portal, sentyment).
- Podszacowanie nakładu pracy przy telemetry, ownership transfer i chat.

---

## 5. Sposób przekazywania feedbacku (warsztat)

### Jako zespół udzielający feedbacku
- Merytorycznie, konstruktywnie, z przykładami.
- Skupienie na poprawie projektu, nie na krytyce.

**Przykład:**
> „Wasze podejście do Customer Service i Field Service jest bardzo dobre. Mamy kilka propozycji usprawniających kanały komunikacji i obszary telemetry — to pomoże dokładniej odzwierciedlić wymagania.”

### Jako zespół przyjmujący feedback
- Słuchać uważnie, bez przerywania,
- Nie być defensywnym,
- Zadawać pytania doprecyzowujące.

---

## 6. Źle napisane wymaganie – przykład (opcjonalnie)
Jeśli starczy czasu, wskazać jedno wymaganie, które jest nieprecyzyjne.

**Np.: Req 7 – Proaktywne maintenance.**  
Wymaganie nie określa:
- jakie dokładnie dane telemetryczne są potrzebne,
- jak często powinny być analizowane,
- jaki próg powoduje „proaktywne działanie”.

Wymaganie można doprecyzować, aby uniknąć rozbieżnych interpretacji.

