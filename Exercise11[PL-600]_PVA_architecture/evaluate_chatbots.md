# Evaluate Requirements for Chatbots

## Scenario 1 – Product and Pricing

### Wymagania
- Handlowcy muszą łatwo znaleźć informacje o produktach.
- Handlowcy muszą widzieć zasady cenowe dla produktów.
- Zasady produktów i cen różnią się w zależności od kraju.

### Rekomendowane rozwiązanie

### ✔ Copilot Studio (dawniej Power Virtual Agents) + Dataverse + Product Catalog / External Knowledge

Najlepsza architektura:

1. **Copilot Studio**  
   - Tworzymy bota „Product & Pricing Assistant”.
   - Bot udostępniony w Teams (dla handlowców).

2. **Źródła wiedzy bota:**
   - **Dataverse Product Catalog** (Produkty, Product Families, Product Bundles, Price Lists).
   - Dedykowana tabela „PricingRules” z kolumną `CountryCode`.
   - Dokumentacje produktowe w SharePoint → podłączone jako **External Knowledge**.

3. **Funkcjonalności:**
   - Użytkownik pyta: „Co mogę sprzedawać razem z produktem X?” → bot odczytuje Product Relationships.
   - Użytkownik pyta: „Jaka jest cena produktu Y w Niemczech?” → bot pobiera odpowiednią Price List lub Pricing Rules.
   - Bot przestrzega logiki: kraj → filtr produktów i cen.

4. **Dynamic data access:**
   - Można użyć pluginów Copilota lub **Custom Connectors** do pobierania produktów/cen z Dataverse.

### Dlaczego to działa

- Handlowcy nie muszą przeklikiwać formularzy – pytają w naturalnym języku.
- Bot automatycznie wyświetla zasady cenowe zależne od kraju.
- Utrzymanie w Dataverse → scentralizowane zasady, spójność danych.

---

## Scenario 2 – Customer Service

### Obecne systemy
- Dynamics 365 Customer Service (ticketing).
- Stary portal WWW (tylko create ticket + podstawowe dane).
- SharePoint z dokumentacją.

### Wymagania
- Klient musi widzieć status i ostatnią akcję na sprawie.
- Klient chce wyszukiwać informacji (FAQ, dokumenty).
- Proste pytania ma obsługiwać bot.
- Klient może przejść do agenta – agent ma widzieć historię rozmowy bota.

### Rekomendowane rozwiązanie

### ✔ Power Pages + Copilot Studio + Omnichannel for Customer Service

Najbardziej kompletne rozwiązanie:

---

### 1. Power Pages – modernizacja portalu
- Power Pages połączone z Dataverse.
- Klient loguje się i widzi:
  - listę swoich spraw (Cases),
  - status, SLA, ostatnią notę (Case Timeline),
  - możliwość zgłoszenia nowej sprawy,
  - możliwość anulowania lub aktualizacji.

To rozwiązuje pierwsze wymaganie.

---

### 2. Copilot Studio chatbot osadzony w Power Pages
- Bot obsługuje:
  - „Jaki jest status mojej sprawy?” → bot pobiera dane z Dataverse.
  - Pytania „FAQ” → bot używa **External Knowledge** z SharePoint.
  - Najczęstsze problemy → bot może proponować KB articles.

To rozwiązuje drugie i trzecie wymaganie.

---

### 3. Handoff to human agent – Omnichannel
- Copilot Studio z **handoff do Omnichannel**:
  - klient prosi o kontakt z agentem,
  - agent w Omnichannel dostaje:
    - kontekst rozmowy bota,
    - transkrypt,
    - identyfikację klienta.

To rozwiązuje ostatnie wymaganie.

---

### 4. Integracja dokumentacji z SharePoint
- Bot używa External Knowledge:
  - Power Virtual Agents integruje SharePoint jako źródło dokumentów i FAQ.
  - Klient może wyszukiwać dokumenty w naturalnym języku.

---

## Podsumowanie

### Scenario 1 – Product & Pricing
**Copilot Studio + Dataverse (Product Catalog + Pricing Rules) + SharePoint**  
→ Bot w Teams dla handlowców, dynamiczne dane z Dataverse, reguły cenowe per kraj.

### Scenario 2 – Customer Service
**Power Pages + Copilot Studio + SharePoint External Knowledge + Omnichannel**  
→ Modernizacja portalu, bot do FAQ i statusów, płynne przekazanie rozmowy agentowi.

Te dwa scenariusze pokazują dojrzałe zastosowanie Copilota w modelu:
- „internal bot” dla sprzedaży,
- „customer-facing bot” dla portalu.
