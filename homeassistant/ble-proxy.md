# BLE Proxy + PIR — ESP32 multisenzor pre Home Assistant

BLE proxy, PIR pohybový senzor a voliteľný senzor osvetlenia v jednej inštalačnej krabici.
Zabudované v Legrand Valena Life rámčeku (zaslepovacia pozícia).

---

## Prečo

**BLE dosah:** RPi4 má vstavaný BT adaptér (bcm43438) s dosahom ~10-15m. Vzdialené BLU H&T senzory majú slabý signál:

| Zariadenie | Signál cez RPi4 |
|------------|-----------------|
| H&T / Kuchyňa | -56 dBm (OK) |
| H&T / Druhé poschodie | -87 dBm (slabý) |
| H&T / BLU H&T 0002 | -90 dBm (veľmi slabý) |

**Pohybový senzor:** V kúpeľni nie je žiadny senzor pohybu — automatické zapínanie svetla pri vstupe.

**Riešenie:** ESP32 s BLE proxy + PIR senzor v jednej krabici, schované v Legrand Valena Life rámčeku.

---

## Hardvér

### Čo treba kúpiť

| Komponent | Model | Cena (~) | Poznámka |
|-----------|-------|----------|----------|
| ESP32 doska | M5Stack ATOM Lite | ~8€ | ESP32, WiFi + BLE, USB-C, 24×24mm |
| AC/DC zdroj | Hi-Link HLK-5M03 | ~5€ | 230V AC → 3.3V DC, 5W |
| PIR senzor | **SR602** | ~1€ | 3.3V, 10×23mm, dosah 3m, retriggering |
| Senzor osvetlenia | BH1750 (voliteľné) | ~2€ | I2C, meranie lux |
| Rámček | Legrand Valena Life | — | Zaslepovacia pozícia pre PIR |

### Príslušenstvo

- Wago svorky (pripojenie 230V)
- Scvrkávacia bužírka (izolácia 5V spojov)
- Krátke vodiče na prepojenie

### Náradia

- Spájkovačka + cín (prepojenie Hi-Link → ATOM → SR602)
- Skúšačka / multimeter

### Prečo SR602 (porovnanie zvažovaných senzorov)

| Senzor | Napájanie | Veľkosť | Retriggering | Dosah | Záver |
|--------|-----------|---------|--------------|-------|-------|
| HC-SR501 (modrá/zelená) | 5-20V | 32×24mm | áno (nast.) | 3-7m (nast.) | Príliš veľký do krabice |
| HC-SR505 | 5V | stredný | áno | ~3m | OK, ale 5V |
| AM312 | 3.3V | 10×10mm | **nie** | ~3m | Bez retriggering — svetlo by blikalo |
| **SR602** | **3.3V** | **10×23mm** | **áno** | **~3m** | **Ideálny — malý, 3.3V, retriggering** |

**Retriggering** = senzor predlžuje HIGH signál kým je pohyb. Bez toho by svetlo zhasínalo každé 2s a znova svietilo — čo nie je použiteľné.

---

## Fyzická inštalácia

### Umiestnenie v Legrand Valena Life rámčeku

```
┌─────────────────────────────┐
│  Legrand Valena Life rámček │
│                             │
│  ┌───────┐    ┌───────┐    │
│  │Vypínač│    │Zaslepka│    │
│  │svetla │    │+ PIR   │    │
│  │       │    │ (otvor)│    │
│  └───────┘    └───────┘    │
│                             │
│  Za stenou:   Za stenou:    │
│  Shelly 1     Hi-Link       │
│  Mini Gen3    ATOM Lite     │
│               AM312         │
└─────────────────────────────┘
```

PIR senzor (AM312) sa schová za malý otvor v zaslepovacom kryte — senzor je len 10×10mm.
Fáza + nulák sa berú z rovnakého prívodu ako svetlo.

### Zapojenie v inštalačnej krabici

```
┌──────────────────────────────────────────┐
│  Inštalačná krabica (230V)               │
│                                          │
│  L (fáza) ──┬──── Shelly 1 Mini Gen3    │
│              │     (svetlo kúpeľňa)      │
│              │                           │
│              │  ┌──────────┐             │
│              ├──┤ Hi-Link  │             │
│              │  │ HLK-PM01 │             │
│  N (nulák) ─┤  │ 230V→5V  │             │
│              │  └──┬───┬───┘             │
│              │     │   │                 │
│              │   +5V  GND                │
│              │     │   │                 │
│              │  ┌──┴───┴───┐   ┌──────┐ │
│              │  │ ATOM Lite│───┤AM312 │ │
│              │  │          │   │ PIR  │ │
│              │  │ BLE proxy│   └──────┘ │
│              │  │ + PIR    │             │
│              │  └──────────┘             │
│              │                           │
└──────────────────────────────────────────┘
```

### Prepojenie

| Zdroj | Cieľ | Pin |
|-------|------|-----|
| Hi-Link +Vo (5V) | ATOM Lite | 5V |
| Hi-Link -Vo (GND) | ATOM Lite | GND |
| ATOM Lite 3.3V | SR602 | VCC |
| ATOM Lite GND | SR602 | GND |
| SR602 OUT | ATOM Lite | GPIO32 |
| ATOM Lite GPIO25 (voliteľné) | BH1750 SDA | SDA |
| ATOM Lite GPIO21 (voliteľné) | BH1750 SCL | SCL |

### Bezpečnostné upozornenia

- 230V a 5V časť musia byť fyzicky oddelené (dodržať vzdialenosti)
- Hi-Link HLK-PM01 je certifikovaný na použitie v uzavretých priestoroch
- Spoje na 230V strane cez Wago svorky, nie skrútené
- 5V spoje spájkované a zaizolované bužírkou
- Krabica musí byť zatvorená — nikdy neponechať otvorenú s 230V

---

## Softvér — ESPHome

### 1. Nainštalovať ESPHome add-on

V HA: **Settings → Add-ons → Add-on Store → ESPHome → Install**

### 2. Prvý flash cez USB

Pripojiť ATOM Lite cez USB-C k počítaču a flashnúť cez ESPHome dashboard alebo https://web.esphome.io

### 3. ESPHome konfigurácia

```yaml
esphome:
  name: ble-proxy-kupelna
  friendly_name: "BLE Proxy / Kúpeľňa"

esp32:
  board: m5stack-atom
  framework:
    type: esp-idf

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  manual_ip:
    static_ip: 192.168.88.250
    gateway: 192.168.88.1
    subnet: 255.255.255.0

  ap:
    ssid: "BLE-Proxy-Kupelna"
    password: "fallback123"

captive_portal:

logger:

api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password

# BLE proxy pre Shelly BLU H&T
bluetooth_proxy:
  active: true

esp32_ble_tracker:
  scan_parameters:
    active: true
    interval: 1100ms
    window: 1100ms

# PIR pohybový senzor (AM312 na GPIO32)
binary_sensor:
  - platform: gpio
    pin:
      number: GPIO32
      mode: INPUT_PULLDOWN
    name: "Kúpeľňa pohyb"
    device_class: motion
    filters:
      - delayed_off: 30s

# Voliteľné: senzor osvetlenia (BH1750 cez I2C)
# i2c:
#   sda: GPIO25
#   scl: GPIO21
#
# sensor:
#   - platform: bh1750
#     name: "Kúpeľňa osvetlenie"
#     address: 0x23
#     update_interval: 10s
```

### 4. Po flashnutí

1. ATOM sa pripojí na WiFi
2. HA automaticky detekuje nový ESPHome zariadenie
3. V **Settings → Devices & Services → Bluetooth** sa objaví nový BLE adaptér
4. BLU H&T senzory v dosahu budú mať lepší signál

---

## Automatizácia — svetlo v kúpeľni

Po inštalácii pridať do `/config/packages/kupelna_pohyb.yaml`:

```yaml
automation:
  - id: kupelna_svetlo_zapnut
    alias: "Kúpeľňa / Svetlo zapnúť pri pohybe"
    triggers:
      - trigger: state
        entity_id: binary_sensor.kupelna_pohyb
        to: "on"
    conditions:
      - condition: state
        entity_id: light.svetlo_kupelna
        state: "off"
    actions:
      - action: light.turn_on
        target:
          entity_id: light.svetlo_kupelna

  - id: kupelna_svetlo_vypnut
    alias: "Kúpeľňa / Svetlo vypnúť bez pohybu"
    triggers:
      - trigger: state
        entity_id: binary_sensor.kupelna_pohyb
        to: "off"
        for:
          minutes: 5
    actions:
      - action: light.turn_off
        target:
          entity_id: light.svetlo_kupelna
```

> **Poznámka:** `light.svetlo_kupelna` ešte neexistuje — pridať keď bude Shelly 1 Mini Gen3 nainštalovaný.

---

## Po inštalácii

### Overenie v HA

- **Settings → Devices & Services → ESPHome** — zariadenie online
- **Settings → Devices & Services → Bluetooth** — nový adaptér "BLE Proxy / Kúpeľňa"
- **binary_sensor.kupelna_pohyb** — testovať mávnutím pred PIR
- Signál H&T / Spodná kúpeľňa by sa mal zlepšiť (bližšie k -50/-60 dBm)

### Overenie cez CLI

```bash
ha-api search ble_proxy
ha-api search kupelna_pohyb
ha-ws device list esphome
```

---

## Umiestnenie

### Kúpeľňa (primárne)

Spodná kúpeľňa — Legrand Valena Life rámček vedľa vypínača svetla:
- BLE proxy pre H&T / Spodná kúpeľňa
- PIR pre automatické svetlo

### Ďalšie možné umiestnenia (budúcnosť)

Rovnaký setup sa dá replikovať:
- **2. poschodie / Chodba** — pokrytie detských izieb (H&T -87 dBm) + svetlo na chodbe
- **Zádverie** — automatické svetlo pri príchode
