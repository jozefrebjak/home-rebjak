# BLE Proxy — ESP32 Bluetooth Proxy pre Home Assistant

Rozšírenie BLE dosahu pre Shelly BLU H&T senzory pomocou ESP32 s ESPHome.

---

## Prečo

RPi4 má vstavaný BT adaptér (bcm43438) s dosahom ~10-15m. Vzdialené BLU H&T senzory majú slabý signál:

| Zariadenie | Signál cez RPi4 |
|------------|-----------------|
| H&T / Kuchyňa | -56 dBm (OK) |
| H&T / Druhé poschodie | -87 dBm (slabý) |
| H&T / BLU H&T 0002 | -90 dBm (veľmi slabý) |

ESP32 BLE proxy na 2. poschodí vyrieši pokrytie pre vzdialené senzory.

---

## Hardvér

### Čo treba kúpiť

| Komponent | Model | Cena (~) | Odkiaľ |
|-----------|-------|----------|--------|
| ESP32 doska | M5Stack ATOM Lite | ~8€ | AliExpress |
| AC/DC zdroj | Hi-Link HLK-PM01 (230V AC → 5V DC, 3W) | ~4€ | AliExpress |

### Voliteľné

- Scvrkávacia bužírka na izoláciu spojov
- Wago svorky na pripojenie 230V v krabici

### Náradia

- Spájkovačka + cín (prepojenie Hi-Link → ATOM)
- Skúšačka / multimeter

---

## Zapojenie v inštalačnej krabici

```
┌─────────────────────────────────────┐
│  Inštalačná krabica (230V)          │
│                                     │
│  L (fáza) ───┐                      │
│               │  ┌──────────┐       │
│               ├──┤ Hi-Link  │       │
│               │  │ HLK-PM01 │       │
│  N (nulák) ──┘  │          │       │
│                  │ 230V→5V  │       │
│                  └──┬───┬───┘       │
│                     │   │           │
│                   +5V  GND          │
│                     │   │           │
│                  ┌──┴───┴───┐       │
│                  │  M5Stack │       │
│                  │ ATOM Lite│       │
│                  │          │       │
│                  │ (WiFi+BLE│       │
│                  │  proxy)  │       │
│                  └──────────┘       │
│                                     │
└─────────────────────────────────────┘
```

### Prepojenie Hi-Link → ATOM Lite

| Hi-Link výstup | ATOM Lite pin |
|-----------------|---------------|
| +Vo (5V) | 5V pin (Grove port alebo pájacie body) |
| -Vo (GND) | GND pin |

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
  name: ble-proxy-2p
  friendly_name: "BLE Proxy / 2. poschodie"

esp32:
  board: m5stack-atom
  framework:
    type: esp-idf

wifi:
  ssid: !secret wifi_ssid
  password: !secret wifi_password

  # Statická IP (voliteľné, odporúčané)
  manual_ip:
    static_ip: 192.168.88.250
    gateway: 192.168.88.1
    subnet: 255.255.255.0

  ap:
    ssid: "BLE-Proxy-2P"
    password: "fallback123"

captive_portal:

logger:

api:
  encryption:
    key: !secret api_key

ota:
  - platform: esphome
    password: !secret ota_password

bluetooth_proxy:
  active: true

esp32_ble_tracker:
  scan_parameters:
    active: true
    interval: 1100ms
    window: 1100ms
```

### 4. Po flashnutí

1. ATOM sa pripojí na WiFi
2. HA automaticky detekuje nový ESPHome zariadenie
3. V **Settings → Devices & Services → Bluetooth** sa objaví nový BLE adaptér
4. BLU H&T senzory v dosahu budú mať lepší signál

---

## Po inštalácii

### Overenie v HA

- **Settings → Devices & Services → ESPHome** — zariadenie online
- **Settings → Devices & Services → Bluetooth** — nový adaptér "BLE Proxy / 2. poschodie"
- Signál vzdialených BLU H&T by sa mal zlepšiť (bližšie k -50/-60 dBm)

### Overenie cez CLI

```bash
ha-api search ble_proxy
ha-ws device list esphome
```

---

## Umiestnenie

ESP32 umiestniť na 2. poschodie — pokryje:
- Detská izba sever (H&T / Druhé poschodie — aktuálne -87 dBm)
- Detská izba juhozápad (H&T / Detská izba juhozápad)
- Spálňa
- Chodba 2P

Optimálne umiestnenie: centrálne na 2. poschodí (napr. chodba).
