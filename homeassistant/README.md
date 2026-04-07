# HA Rebjak - Konfigurácia Home Assistant

Konfiguračné súbory pre Home Assistant inštaláciu na adrese Dargov 189 / Pod dubinami 58/42.

## Štruktúra súborov

```
homeassistant/
├── configuration.yml       → hlavný konfiguračný súbor (skopírovať do /config/configuration.yaml)
├── energie_faktury.yaml    → história faktúr, ceny energií, odhadované náklady
└── README.md               → tento súbor
```

---

## Prehľad inštalácie

545 entít, 12 oblastí, všetko postavené na **Shelly** zariadeniach + **EMS-ESP** (kotol).

### Oblasti

Chodba, Detská izba juhozápad, Detská izba sever, Hosťovská, Kuchyňa, Obývačka, Schodisko, Spodná kúpeľňa, Spálňa, Technická miestnosť, Zádverie, Záhrada

### Zariadenia

#### Svetlá (10× Shelly 1 Mini Gen3 + 1× Shelly Wall Display)

| Zariadenie | Oblasť |
|------------|--------|
| Svetlo / Obývačka | Obývačka (Shelly Wall Display) |
| Svetlo / Kuchyňa | Kuchyňa |
| Svetlo / Schody | Schodisko |
| Svetlo / Zádverie (101.201) | Zádverie |
| Svetlo / Technická miestnosť | Technická miestnosť |
| Svetlo / Spálňa | Spálňa |
| Svetlo / Chodba 2P | Chodba |
| Svetlo / Hosťovská (106.201) | Hosťovská |
| Svetlo / Detská izba (206.201) | Detská izba juhozápad |
| Svetlo / Detská izba sever (205.201) | Detská izba sever |

#### Žalúzie (5× Shelly 2PM Gen3)

| Zariadenie | Oblasť |
|------------|--------|
| Žalúzia / Obývačka | Obývačka |
| Žalúzia / Spálňa | Spálňa |
| Žalúzia / Hosťovská | Hosťovská |
| Žalúzia / Detská izba juhozápad | Detská izba juhozápad |
| Žalúzia / Detská izba sever | Detská izba sever |

#### Kúrenie — ventily (1× Shelly Pro 3 + 1× Shelly Pro 4PM)

| Zariadenie | Oblasť |
|------------|--------|
| Kúrenie / Technická miestnosť | Technická miestnosť |
| Kúrenie / Chodba a zádverie | Zádverie |
| Kúrenie / Hosťovská | Hosťovská |
| Kúrenie / Kuchyňa | Kuchyňa |
| Kúrenie / Obývačka | Obývačka |
| Kúrenie / Kúpeľňa | Spodná kúpeľňa |
| Kúrenie / Kúpeľňa rebrík [Nepripojené] | Spodná kúpeľňa |

#### Meranie spotreby

| Zariadenie | Model | Oblasť |
|------------|-------|--------|
| Meranie / Hlavný prívod (109.1) | Shelly Pro 3EM (3 fázy) | Technická miestnosť |
| Zásuvka / Plynový kotol (109.107) | Shelly PM Mini Gen3 | Technická miestnosť |
| Zásuvka / Grundfos čerpadlo | Shelly Plus 1PM | Technická miestnosť |
| Zásuvka / Chladnička | Shelly PM Mini Gen3 | Kuchyňa |
| Zásuvka / Mraznička | Shelly PM Mini Gen3 | Kuchyňa |
| Zásuvka / Sušička | Shelly PM Mini Gen3 | Technická miestnosť |
| Zásuvka / Dátový rack | Shelly PM Mini Gen3 | Technická miestnosť |
| Zásuvka / Hosťovská (106.102) | Shelly PM Mini Gen3 | Hosťovská |

#### Teplota a vlhkosť (7× Shelly BLU H&T)

| Zariadenie | Oblasť |
|------------|--------|
| H&T / Kuchyňa | Kuchyňa |
| H&T / Spálňa | Spálňa |
| H&T / Hosťovská | Hosťovská |
| H&T / Detská izba severovýchod | Detská izba sever |
| H&T / Druhé poschodie | Detská izba sever |
| H&T / Detská izba juhozápad | Detská izba juhozápad |
| H&T / Spodná kúpeľňa | Spodná kúpeľňa |

#### Kúrenie — kotol a termostat (EMS-ESP cez MQTT)

| Zariadenie | Model |
|------------|-------|
| ems-esp Boiler | Bosch Logamax Plus GB122 / Junkers Cerapur GC2200W |
| ems-esp Thermostat | Bosch CW100 / CR120 |

#### Spínače (2× Shelly Pro 3 — nepomenované)

`shellypro3-2cbcbba0b450` (3 výstupy), `shellypro3-2cbcbba67034` (3 výstupy)

#### Ostatné

| Zariadenie | Integrácia | Popis |
|------------|------------|-------|
| Toyota Corolla HB/TS '23 | toyota | Dvere, zámky, okná, klimatizácia |
| NVR (3 kamery) | ONVIF | Detekcia pohybu, tamper detection |
| JR (iPhone 14 Pro) | mobile_app | Tracking, batéria, aktivita, focus |
| Mikrotik Router | HACS | Sieťový monitoring |
| Raspberry Pi (BT gateway) | bluetooth | BLU H&T BT proxy |

### Integrácie

| Integrácia | Účel |
|------------|------|
| Shelly | Svetlá, žalúzie, kúrenie, meranie, spínače |
| MQTT (EMS-ESP) | Plynový kotol Bosch + termostat |
| BTHome / Bluetooth | Shelly BLU H&T senzory teploty a vlhkosti |
| Toyota | Auto — stav dverí, zámkov, okien, klíma |
| ONVIF | NVR — 3 kamery |
| Met.no | Predpoveď počasia |
| InfluxDB | Logovanie dát |
| MQTT (Mosquitto) | MQTT broker |
| Google Translate TTS | Text-to-speech |
| SMS.to | SMS notifikácie |
| Mobile App | iPhone tracking |
| HACS | Mikrotik, Power Outages, Multiscrape, Network Scanner a frontend karty |

### HACS — Frontend karty

Mushroom, Bubble Card, ApexCharts, Button Card, Mini Graph Card, Power Flow Card Plus, Advanced Camera Card, Banner Card, Heat Pump Flow Card, TrashCard, Layout Card, HA Floorplan, Mushroom Dashboard Strategy

---

## Inštalácia

### 1. Hlavný konfiguračný súbor

Skopíruj obsah `configuration.yml` do `/config/configuration.yaml` na HA.

Obsahuje:
- HTTP nastavenia (Nginx Proxy Manager)
- InfluxDB integrácia
- REST senzory: Open-Meteo (predpoveď), METAR LZKZ (letisko Košice)
- Template senzory: vietor, poryvy, teplota, plyn m³
- Utility meter: plyn a elektrina (dnes / mesiac / rok)

### 2. Balík pre energie a faktúry

```bash
mkdir -p /config/packages
cp energie_faktury.yaml /config/packages/energie_faktury.yaml
```

Do `/config/configuration.yaml` pridaj (ak tam ešte nie je):
```yaml
homeassistant:
  packages:
    energie_faktury: !include packages/energie_faktury.yaml
```

Potom spusti plný reštart:
```bash
ha core restart --no-progress --raw-json
```

---

## Senzory po inštalácii

### Počasie
| Senzor | Zdroj | Popis |
|--------|-------|-------|
| `sensor.vietor_rychlost` | Open-Meteo | Predpoveď rýchlosti vetra km/h |
| `sensor.vietor_poryvy` | Open-Meteo | Predpoveď poryvov km/h |
| `sensor.vietor_smer` | Open-Meteo | Smer vetra ° |
| `sensor.teplota_vonku` | Open-Meteo | Vonkajšia teplota °C |
| `sensor.metar_rychlost_vetra` | METAR LZKZ | Reálne meranie letisko Košice km/h |
| `sensor.metar_teplota` | METAR LZKZ | Reálna teplota letisko °C |

### Spotreba energií
| Senzor | Popis |
|--------|-------|
| `sensor.plyn_dnes_kwh` | Plyn dnes kWh (reset polnoc) |
| `sensor.plyn_mesiac_kwh` | Plyn tento mesiac kWh (reset 1. v mesiaci) |
| `sensor.plyn_rok_kwh` | Plyn tento rok kWh (reset 1.1.) |
| `sensor.plyn_dnes_m3` | Plyn dnes m³ |
| `sensor.plyn_mesiac_m3` | Plyn tento mesiac m³ |
| `sensor.plyn_rok_m3` | Plyn tento rok m³ |
| `sensor.elektrina_dnes` | Elektrina dnes kWh |
| `sensor.elektrina_mesiac` | Elektrina tento mesiac kWh |
| `sensor.elektrina_rok` | Elektrina tento rok kWh |

### Odhadované náklady v EUR
| Senzor | Popis |
|--------|-------|
| `sensor.plyn_dnes_eur` | Odhadovaný náklad za plyn dnes |
| `sensor.plyn_mesiac_eur` | Odhadovaný náklad za plyn tento mesiac (vrátane fixnej platby) |
| `sensor.plyn_rok_eur` | Odhadovaný náklad za plyn tento rok |
| `sensor.elektrina_dnes_eur` | Odhadovaný náklad za elektrinu dnes |
| `sensor.elektrina_mesiac_eur` | Odhadovaný náklad za elektrinu tento mesiac (vrátane fixnej platby) |
| `sensor.elektrina_rok_eur` | Odhadovaný náklad za elektrinu tento rok |

---

## Aktualizácia cien po novej faktúre

Ceny sa nastavujú v HA UI — nie je potrebné editovať YAML.

**Settings → Devices & Services → Helpers** alebo priamo v dashboarde:

| Helper | Aktuálna hodnota | Popis |
|--------|-----------------|-------|
| `input_number.cena_plyn_kwh` | 0.0351 €/kWh | Variabilná cena plynu s DPH |
| `input_number.cena_plyn_fixna_mesiac` | 1.845 €/mes | Fixná mesačná platba plyn |
| `input_number.cena_elektrina_kwh` | 0.1096 €/kWh | Cena elektriny VT s DPH |
| `input_number.cena_elektrina_fixna_mesiac` | 1.785 €/mes | Fixná mesačná platba elektrina |

---

## História vyúčtovacích faktúr

Faktúry sa zadávajú manuálne cez HA UI (Helpers → input_text) po každom vyúčtovaní.

VSE posiela ročné vyúčtovanie vždy **okolo 16. septembra**. HA automaticky upozorní 1. septembra.

### Plyn (Energetika Slovensko / VSE)
| Obdobie | Spotreba | Suma | Výsledok |
|---------|----------|------|---------|
| 27.02.2024 – 03.09.2024 | — | 51,93 € | preplatok 18,07 € |
| 04.09.2024 – 09.09.2025 | 12 480 kWh (~1 183 m³) | 750,67 € | nedoplatok 260,67 € |
| Preddavky 2025/2026 | — | 69,40 €/mes | — |

### Elektrina (Energetika Slovensko / VSE)
| Obdobie | Spotreba | Suma | Výsledok |
|---------|----------|------|---------|
| 28.02.2024 – 09.09.2024 | 102 kWh | 65,94 € | preplatok 114,06 € |
| 10.09.2024 – 01.09.2025 | 920 kWh | 246,56 € | preplatok 243,44 € |
| Preddavky 2025/2026 | — | 50,00 €/mes | — |

---

## Riešenie problémov

### Template senzory ukazujú `unknown` po reštarte

REST senzory (Open-Meteo, METAR) sa načítajú až po prvom fetch cykle. Spusti:
```bash
ha-api call template reload
```
Alebo počkaj max 15 minút — senzory sa naplnia automaticky.

### Utility meter senzory neexistujú

`utility_meter` vyžaduje plný reštart, nie len reload:
```bash
ha core restart --no-progress --raw-json
```

### Plyn mesačný / ročný ukazuje zlú hodnotu

Pri prvom spustení môže meter obsahovať starú kumulovanú hodnotu. Resetuj manuálne:
**Developer Tools → States → nájdi sensor → Reset**
