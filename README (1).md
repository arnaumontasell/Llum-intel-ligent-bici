# 🚴 Llum Intel·ligent per a Bicicleta

> Llum posterior intel·ligent amb alarma antirobo que s'activa i es desactiva sola quan t'allunyes, detecció de frenada automàtica i control via Bluetooth des del mòbil.

![Estat](https://img.shields.io/badge/estat-en%20desenvolupament-orange)
![Plataforma](https://img.shields.io/badge/hardware-ESP32--C3-blue)
![Llicència](https://img.shields.io/badge/llicència-MIT-green)

---

## 📋 Descripció

Projecte de disseny i fabricació d'una llum intel·ligent per a bicicleta que combina seguretat viària i antirobo en un sol dispositiu compacte.

**Funcionalitats:**
- Llum vermella posterior fixa sempre encesa mentre circules
- Detecció de frenada automàtica via acceleròmetre (MPU6050) → pampallugues
- Sistema d'alarma antirobo: s'activa automàticament quan t'allunyes de la bici i es desactiva al aproparte
- Control via Bluetooth des de l'app web al mòbil
- 1 sol botó físic per encendre/apagar
- Notificació al mòbil quan salta l'alarma

---

## 🏗️ Arquitectura del sistema

```
┌─────────────────────────────────────────────┐
│              ESP32-C3 SuperMini              │
│                                             │
│  MPU6050 ──► Detecció frenada / moviment    │
│  WS2812B ──► LED ring 12 LEDs               │
│  Buzzer  ──► Confirmacions i alarma         │
│  Botó    ──► Encesa / apagada               │
│  BLE     ──► App mòbil (RSSI + control)     │
└─────────────────────────────────────────────┘
              │
              │ Bluetooth Low Energy
              ▼
┌─────────────────────────────────────────────┐
│           App Web (llumbici.html)            │
│                                             │
│  • Alarma On / Alarma Off                   │
│  • Encendre / Apagar llum                   │
│  • Estat en temps real                      │
│  • Indicador de proximitat RSSI             │
│  • Registre d'esdeveniments                 │
└─────────────────────────────────────────────┘
```

---

## ⚡ Màquina d'estats

```
APAGAT ──(botó)──► CIRCULANT ──(frenada)──► FRENANT
                       │                       │
                  (BLE lluny)              (acaba frenada)
                       │                       │
                       ▼                       ▼
                     ARMAT ◄──────────── CIRCULANT
                       │
                  (moviment)
                       │
                       ▼
                ALARMA_ACTIVA ──(BLE prop / botó)──► CIRCULANT
```

---

## 🔧 Hardware

### Components (Fase 1 ~11€)

| Component | Especificació | Preu |
|---|---|---|
| Microcontrolador | ESP32-C3 SuperMini (ESP32-C3FH4) | ~3.5€ |
| Acceleròmetre | MPU6050 SMD 4×4mm | ~1€ |
| LEDs | WS2812B LED ring 12 LEDs | ~2€ |
| Bateria | LiPo 402030 150mAh JST PH 2.0 | ~4€ |
| Buzzer | Buzzer SMD passiu 5V | ~0.5€ |

### Pinatge ESP32-C3 SuperMini

| Funció | Pin |
|---|---|
| WS2812B data | GPIO 8 |
| MPU6050 SDA | GPIO 4 |
| MPU6050 SCL | GPIO 5 |
| Buzzer | GPIO 20 |
| Botó | GPIO 21 |

---

## 📁 Estructura del projecte

```
Llum-intel-ligent-bici/
│
├── llumbici.html          ← App web (obrir amb Bluefy a iPhone)
│
├── wokwi/                 ← Simulació Wokwi (sense hardware)
│   ├── sketch.ino         ← Codi per simular a wokwi.com
│   └── diagram.json       ← Circuit pre-muntat
│
└── hardware_real/         ← Codi per al hardware real
    ├── platformio.ini     ← Configuració PlatformIO
    ├── src/
    │   └── main.cpp       ← Programa principal
    └── include/
        ├── Config.h       ← Pins i constants
        ├── Estat.h        ← Màquina d'estats
        ├── Leds.h         ← Control WS2812B
        ├── Accelerometre.h← MPU6050 + detecció
        ├── Buzzer.h       ← Buzzer no bloquejant
        ├── Proximitat.h   ← RSSI + filtre Kalman
        └── Bluetooth.h    ← BLE GATT Server
```

---

## 🚀 Com utilitzar-ho

### Simulació a Wokwi (sense hardware)

1. Ves a [wokwi.com](https://wokwi.com) → New Project → ESP32
2. Substitueix el `sketch.ino` pel de la carpeta `wokwi/`
3. Substitueix el `diagram.json` pel de la carpeta `wokwi/`
4. Afegeix les llibreries: **FastLED** i **MPU6050**
5. Clica ▶ per simular

**Controls de simulació:**
- Botó verd → Encén/apaga la llum
- Serial Monitor `lluny` + Enter → Simula mòbil lluny (arma alarma)
- Serial Monitor `prop` + Enter → Simula mòbil prop (desarma alarma)
- Serial Monitor `estat` + Enter → Informació del sistema

### Hardware real

1. Instal·la [VS Code](https://code.visualstudio.com) + extensió [PlatformIO](https://platformio.org)
2. Obre la carpeta `hardware_real/`
3. Connecta l'ESP32-C3 per USB
4. Clica **Upload** a PlatformIO

### App mòbil

L'app web funciona a qualsevol mòbil via Bluetooth:

**Android (Chrome):**
1. Obre Chrome → [llumbici.html](https://arnaumontasell.github.io/Llum-intel-ligent-bici/llumbici.html)
2. Clica "Connectar via Bluetooth"

**iPhone:**
1. Descarrega **Bluefy** (App Store, gratuït)
2. Obre la URL amb Bluefy
3. Clica "Connectar via Bluetooth"

---

## 📡 Protocol BLE

### UUIDs

| Característica | UUID | Tipus |
|---|---|---|
| Servei | `bb000001-dead-beef-cafe-000000000000` | — |
| Control | `bb000002-dead-beef-cafe-000000000000` | Write |
| Estat | `bb000003-dead-beef-cafe-000000000000` | Read + Notify |

### Comandes (1 byte)

| Valor | Comanda |
|---|---|
| `0x01` | Llum On |
| `0x02` | Llum Off |
| `0x03` | Alarma On |
| `0x04` | Alarma Off |
| `0x05` | Llegir estat |

### Estats notificats

| Valor | Estat |
|---|---|
| `0x00` | Apagat |
| `0x01` | Circulant |
| `0x02` | Frenant |
| `0x03` | Armat |
| `0x04` | Alarma activa |

---

## 🗺️ Roadmap

- [x] Firmware ESP32 complet
- [x] Simulació Wokwi funcionant
- [x] App web amb Web Bluetooth
- [ ] Proves en hardware real
- [ ] Calibratge llindars frenada/moviment
- [ ] Disseny PCB a KiCad
- [ ] Carcassa 3D (objectiu: 50×25×12mm, ~15g)
- [ ] Fase 2: GPS (NEO-6M) + localització remota
