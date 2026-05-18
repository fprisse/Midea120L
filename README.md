# AMH6DC Warmtepomp Project — Bevindingen & Documentatie

**Datum:** 18 mei 2026  
**Systeem:** GreenPump AMH6DC · BT-500 Buffertank · Elektrische DHW-boiler  
**Doel:** Zonne-energieopslag via thermische buffer, weekendverwarming, DHW-voorbereiding

---

## 1. Warmtepomp — GreenPump AMH6DC

### 1.1 Identificatie

| Parameter | Waarde |
|---|---|
| Handelsmerk | GreenPump (Renova Heating B.V.) |
| OEM merk | **Midea** (platform: Midea 120L / KJR-120X) |
| Typenummer | AMH6DC |
| RVO meldcode | KA26525 |
| Compressormerk | GMCC (Midea dochteronderneming) |
| Koudemiddel | R290 (Propaan) |
| Type | Lucht-water monoblock |
| ISDE subsidiebedrag | €2.125 (subsidiabel vermogen 4 kW) |

> **Let op:** Renova Heating B.V. is failliet. Geen fabrikantondersteuning beschikbaar.  
> Documentatie moet worden verkregen via Midea OEM-kanalen of het elektriciteitskastdeksel.

### 1.2 Technische specificaties (uit fabrieksblad)

**Ruimteverwarming @ 7°C/6°C buitenlucht (DB/WB), watertemperatuur 30°C/35°C**

| Parameter | Waarde |
|---|---|
| Verwarmingsvermogen | 6,40 kW |
| Elektrisch vermogen | 1,33 kW |
| COP | 4,81 |

**Ruimteverwarming @ 7°C/6°C buitenlucht, watertemperatuur 47°C/55°C**

| Parameter | Waarde |
|---|---|
| Verwarmingsvermogen | 5,50 kW |
| Elektrisch vermogen | 1,70 kW |
| COP | 3,23 |

**Algemeen**

| Parameter | Waarde |
|---|---|
| Min./max. buitentemperatuur | -25°C tot +45°C |
| Max. aanvoertemperatuur | 75°C |
| Instelwater verwarmingsmodus | 25–75°C |
| Voeding | 220–240V~ / 50Hz (eenfasig) |
| Ontdooien | Automatisch, 4-wegklep |
| Nom. waterstroom | 1,10 m³/h |
| Waterzijdige aansluitingen | DN 25 (1") |
| IP-klasse | IPX4 |
| Geluidsniveau | 51 dB(A) op 1m |
| Netto gewicht | 90 kg |
| Afmetingen (L×B×H) | 1115×415×900 mm |
| SG Ready | ✅ |
| PV Ready | ✅ |

---

## 2. Systeemontwerp

### 2.1 Hydraulisch schema

```
ZONNEPANELEN
      ↓ surplus
WARMTEPOMP AMH6DC
      ↓
BT-500 BUFFERTANK (500L, verwarmingswater gesloten circuit)
      ├──→ Radiatoren (direct)
      └──→ DHW-voorbarmingsspiraal (koud leidingwater doorheen)
                    ↓
            Elektrische DHW-boiler (topt af naar 60°C)
                    ↓
                  Douche / kranen
```

### 2.2 Ontwerpkeuzes en motivatie

**Buffertank als thermische accu:**  
Dagelijks laadcyclus tijdens zonne-surplus (overdag), ontlading 's avonds/nacht en weekenden.

**DHW-configuratie:**  
Koud leidingwater stroomt door een spiraal *in* de buffertank, wordt voorverwarmd door het omringende bufferwater, en stroomt dan naar een aparte elektrische boiler die aftopt naar 60°C. Het bufferwater zelf blijft een gesloten, niet-drinkbaar verwarmingscircuit.

- Geen legionelarisico in buffertank (gesloten circuit, niet-drinkwater)
- Elektrische boiler op 60°C elimineert legionelarisico in DHW-vat
- Wanneer onbewoond: boiler UIT, vat gevuld met koud leidingwater (<25°C) — veilig
- Spiraalvolume (enkele liters) spoelen bij ingebruikname na langere stilstand

**Buffergrootte 500L — onderbouwing:**

| Parameter | Waarde |
|---|---|
| Bruikbare ΔT (dagcyclus, 70°C→35°C) | 35°C |
| Opgeslagen energie | 500 × 4,18 × 35 = **~20 kWh** |
| Laadtijd bij 5,50 kW HP-output | **~3,5 uur** |
| Staande verliezen (nacht, ~14u) | ~1,1 kWh |
| Netto bruikbaar 's nachts | **~19 kWh** |
| Weekendgebruik optioneel | Buffer per vrijdag volladen |

500L is correct gedimensioneerd voor een label C/D woning (nachtvraag ~15–21 kWh) met gecombineerd verwarmings- en DHW-voorverwarmingsgebruik.

### 2.3 Spiraal DHW-voorverwarming — verwachte uitvoertemperatuur

| Vulsnelheid | Warmteoverdracht | Uitvoertemperatuur |
|---|---|---|
| Langzaam (2 L/min, boilernavulling) | ~31 kW beschikbaar, 6,7 kW nodig | **~55–62°C** |
| Snelle doorstroming (10 L/min, direct douchegebruik) | ~8–12 kW leverbaar | **~25–35°C** |

Conclusie: spiraal werkt als **langzame voorlader** van de elektrische boiler, niet als directe doorstroomverwarmer. Optimale bediening: boiler vult langzaam bij via vlotterkraan of doorstroombegrenzer.

---

## 3. Regeling

### 3.1 SG Ready — modi

De AMH6DC heeft twee binaire SG Ready-contacten die vier bedrijfsmodi definiëren:

| Modus | Contact 1 | Contact 2 | Beschrijving |
|---|---|---|---|
| 1 | Open | Open | **Geblokkeerd** — compressor uit |
| 2 | Gesloten | Open | **Normaal** — HP volgt eigen regeling |
| 3 | Open | Gesloten | **Zonne-surplus** — hogere setpoint, laad buffer bij voorkeur |
| 4 | Gesloten | Gesloten | **Maximum** — vollast, hoogste temperatuur |

> **Belangrijk:** het exacte gedrag van modi 3 en 4 (temperatuuroffset, etc.) is fabrikantspecifiek. Verifieer met Renova/Midea-documentatie of via meting.

**In dit project: SG Ready-contacten worden overbrugd op modus 2 (normaal).  
Alle sturing verloopt via Modbus.**

### 3.2 Modbus-verbinding

| Parameter | Waarde |
|---|---|
| Fysieke interface | RS-485 |
| Aansluiting | **H1 = minus (−)**, **H2 = plus (+)** |
| Baudrate | 9600 bps |
| Databits | 8 |
| Pariteit | Geen |
| Stopbits | 1 |
| Protocol | Modbus RTU (ASCII niet ondersteund) |
| Standaard Slave ID | 1 |
| Ondersteunde functiecodes | 03H (lezen), 06H (schrijf enkel register), 10H (schrijf meerdere registers) |

**Hardware:** RS-485/USB-adapter (~€10) aangesloten op Raspberry Pi met Home Assistant of Node-RED.

---

## 4. Modbus Registerkaart — Midea 120L V4.1

> **Bron:** Fabrikantsdocument `0052003044313-V-D` (Midea 120L Modbus Mapping Table, versie V4.1)  
> **Zekerheid sensoren (lezen):** ~95%  
> **Zekerheid schrijfregisters:** ~80% — verifieer altijd door teruggelezen waarde te vergelijken met display

### 4.1 Besturingsregisters (Read/Write) — Reg 0–20

#### Register 0 — Aan/Uit (BIT-veld)

> ⚠️ **Nooit direct overschrijven.** Altijd Read-Modify-Write uitvoeren om andere bits te bewaren.

| Bit | Functie | Waarden |
|---|---|---|
| BIT0 | Zone 1/2 aan/uit (kamertemperatuurregeling) | 0=uit, 1=aan |
| BIT1 | Zone 1 aan/uit (watertemperatuurregeling) | 0=uit, 1=aan |
| BIT2 | DHW aan/uit | 0=uit, 1=aan |
| BIT3 | Zone 2 aan/uit (watertemperatuurregeling) | 0=uit, 1=aan |
| BIT5 | ECO-modus | 0=uit, 1=aan |
| BIT8 | Stille modus | 0=uit, 1=aan |
| BIT9 | Desinfectie | 0=uit, 1=aan |

**Zone 1 aanzetten (water flow control):** schrijf `current_reg0 | 0x0002`  
**Zone 1 uitzetten:** schrijf `current_reg0 & ~0x0002`

#### Registers 1–20 — Overige besturing

| Register | Omschrijving | Waarden / Bereik |
|---|---|---|
| 1 | Bedrijfsmodus | 1=Auto, 2=Koelen, 3=Verwarmen |
| 2 | Setpoint watertemperatuur T1s (Zone 1) | 20–75°C (direct in °C) |
| 3 | Setpoint watertemperatuur T1s2 (Zone 2) | 20–75°C |
| 4 | Setpoint luchttemperatuur Tas | °C |
| 5 | Setpoint tapwatertemperatuur T5s | 30–70°C |
| 6 | Functie-instelling (BIT-veld) | zie document |
| 7 | Temperatuurcurveselectie | Curven 1–9 |
| 10 | t_SG_MAX — max. bedrijfstijd bij hoge elektriciteitsprijs | 0–24 uur |
| 11 | Aan/Uit Zone 1 (water flow) — alternatief voor reg 0 BIT1 | 0=uit, 1=aan |
| 12 | Aan/Uit Zone 2 (water flow) — alternatief voor reg 0 BIT3 | 0=uit, 1=aan |
| 14 | Aan/Uit Zone 1/2 (alternatief) | 0=uit, 1=aan |
| 15 | Aan/Uit DHW (alternatief) | 0=uit, 1=aan |
| 20 | Niveau stille modus | 0=level 1, 1=level 2, 2=Boost |

> **Tip:** registers 14–17 zijn functioneel equivalent aan register 0. Het document adviseert gebruik van 14–17 voor directe aansturing. In de huidige implementatie wordt reg 0 met RMW gebruikt.

### 4.2 Sensorregisters (Read Only) — Reg 100–127

| Register | Naam | Omschrijving | Eenheid / Schaal |
|---|---|---|---|
| 100 | comp_hz | Compressorfrequentie | Hz, directe waarde |
| 101 | bedrijfsmodus | Actuele bedrijfsmodus | 0=uit, 2=koelen, 3=verwarmen |
| 102 | fan_rpm | Ventilatorsnelheid | r/min |
| 103 | exv1_opening | Expansieventiel 1 | P |
| 104 | Tw_in | Waterinlaattemperatuur platenwarmtewisselaar (retour) | °C, s16 |
| 105 | Tw_out | Wateruitlaattemperatuur platenwarmtewisselaar (aanvoer) | °C, s16 |
| 106 | T3 | Condensatortemperatuur | °C, s16 |
| 107 | T4 | Buitenluchttemperatuur | °C, s16 |
| 108 | Tp | Compressor uitlaattemperatuur (discharge) | °C, s16 |
| 109 | Th | Compressor inzuigtemperatuur (suction) | °C, s16 |
| 110 | T1 | Totale uitvoerwatertemperatuur | °C, s16 |
| 111 | Tw2 | Watertemperatuur Zone 2 | °C, s16 (25=niet beschikbaar) |
| 112 | T2 | Koudemiddel vloeistofzijde | °C, s16 |
| 113 | T2B | Koudemiddel gaszijde | °C, s16 |
| 114 | Ta | Kamertemperatuur | °C, s16 (25=niet beschikbaar) |
| 115 | T5 | Tapwatertemperatuur (DHW-vat) | °C, s16 |
| 116 | P1 | Hoge druk koudemiddelcircuit | kPa |
| 117 | P2 | Lage druk koudemiddelcircuit | kPa |
| 118 | odu_stroom | Buitenunit bruto bedrijfsstroom | A |
| 119 | odu_spanning | Buitenunit bruto bedrijfsspanning | V |
| 120 | **Tbt1** | **Buffertank BOVEN temperatuur** | °C, s16 |
| 121 | **Tbt2** | **Buffertank ONDER temperatuur** | °C, s16 |
| 122 | comp_uren | Compressor bedrijfstijd | uur |
| 123 | unit_vermogen | Eenheidsvermogen | kW |
| 124 | **foutcode** | **Actuele foutcode** | 0=OK, >0=storingscode |

> **Tbt1 (reg 120) en Tbt2 (reg 121)** zijn direct de buffertanksensoren van de BT-500 — mits aangesloten op de HP-controller ingangen. Geen externe DS18B20 sensoren nodig.

### 4.3 Statusregisters (Read Only) — Reg 128–129

| Register | Naam | Relevante bits |
|---|---|---|
| 128 | Status bit 1 | BIT0=olieterugvoer, BIT1=antibevriezing, BIT2=ontdooien, BIT5=DHW actief, BIT6=verwarming actief, BIT7=koelen actief |
| 129 | Belastinguitgang | BIT0=IBH1, BIT1=IBH2, BIT2=TBH, BIT3=interne pomp, BIT5=externe pomp, BIT6=AHS |

### 4.4 Energie & COP registers (Read Only) — Reg 148–155

> ⚠️ **R290-eenheden:** werkelijke waarde = registerwaarde ÷ 100

| Register | Omschrijving | Eenheid |
|---|---|---|
| 148 | Actueel verwarmingsvermogen | kW (÷100) |
| 149 | Actueel hernieuwbaar verwarmingsvermogen | kW (÷100) |
| 150 | Actueel elektrisch verbruik verwarming | kW (÷100) |
| **151** | **Actuele COP verwarming** | **dimensieloos (÷100)** |
| 152 | Actueel koelvermogen | kW (÷100) |
| 153 | Actueel hernieuwbaar koelvermogen | kW (÷100) |
| 154 | Actueel elektrisch verbruik koeling | kW (÷100) |
| 155 | Actuele EER koeling | dimensieloos (÷100) |

### 4.5 Cumulatieve energieregisters (Read Only) — Reg 152–197

Uitgebreide cumulatieve kWh-registers voor verwarming, koeling en DHW. Beschikbaar voor langetermijnenergielogging. Zie origineel document voor volledige lijst.

### 4.6 Parameterinstellingen (Read/Write) — Reg 200–289

Configuratieparameters zoals temperatuurlimieten, pompinstellingen, klimaatcurves. Alleen aanpassen via installateursmenu of gedocumenteerde schrijfopdrachten.

---

## 5. Foutcodes — Midea 120L (Tabel 1)

| Code | Waarde (reg 124) | Omschrijving |
|---|---|---|
| E0 | 1 | Waterstroomfout (E8 driemaal) |
| E2 | 3 | Communicatiefout controller ↔ hydraulische module |
| E3 | 4 | Einduitvoerwatertemperatuursensor T1 defect |
| E4 | 5 | Tapwatervatsensor T5 defect |
| E5 | 6 | Condensatoruitlaatkoelmediumsensor T3 defect |
| E6 | 7 | Buitenluchttemperatuursensor T4 defect |
| E7 | 8 | Buffertank boven sensor Tbt1 defect |
| E8 | 9 | Waterstroomstoring |
| E9 | 10 | Inzuigtemperatuursensor Th defect |
| EA | 11 | Uitlaattemperatuursensor Tp defect |
| Eb | 12 | Zonnecolletor sensor Tsolar defect |
| Ec | 13 | Buffertank onder sensor Tbt2 defect |
| Ed | 14 | Waterinlaattemperatuursensor Tw_in defect |
| EE | 15 | Hydraulische module EEPROM-storing |
| P0 | 20 | Lagedrukschakelaarbeveiliging |
| P1 | 21 | Hogedrukschakelaarbeveiliging |
| P3 | 23 | Compressor overcurrent |
| P4 | 24 | Hoge uitlaattemperatuurbeveiliging |
| P5 | 25 | \|Tw_out − Tw_in\| te groot |
| P6 | 26 | Omvormmodulebeveiliging |
| Pb | 31 | Antibevriezingsbedrijf |
| Pd | 33 | Hoge temperatuurbeveiliging koudemiddeluitlaat condensator |
| PP | 38 | Tw_out − Tw_in ongebruikelijke beveiliging |
| H0 | 39 | Communicatiefout hoofdprint PCB B ↔ hydraulische modulebesturing |
| H2 | 41 | Koudemiddel vloeistoftemperatuursensor T2 defect |
| H4 | 43 | Driemaal P6 (L0/L1) beveiliging |
| H6 | 45 | DC-ventilatormotor storing |
| H7 | 46 | Spanningsbeveiliging |
| HP | 57 | Lagedrukbeveiliging (Pe<0,6) driemaal per uur |

---

## 6. Node-RED Implementatie

### 6.1 Architectuur

```
SECTIE 1: MONITORING (elke 10 sec)
  Poll trigger
    ├──→ modbus-getter reg 100–127 (28x) → Parse Sensoren → Debug + sensors.csv
    ├──→ modbus-getter reg 0–7 (8x)      → Parse Controls → Debug + controls.csv
    └──→ modbus-getter reg 148–155 (8x)  → Parse Energie  → Debug + energy.csv

SECTIE 2: AANSTURING
  ▶ Efficient (45°C) ──┐
  ▶ Mid Power (65°C)  ──┤→ Bouw Setpoint → [modbus-flex-write, cmd log]
  ▶ Hoog (72°C) ──────┘
  
  ▶ HP Zone1 AAN ──┐→ Lees Reg 0 (keepMsgProperties:TRUE)
  ▶ HP Zone1 UIT ──┘     → Bouw Aan/Uit (RMW) → [modbus-flex-write, cmd log]
  
  modbus-flex-write → delay 500ms → Verificatie Lees Reg 0–2 → Verificeer → Debug + cmd log

SECTIE 3: FOUTAFHANDELING
  Catch + modbus error outputs → Fout Handler (met Midea foutcodevertaling) → Debug + errors.csv
```

### 6.2 Kritieke implementatiedetails

**Read-Modify-Write op register 0:**  
Register 0 is een 16-bit BIT-veld. Nooit direct overschrijven — altijd eerst lezen, bit aanpassen, terugschrijven. Anders worden ECO-modus, stille modus etc. gewist.

```javascript
// Zone 1 aanzetten:
newReg0 = currentReg0 | 0x0002;   // SET BIT1

// Zone 1 uitzetten:
newReg0 = currentReg0 & ~0x0002;  // CLEAR BIT1
newReg0 = newReg0 & 0xFFFF;       // Zorg voor 16-bit unsigned
```

**keepMsgProperties op de RMW getter:**  
De `modbus-getter` node voor het lezen van reg 0 (vóór de RMW-schrijfactie) moet `keepMsgProperties: true` hebben. Anders gaat `msg.topic` ("HP_AAN" / "HP_UIT") verloren en weet de schrijffunctie niet of hij aan of uit moet zetten.

**Temperatuurschaling:**  
- Setpointregisters (reg 2, 3, 4, 5): directe °C-waarden, geen ×10 schaling
- Sensorregisters (reg 100+): directe °C, geen schaling
- Energieregisters (reg 148–155): **÷100** voor R290-eenheden
- Negatieve temperaturen (buitenlucht): two's complement, `v > 32767 ? v - 65536 : v`

**Polling interval:** minimaal 5–10 seconden om de interne CPU van de warmtepomp niet te overbelasten.

### 6.3 Logbestanden

| Bestand | Inhoud | Interval |
|---|---|---|
| `amh6dc_sensors.csv` | Alle sensorwaarden (temps, druk, stroom, buffertank) | 10 sec |
| `amh6dc_controls.csv` | Besturingsstatus (aan/uit, setpoints, modi) | 10 sec |
| `amh6dc_energy.csv` | Actueel vermogen, verbruik en COP | 10 sec |
| `amh6dc_commands.csv` | Elke schrijfopdracht + verificatieresultaat | Per opdracht |
| `amh6dc_errors.csv` | Alle Modbus-fouten met tijdstempel en bron | Per fout |

### 6.4 Configuratie modbus-client

| Parameter | Waarde |
|---|---|
| Type | Serieel (RTU) |
| Seriële poort | `/dev/ttyUSB0` |
| Baudrate | 9600 |
| Databits | 8 |
| Stopbits | 1 |
| Pariteit | Geen |
| Unit ID | 1 |
| commandDelay | 100 ms |
| parallelUnitIdsAllowed | false |

---

## 7. Beveiliging & Onzekerheden

### 7.1 Nog te verifiëren bij eerste verbinding

| Check | Methode |
|---|---|
| Registerkaart correct? | Lees reg 107 (T4 buiten), vergelijk met actuele buitentemperatuur |
| Setpointschaling juist? | Schrijf 45 naar reg 2, lees terug, controleer op display |
| BIT-veld reg 0 correct? | Lees reg 0, decodeer bits, vergelijk met display |
| COP-schaling? | Vergelijk reg 151 ÷ 100 met display-COP tijdens bedrijf |
| Tbt1/Tbt2 aangesloten? | Controleer of reg 120/121 realistische waarden geven |

### 7.2 Bekende onzekerheden

- Registerkaart is gebaseerd op Midea 120L V4.1 — AMH6DC kan een licht afwijkende firmwareversie hebben
- Register 0 BIT-definitie voor ECO-modus (BIT5) — niet bevestigd, observeren
- Setpoint bovengrens reg 2: document zegt max 60°C voor sommige configuraties — 75°C in setpointformulier is HP-spec, verifieer bij praktijktest
- Registers 23–25 (index 7–9 in het 28x leesblok) zijn onbekend — observeer waarden tijdens bedrijf

### 7.3 Legionelabeheersing

- Buffertank: gesloten circuit, niet-drinkwater — geen legionelarisico
- DHW-spiraal: koud leidingwater stroomt door → spoelen eerste 2L bij gebruik na stilstand
- Elektrische boiler: 60°C setpoint elimineert legionelarisico tijdens bewoning
- Bij onbewoond: boiler UIT, vat koud (<25°C) — veilig

---

## 8. Hardware Samenvatting

| Component | Specificatie | Opmerking |
|---|---|---|
| Warmtepomp | GreenPump AMH6DC, 6kW R290 | Midea 120L platform |
| Buffertank | NEXUS BT-500, 500L RVS SUS304 | 8× G1¼" aansluitingen, sensorbuizen |
| DHW-spiraal | Interne spiraal in buffertank | Koud leidingwater door spiraal |
| Elektrische boiler | Separaat, elektrisch, 60°C setpoint | Alleen aan bij bewoning |
| RS-485 adapter | USB naar RS-485, ~€10 | `/dev/ttyUSB0` |
| Controller | Raspberry Pi + Node-RED of Home Assistant | node-red-contrib-modbus 5.45.2 |

---

## 9. Aansluiting RS-485

```
Warmtepomp aansluitstrook:
  H1  →  minus (−)  →  RS-485 B
  H2  →  plus  (+)  →  RS-485 A

Kabel: afgeschermd twisted pair (Cat5e of dedicated RS-485)
Max. kabellengte: 50m (per Midea specificatie)
Afscherming: aarden aan één kant
```

---

*Document gegenereerd op basis van projectbevindingen 18 mei 2026.*  
*Registerkaart: Midea document 0052003044313-V-D, versie V4.1.*  
*Node-RED flow: amh6dc_nodered_flow_v5.json*
