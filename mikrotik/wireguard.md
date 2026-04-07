# WireGuard VPN — Mikrotik RB2011UiAS + macOS

Konfigurácia WireGuard VPN pre vzdialený prístup do LAN `192.168.88.0/24`.

- **Router:** Mikrotik RB2011UiAS, RouterOS 7.18+
- **Klient:** macOS (WireGuard z App Store)
- **WireGuard subnet:** `10.10.10.0/24`
- **Port:** `51820/udp`

---

## Princíp

WireGuard funguje na výmene **public keys** medzi dvoma stranami:

- Mikrotik má svoj key pair (vygeneruje sa automaticky pri vytvorení interface)
- Mac má svoj key pair (vygeneruje sa v WireGuard appke)

**Private key sa nikdy nekopíruje — ostáva len v zariadení, ktoré ho vygenerovalo.**

Každá strana pozná len **public key** tej druhej.

---

## Postup krok za krokom

### Krok 1 — Mikrotik: vytvor WireGuard interface

```bash
/interface wireguard add name=wg0 listen-port=51820 mtu=1420
/ip address add address=10.10.10.1/24 interface=wg0
```

### Krok 2 — Mikrotik: zapíš si PUBLIC key

```bash
/interface wireguard print
```

Výstup obsahuje `public-key` — **zapíš si ho**, budeš ho potrebovať na Macu v kroku 6.

### Krok 3 — Mac: vytvor tunel

1. Otvor WireGuard appku
2. **Add Empty Tunnel** — vygeneruje sa Private Key + **Public Key**
3. **Zapíš si Public Key** — budeš ho potrebovať na Mikrotiku v kroku 4

### Krok 4 — Mikrotik: pridaj Mac ako peer

```bash
/interface wireguard peers add interface=wg0 \
  allowed-address=10.10.10.2/32 \
  public-key="MAC_PUBLIC_KEY_Z_KROKU_3"
```

> **POZOR:** sem patrí **PUBLIC** key z Macu, nikdy nie private!

### Krok 5 — Mikrotik: firewall pravidlá

Pravidlá musia byť **pred** drop pravidlami. Čísla `place-before` podľa aktuálneho firewallu:

```bash
# Povoliť WireGuard port na input (pred "drop all not coming from LAN")
/ip firewall filter add chain=input protocol=udp dst-port=51820 action=accept \
  comment="WireGuard" place-before=13

# Povoliť traffic z VPN do LAN (pred "drop all from WAN not DSTNATed")
/ip firewall filter add chain=forward src-address=10.10.10.0/24 action=accept \
  comment="WireGuard to LAN" place-before=14

# Povoliť DNS z WireGuard subnetu (UDP + TCP)
/ip firewall filter add chain=input protocol=udp src-address=10.10.10.0/24 dst-port=53 action=accept \
  comment="WireGuard DNS" place-before=15

/ip firewall filter add chain=input protocol=tcp src-address=10.10.10.0/24 dst-port=53 action=accept \
  comment="WireGuard DNS TCP" place-before=16
```

> **Poznámka:** Po pridaní každého pravidla sa čísla posunú o 1, preto `place-before` sa zvyšuje.
> Vždy over aktuálne poradie cez `/ip firewall filter print`.
> DNS pravidlá sú potrebné, lebo Mikrotik štandardne akceptuje DNS len z LAN, nie z WireGuard subnetu.

### Krok 6 — Mac: dokonči konfiguráciu tunela

V WireGuard appke doplň konfiguráciu:

```ini
[Interface]
PrivateKey = AUTOMATICKY_VYGENEROVANY_V_KROKU_3
Address = 10.10.10.2/32
DNS = 10.10.10.1

[Peer]
PublicKey = MIKROTIK_PUBLIC_KEY_Z_KROKU_2
Endpoint = VEREJNA_IP:51820
AllowedIPs = 192.168.88.0/24, 10.10.10.0/24
PersistentKeepalive = 25
```

### Krok 7 — Test

Aktivuj tunel na Macu a over:

```bash
ping 10.10.10.1       # WireGuard IP routera
ping 192.168.88.1     # LAN IP routera
```

---

## Overenie stavu

### Mikrotik

```bash
/interface wireguard print                # interface status
/interface wireguard peers print          # peers + last handshake
/ip firewall filter print                 # firewall pravidlá
```

### Mac

```bash
sudo wg show                              # stav tunela + last handshake
```

`last-handshake` musí mať hodnotu (nie `0` alebo prázdny) — inak spojenie neprebehlo.

---

## Pridanie ďalšieho klienta

Pre každý nový klient (napr. iPhone):

1. Vygeneruj key pair na klientovi
2. Na Mikrotiku pridaj nový peer s ďalšou IP:

```bash
/interface wireguard peers add interface=wg0 \
  allowed-address=10.10.10.3/32 \
  public-key="NOVY_KLIENT_PUBLIC_KEY"
```

3. Na klientovi nastav konfiguráciu s `Address = 10.10.10.3/32`

---

## Troubleshooting

| Problém | Riešenie |
|---------|---------|
| `last-handshake` je prázdny | Skontroluj public keys na oboch stranách, Endpoint a port |
| Handshake OK ale ping nefunguje | Skontroluj firewall pravidlá (poradie!) a AllowedIPs |
| Timeout na pripojenie | Over že port 51820/udp nie je blokovaný ISP |
| Funguje ping na 10.10.10.1 ale nie na 192.168.88.x | Chýba forward pravidlo vo firewalle |
| DNS nefunguje cez VPN | Pridaj DNS firewall pravidlá (port 53 UDP+TCP) a použi `DNS = 10.10.10.1` |
