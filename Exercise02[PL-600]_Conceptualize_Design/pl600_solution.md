# High-Level Architecture (Opis)

## Kanały / Frontend

### Strona WWW + portal klienta – Power Pages
Klient może:
- zgłaszać sprawy (Case),
- sprawdzać status zgłoszeń (Req 3, 12),
- przejąć własność łóżka lub umowy (Req 16).

### Chat na stronie
- Power Virtual Agents / Copilot Studio
- Omnichannel for Customer Service

### Email / telefon / social media
Wpadają do Omnichannel i stają się Cases.

## Warstwa aplikacyjna

### Dataverse
Tabele: Customers, Beds, Support Agreements, Cases, KB, Activities, SLAs, Technicians, Work Orders, TelemetrySummary.

### Dynamics 365 Customer Service
Cases, Queues, Routing, SLA, KB, Email templates, Entitlements.

### Dynamics 365 Field Service
Work Orders, Bookable Resources, Schedule Board, RSO, Mobile app.

### Power Automate
Routing, powiadomienia, SLA monitoring, sentiment alert.

## Integracje producentów
Dataflows / PA / Logic Apps. Nightly feed, mapping, telemetry → Dataverse.

## Power BI
Dashboardy SLA, backlog, wydajność, proactive maintenance, KPI C-level.

## NFR
Regiony, optymalizacja UI, performance profiles.

# Dynamics 365 Apps Used

- D365 Customer Service
- D365 Field Service
- Power Pages
- Omnichannel
- Power Virtual Agents
- Power Automate
- Power BI

# Fit-Gap (skrót)

1. Routing – H/M – Config – Unified Routing.
2. Lista spraw – H/S – OOTB.
3. Chat → Case – H/M – Integration.
4. KB search – H/S – OOTB.
5. Nightly feed – H/L – Integration.
6. Onsite – H/M – Field Service.
7. Proactive – H/ML – Integration/Custom.
8. Technician schedule – H/M – OOTB.
9. Support agreement sale – M/M – Custom/Config.
10. Email after close – M/S – OOTB.
11. SLA violations view – H/S – OOTB.
12. Portal status – H/M – Config.
13. Sentiment alert – M/M – Custom.
14. SLA timer – H/S – OOTB.
15. Days open – H/S – OOTB.
16. Ownership transfer – M/M – Custom/Config.
17. Performance – H/M – NFR.
18. Real-time metrics – H/M – Reporting.
