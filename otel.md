
# Observability & OpenTelemetry – Študijné poznámky

## Observability – Čo to je?
**Observability** je schopnosť **sledovať a porozumieť vnútornej činnosti systému** na základe jeho **výstupných dát**.

- Umožňuje efektívne **monitorovanie, troubleshooting a optimalizáciu**.
- Postavená na **troch základných pilieroch**:
  1. **Traces** – stopy
  2. **Metrics** – metriky
  3. **Logs** – logy

---

## 1. Traces – Stopy

- **Traces** sú dáta, ktoré zaznamenávajú **tok jednotlivých požiadaviek** naprieč systémami.
- Skladajú sa z viacerých častí nazývaných **spany**:
  - Každý **span** predstavuje jeden krok (napr. API volanie, DB dotaz, interná logika).

**Príklad**:
Používateľ zadá objednávku → front-end → objednávkový mikroservis → platobná brána → databáza → e-mail.

- Každý krok = jeden **span**
- Celý tok = jeden **trace**

### Distribuované Traces
- Umožňujú sledovať požiadavku, ktorá prechádza **cez viacero služieb** v rámci distribuovaného systému.

---

## 2. Metrics – Metriky

- **Metriky** sú **kvantitatívne merania** o správaní systému počas jeho behu.
- Príklady:
  - Čas odozvy
  - Počet požiadaviek za sekundu
  - Využitie CPU, pamäte

### Metric Event
- **Udalostná situácia**, ktorá spôsobí **zmenu alebo aktualizáciu metrík**.

### Metric Instruments
- Nástroje SDK na meranie:
  - `Counter`, `UpDownCounter`, `Histogram`, `ObservableGauge`

### Metric Exporter
- Komponent, ktorý **exportuje metrické údaje** do nástroja na analýzu (napr. Prometheus, Datadog).

---

## 3. Logs – Logy

- **Textové záznamy udalostí**, ktoré nastali počas behu aplikácie.
- Môžu byť štruktúrované alebo neštruktúrované.
- Obsahujú čas, úroveň (napr. INFO, ERROR), správu a často aj kontext (napr. trace ID).

---

## Metrics & Reliability

- Metriky poskytujú **kvantitatívne podklady** na hodnotenie **spoľahlivosti systému** (dostupnosť, odozva, chybovosť).

---

# Signals v OpenTelemetry

## Čo sú „signals“?
**Signals** sú typy **telemetrických dát**, ktoré OpenTelemetry zbiera z aplikácií alebo systémov.

## Typy signalov:
- **Traces** – cesta požiadavky, prehľad výkonnosti, latencie, chýb
- **Metrics** – merania o správaní systému v čase
- **Logs** – záznamy o konkrétnych udalostiach počas behu

---

# Instrumentation (Inštrumentácia)

**Inštrumentácia** je proces, ktorým do aplikácie vkladáme **kód alebo nástroje** na zber telemetrických dát (traces, metrics, logs).

## Typy inštrumentácie:

### Zero-code / Auto-instrumentation
- Automatická inštrumentácia bez úprav zdrojového kódu
- Aktivuje sa cez agenta (napr. `opentelemetry-instrument`)

### Code-based / Manual instrumentation
- Ručné vkladanie kódu do aplikácie
- Používateľ má plnú kontrolu nad tým, čo a ako sa zaznamenáva

### Libraries / Instrumented Libraries
- Predpripravené knižnice s OpenTelemetry hookmi
- Príklad: `opentelemetry-instrumentation-flask`, `requests`, `express`

---

## Summary

| Pilar      | Popis                                                  |
|------------|---------------------------------------------------------|
| Traces     | Sledujú požiadavky naprieč službami (spany)             |
| Metrics    | Kvantitatívne merania (čas, počet, využitie)            |
| Logs       | Textové záznamy udalostí počas behu                     |
| Signals    | Typy dát: traces, metrics, logs                         |
| Instrumentation | Proces vloženia merania do aplikácie               |

---

_Vypracoval ako učebný materiál: OpenTelemetry študijné poznámky_
