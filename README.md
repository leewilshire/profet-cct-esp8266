# profet-cct-esp8266

High-side CCT (tunable white) LED controller using an Infineon PROFET 24 V protected switch shield and a WeMos D1 mini (ESP8266). Designed for 24 V constant-voltage LED spots with common negative and separate CW/WW positives, integrated with Home Assistant via ESPHome.

---

## Overview

This project documents a working prototype for driving **24 V CCT LED lighting** using:

- **High-side switching** (no modification to common-negative LED fittings)
- An **Infineon PROFET™ 24 V protected switch shield**
- A **WeMos D1 mini (ESP8266)** running **ESPHome**
- Native **Home Assistant** integration

The repository focuses on *real-world wiring*, *correct channel pairing*, and *common pitfalls* when using the PROFET shield outside a standard Arduino Uno.

---

## Hardware

### Core components

- **Infineon PROFET™ 24 V Protected Switch Shield**  
  (e.g. BTT6030 / BTT6020 family)
  - Product page (Infineon): https://www.infineon.com/evaluation-board/24V-SHIELD-BTT6030
  - Product Manual: https://docs.rs-online.com/404e/A700000009578339.pdf
  - Example distributor listing: https://www.digikey.co.uk/en/products/detail/infineon-technologies/24VSHIELDBTT6030TOBO1/

- **WeMos D1 mini (ESP8266)**  
  Powered via USB during prototyping

- **24 V constant-voltage LED spots**  
  - Common negative (−)
  - Separate **CW+** and **WW+**

- **24 V DC power supply**

### Wemod D1 Mini reference
<img width="600" alt="image" src="https://github.com/user-attachments/assets/8ed207f4-96ae-4d15-a1de-a9271f51a987" />

### PROFET shield reference

<img width="500" alt="image" src="https://github.com/user-attachments/assets/3597c176-852a-4fec-9140-5145ee8f74ba" />

#### Pin Definitions and Functions
- IN0_0: Input to switch channel 0 on PROFETTM+ 24V device no. 0
- IN1_0: Input to switch channel 1 on PROFETTM+ 24V device no. 0
- IN0_1: Input to switch channel 0 on PROFETTM+ 24V device no. 1
- IN1_1: Input to switch channel 1 on PROFETTM+ 24V device no. 1
- IN0_2: Input to switch channel on PROFETTM+ 24V device no. 2
- DEN_0: Turns diagnosis for PROFETTM+ 24V device no. 0 on or off
- DEN_1: Turns diagnosis for PROFETTM+ 24V device no. 1 on or off
- DEN_2: Turns diagnosis for PROFETTM+ 24V device no. 2 on or off
- DSEL_0: Selects if the diagnosis of channel 0 or 1 is muxed to the IS Pin
- DSEL_1: Selects if the diagnosis of channel 0 or 1 is muxed to the IS Pin
- IS_0: Current sense of PROFETTM+ 24V device no. 0
- IS_1: Current sense of PROFETTM+ 24V device no. 1
- IS_2: Current sense of PROFETTM+ 24V device no. 2


### Important PROFET concepts

The shield uses **two dual-channel PROFET devices**. Output labels are *not* simple channel numbers:

| Output label | Meaning |
|-------------|--------|
| 0.0 | Channel 0 of device 0 |
| 1.0 | Channel 1 of device 0 |
| 0.1 | Channel 0 of device 1 |
| 1.1 | Channel 1 of device 1 |

**Correct CCT pairs must stay on the same device:**

✅ Valid CW/WW pairs:
- **0.0 + 1.0**
- **0.1 + 1.1**

❌ Invalid (do not use together):
- 0.0 + 0.1
- 1.0 + 1.1

Each output is controlled by a specific input pin:

| Output | Input pin |
|------|----------|
| 0.0 | IN0_0 |
| 1.0 | IN1_0 |
| 0.1 | IN0_1 |
| 1.1 | IN1_1 |

---

## Wiring summary

### Power

- **24 V PSU +** → Shield **VS** (Screw Terminal)
- **24 V PSU −** → Shield **GND** (Screw Terminal)
- **D1 mini GND** → Shield **GND**
- D1 mini powered via **USB** (or 24v to 5v step down)

### Example lighting groups

| Room | CW output | WW output |
|----|----------|----------|
| Kitchen | 0.1 | 1.1 |
| Lounge | 0.0 | 1.0 |

- Tie all CW+ for a room together
- Tie all WW+ for a room together
- LED negatives remain common to PSU −

### Controller → Shield inputs

| Function | D1 mini pin | Shield input |
|--------|------------|-------------|
| Kitchen CW | D5 (GPIO14) | IN0_1 |
| Kitchen WW | D6 (GPIO12) | IN1_1 |
| Lounge CW | D7 (GPIO13) | IN0_0 |
| Lounge WW | D8 (GPIO15) | IN1_0 |

> Note: Pull-down resistors on input pins are **not required**.  
> The shield is designed for direct MCU drive; external pull-downs may interfere depending on wiring.

---
## Software
### ESPHome background
This project uses **ESPHome** as the firmware layer between the ESP8266 and Home Assistant.

ESPHome allows device behaviour to be defined declaratively in YAML rather than writing and maintaining custom firmware. It handles Wi-Fi connectivity, OTA updates, Home Assistant discovery, and PWM output generation, leaving this project to focus purely on hardware mapping and behaviour.

ESPHome is particularly well suited to this application because:
- PWM outputs can be defined explicitly and mapped cleanly to hardware pins
- Native support exists for **CWWW (tunable white)** light entities
- Integration with Home Assistant is automatic via the ESPHome API
- Firmware can be updated over USB initially and OTA thereafter

This repository assumes basic familiarity with ESPHome but does **not** require prior ESP8266 firmware development experience.

### Installing ESPHome
The recommended way to install and manage ESPHome is via **Home Assistant**:

- ESPHome documentation:  
  https://esphome.io/

- Installing ESPHome in Home Assistant:  
  https://esphome.io/guides/getting_started_hassio.html

Alternatively, ESPHome can be run standalone using the command line:

- ESPHome CLI installation:  
  https://esphome.io/guides/installing_esphome.html

Once ESPHome is installed, the YAML configuration provided in this repository can be pasted directly into a new device definition and flashed to the WeMos D1 mini over USB.

After the first flash, updates can be performed **over-the-air (OTA)**.

### ESPHome Configuration 

- Platform: **ESP8266**
- Board: **WeMos D1 mini**
- PWM frequency: **400 Hz** (works reliably with PROFET inputs)

### ESPHome Device Config Yaml

```yaml
esphome:
  name: kitchen-lounge-spots
  comment: 2 channel CCT lighting using Infineon PROFET shield

esp8266:
  board: d1_mini

wifi:
  ssid: "YOUR_WIFI_SSID"
  password: "YOUR_WIFI_PASSWORD"

captive_portal:

api:
ota:

logger:
  baud_rate: 0

output:
  - platform: esp8266_pwm
    id: kitchen_cw
    pin: D5
    frequency: 400 Hz
  - platform: esp8266_pwm
    id: kitchen_ww
    pin: D6
    frequency: 400 Hz
  - platform: esp8266_pwm
    id: lounge_cw
    pin: D7
    frequency: 400 Hz
  - platform: esp8266_pwm
    id: lounge_ww
    pin: D8
    frequency: 400 Hz

light:
  - platform: cwww
    name: "Kitchen Spots"
    cold_white: kitchen_cw
    warm_white: kitchen_ww
    cold_white_color_temperature: 6500 K
    warm_white_color_temperature: 2700 K

  - platform: cwww
    name: "Lounge Spots"
    cold_white: lounge_cw
    warm_white: lounge_ww
    cold_white_color_temperature: 6500 K
    warm_white_color_temperature: 2700 K
