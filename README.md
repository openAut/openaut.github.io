# openAut

> Öppet kunskaps- och konceptprojekt för att utforska hur AI med lokala modeller kan användas inom fastighetsautomation.

> [!WARNING]
> **openAut är ett konceptprojekt under utveckling — inte en färdig produkt.**
> Det är byggt för att lära, experimentera och visa på möjligheter med lokala AI-modeller inom fastighetsautomation. Det ska **inte** användas i skarp drift eller live-miljöer.

openAut utforskar hur ett **öppet tillägg till ett befintligt BMS** skulle kunna se ut — ett som samlar in driftsdata via MQTT over TLS, analyserar den lokalt med **lokala AI-modeller**, och i förlängningen skulle kunna driftsätta Python-baserad reglering på edge-noder. Konceptet utforskas med offentlig upphandling, regelefterlevnad (ISO 27001, NIS2, IEC 62443, CRA, AI Act) och långsiktig förvaltning i åtanke.

**Hemsida:** https://openaut.io · **Licens:** MIT · **Status:** v0.1 — konceptstadie, aktiv utveckling

---

## Vad openAut gör

- **Feldetektering (FDD):** Korrelerar signaler över tid och identifierar rotorsak med konfidensgrad — levererat i Teams, inte i BMS-klienten.
- **Energioptimering:** Prediktion, lastprognos och avvikelseanalys mot historisk trenddata.
- **Guidad integration:** Läs in en fabrikants manual, ange edge-nod — openAut guidar teknikern steg för steg och dokumenterar automatiskt.
- **Python edge-reglering:** Driftsätt Python-baserade reglerloopar direkt på edge-noden mot lokal I/O. Utan rundtur till AI-servern. Versionshanterat, loggat och återkallelsebart centralt via NemoClaw.
- **Automatisk dokumentation:** I/O-listor, MQTT topic-schema, driftsättningsprotokoll och FAT/SAT genereras som sidoeffekt av normal drift.

---

## Arkitektur — fyra lager

```
LAGER 04 — GRÄNSSNITT   Teams · Webb-HMI · REST API
LAGER 03 — AI           OpenClaw · NemoClaw · lokal AI-hårdvara · öppna modeller (lokal LLM) · FDD · Energioptimering
                         EMQX-broker · Telegraf (ingest) · TimescaleDB/PostgreSQL · lokal Forge (Forgejo)
LAGER 02 — EDGE         Linux-noder · SSH · Protokolldrivrutiner · Python edge-reglering
                         MQTT over TLS (stream + on-demand request/response via topics)
LAGER 01 — FÄLT         Modbus RTU/TCP · BACnet · M-Bus · LoRaWAN · KNX · DALI
```

**LAGER 01 — FÄLT:** Befintlig fältutrustning ansluts utan modifiering. PLC:er, DDC-regulatorer och mätare kommunicerar via sina befintliga fältprotokoll. openAut läser — fältets reglering behåller prioritet.

**LAGER 02 — EDGE:** Siemens SIMATIC IOT2050-noder kör standard Linux och nås av NemoClaw via krypterad SSH. Protokolldrivrutiner körs direkt på noden. NemoClaw kan via SSH även driftsätta Python-regleringsskript mot lokal I/O — slutna reglerloopar utan molnrundtur. Data transporteras krypterat till EMQX-brokern via MQTT over TLS — både som kontinuerlig mätström och som on-demand request/response via topics.

**LAGER 03 — AI:** En lokal AI-server — valfri LLM- och ML-hårdvara, dimensionerad efter de modeller som ska köras — kör EMQX-brokern, Telegraf (ingest till TimescaleDB/PostgreSQL) och agentstacken lokalt. Agentstacken är **NemoClaw** — NVIDIAs härdade referensimplementation ovanpå **OpenClaw** — som kör lokal inferens med **öppna modeller** (open weights) i en OpenShell-sandbox. NemoClaw läser historik och skriver analyser och larm till masterdatabasen. Kod, manualer, genererad dokumentation och migrationer versionshanteras i en **lokal Forge (Forgejo)** — åtkomlig av både människor och AI, med CI- och granskningsgrindar innan något blir betrott eller driftsatt. Drift- och analysdata stannar i fastigheten; endast beslut och notifieringar når Teams (moln).

**LAGER 04 — GRÄNSSNITT:** Insikter når de som behöver dem i de verktyg de redan använder. Drifttekniker i Teams. Energisamordnare i dashboard. Integrationsteam via REST API.

---

## Teknisk stack

| Lager | Komponenter |
|---|---|
| **openAut** (domänramverk) | BACnet-skill · Modbus-skill · M-Bus-skill · LoRa-skill · FDD-skill · Energianalys-skill · SSH edge-access · Python I/O-skill · Edge-reglerings-skill · MIT |
| **NemoClaw** (agentstack) | NVIDIAs referensimplementation ovanpå OpenClaw · OpenShell-sandbox (Landlock + seccomp + netns) · policy-baserade guardrails · lokal inferens med öppna modeller · livscykelhantering · Apache 2.0 · alpha/tidig fas (mars 2026) |
| **OpenClaw** (agent-gateway) | Självhostad multi-channel-gateway för AI-agenter · Teams-kanal · skills & verktygsstöd · MIT · 250 000+ GitHub-stjärnor (mars 2026) |
| **Modell** (LLM) | Öppna modeller (open weights) · valfri modellfamilj · lokal inferens · modellstorlek dimensioneras efter tillgänglig hårdvara |
| **AI-hårdvara** | Valfri lokal LLM- och ML-hårdvara · GPU eller annan AI-accelerator · dimensioneras efter modellval · ingen hårdvarulåsning |
| **Edge-hårdvara** | Siemens SIMATIC IOT2050 |
| **I/O-modul** | Siemens EM1.8U (8× universell I/O · Modbus RTU · RS485) |
| **Forge** (System of Record) | Forgejo · självhostad · versionerat arkiv för kod/manualer/dok/migrationer · CI + PR-granskning · scope:ade agent-tokens · GPLv3 |
| **MQTT-broker** | EMQX · TLS · klientcertifikat · request/response-topics |
| **Ingest** | Telegraf (EMQX → TimescaleDB) |
| **Transport** | MQTT over TLS · WireGuard VPN |
| **Databas** | TimescaleDB · PostgreSQL · Haystack · Brick Schema |

> **Licenser i stacken:** openAut och OpenClaw är MIT-licensierade. NemoClaw är Apache 2.0 och i tidig förhandsfas (alpha, lanserad mars 2026) — openAut pinnar verifierade versioner och kapslar in stacken bakom stabila gränssnitt så att grunden kan utvecklas utan att driftmiljön påverkas.

---

## Om projektet

openAut är i grunden ett **kunskaps- och konceptprojekt**: ett sätt att utforska hur AI med lokala modeller faktiskt kan användas inom fastighetsautomation — inte bara i teorin. Projektet befinner sig i **konceptstadiet** och är under aktiv utveckling.

Det här är ingen färdig produkt. openAut **ska inte användas i skarp drift eller live-miljöer** — det är byggt för att lära, experimentera och visa på möjligheter. Målet är att över tid förstå vad som krävs för att något liknande ska kunna mogna till en driftsäker lösning.

Arkitekturen hålls medvetet öppen, modulär och fri från inlåsning, så att den kan växa i den riktning som visar sig vara rätt — utan att något utesluts i förväg. Allt utvecklas öppet under MIT-licens, fritt att granska, ifrågasätta och bidra till.

---

## Referenshårdvara

### AI-lager

openAut är hårdvaruagnostiskt i AI-lagret — plattformen ställer kapacitetskrav, inte produktkrav:

| Krav | Beskrivning |
|---|---|
| Hårdvara | Valfri lokal LLM- och ML-hårdvara — GPU-server, AI-workstation eller kompakt system med unified memory |
| Acceleratorminne | Dimensioneras efter vald modell — lättviktig LLM kräver lite, stora resonerande modeller mer |
| OS | Linux (Ubuntu 24.04 LTS rekommenderat) · x86_64 eller ARM64 |
| Lagring | ≥2 TB NVMe rekommenderat |
| Nät | Lokal inferens · air-gap möjligt — inga molnberoenden |

### Edge-lager
| Enhet | CPU | Gränssnitt | Roll |
|---|---|---|---|
| Siemens SIMATIC IOT2050 | TI AM6548 · 4× A53 · 1 GHz | RS232/422/485 · 2× GbE · Arduino Shield | Python-regulator och datainsamlare via SSH |
| Siemens EM1.8U | — | 8× universell I/O · Modbus RTU · RS485 | Industriell I/O, upp till 31 moduler per buss |

Hela stacken körs på både x86_64 och ARM64. `pymodbus`, `paho-mqtt` och `BAC0` är verifierade utan proprietära beroenden — oavsett hårdvaruleverantör.

---

## Säkerhet

Säkerhet är inbyggt från dag ett, inte tillagt i efterhand:

- VLAN-segmentering per lager
- MQTT over TLS med klientcertifikat för all datatransport från edge-noder
- WireGuard VPN för fjärråtkomst
- RBAC och auditloggning
- Regleringskod versionshanteras och granskas som konfiguration
- Separat säkerhetsagent på egen fysisk hårdvara med read-only SSH och listen-only Teams-bot — den observerar men kan inte agera, så en prompt injection saknar verkningsradie

### Regelverk — fem ramverk, tydlig ansvarsfördelning

openAut håller sig inom fem ramverk där vart och ett styr sin domän:

| Ramverk | Domän |
|---|---|
| **ISO 27001** | Ledningssystem och governance — grunden de övriga hängs upp i |
| **NIS2** | Verksamhetskrav och incidentstyrning (24h/72h-rapportering) |
| **IEC 62443** | OT/industriell cybersäkerhet — zonindelning, security levels, härdning |
| **CRA** | Cybersäker produkt med digitala komponenter — SBOM, sårbarhetshantering, uppdateringar |
| **AI Act** | AI-funktionens krav och styrning — mänsklig kontroll, transparens, loggning, dokumentation |

Se fullständig hotmodell, säkerhetsarkitektur och regelverksgenomgång: https://openaut.io/security.html

---

## Offentlig sektor och LOU

openAut är MIT-licensierat — det finns ingen enskild leverantör att upphandla. Plattformen är gratis att använda. Vad din organisation upphandlar är **kompetensen** att implementera och förvalta den — tjänster som kan tilldelas valfri regional konsult eller befintlig ramavtalspartner.

En lösning som en kommun bygger kan återanvändas av alla andra. Bidrag tillbaka till projektet stärker plattformen för hela ekosystemet.

---

## Fältprotokollstöd (edge → fält)

`Modbus RTU` · `Modbus TCP` · `BACnet/IP` · `BACnet MS/TP` · `M-Bus` · `LoRaWAN (EU868)` · `KNX` · `DALI`

**Transport edge → AI-lager:** `MQTT over TLS` via EMQX-broker (kontinuerlig ström och on-demand request/response)

---

## Kom igång

```bash
# Hemsida och dokumentation
https://openaut.io

# Systemtopologi — lager, protokoll, dataflöden och referenshårdvara
https://openaut.io/topologi.html

# IT/OT-säkerhet — hotmodell, säkerhetsarkitektur och compliance
https://openaut.io/security.html

# Källkod
https://github.com/openaut
```

Projektet är ett konceptprojekt under aktiv utveckling — för att lära, inte för skarp drift. Bidra med en protokolldrivrutin, anpassa en applikationsprofil, eller experimentera med edge-reglering i en testmiljö.

---

MIT License · openAut · https://openaut.io
