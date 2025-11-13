# High-Level Architecture (Opis)

## Kanały / Frontend

### Strona WWW + portal klienta – Power Pages
Klient może:
- zgłaszać sprawy (Case),
- sprawdzać status zgłoszeń (Req 3, 12),
- przejąć własność łóżka lub umowy (Req 16).

### Chat na stronie
- Power Virtual Agents / Copilot Studio – bot do zgłoszeń i FAQ
- Omnichannel for Customer Service – live chat, social, voice (Req 3, 13)

### Email / telefon / social media
Wpadają do Omnichannel / Customer Service i zamieniają się w Cases (Req 1–3, 10).

## Warstwa aplikacyjna

### Dataverse – główna baza
Tabele: Customers, Beds, Support Agreements, Cases, Knowledge Articles, Activities, SLAs, Technicians, Work Orders, TelemetrySummary.

### Dynamics 365 Customer Service
- Cases + Queues + Routing + SLA (Req 1, 2, 11, 14, 15, 17)
- Knowledge Base (Req 4)
- Email templates + notifications (Req 10, 13)
- Entitlements dla poziomów supportu

### Dynamics 365 Field Service
- Work Orders
- Bookable Resources
- Schedule Board + RSO (Req 6, 8)
- Mobile app dla techników

### Power Automate
- Routing spraw (Req 1)
- Email po zamknięciu (Req 10)
- SLA monitoring (Req 11, 14)
- Negative sentiment alerts (Req 13)

## Integracje – producenci
- Dataflows / PA / Logic Apps – nightly feed (Req 5)
- Mapowanie Manufacturer → Customer → Bed
- Agregacja telemetry → Dataverse / Power BI (Req 7, 18)

## Warstwa raportowa – Power BI
- SLA dashboards
- Backlog / wydajność agentów
- Proaktywne maintenance (Req 7)
- KPI C-level (Req 18)

## Niefunkcjonalne
- Regiony US/UK/India
- Optymalizacja UI 1024x768
- Minimalizacja pluginów / JS (Req 17)
