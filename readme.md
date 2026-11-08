# P1 Meter ESPHome voor Lolin ESP32-C3 & Makki's SMS Shield

Een robuuste en zelfvoorzienende ESPHome-firmware voor de **Lolin ESP32-C3 Mini** in combinatie met het **Makki P1-shield (SMS v1)** om een **Landis+Gyr E360** (of vergelijkbare DSMR 5.0+) slimme meter uit te lezen.

## 📋 Inhoudsopgave

* [🚀 Belangrijke Features](#-belangrijke-features)
* [🛠️ Hardware & Pinout](#️-hardware--pinout)
* [⚙️ ESPHome Configuratie](#️-esphome-configuratie)
* [📊 LED Logica & Kleurenverloop](#-led-logica--kleurenverloop)
* [📥 Installatie & Snelle Start](#-installatie--snelle-start)
* [📚 Aanvullende Documentatie](#-aanvullende-documentatie)

## 🚀 Belangrijke Features

* **DSMR 5.0+ Ondersteuning**: Leest automatisch alle actuele verbruiks-, terugleverings- en tariefregisters uit (Normaal/Piek en Dal).
* **Dynamische Status-LED (WS2812B)**:
  * **Standby (~0W):** Subtiel zacht wit.
  * **Verbruik (Geel naar Rood):** Schaalt op basis van het vermogen tot een maximum van 8000W (ca. 35A op 1 fase).
  * **Teruglevering (Groen naar Blauw):** Schaalt op basis van zonne-opbrengst tot een maximum van 3680W (ca. 16A op 1 fase).
* **Anti-Flipperkast Filter**: Maakt gebruik van een *Exponential Moving Average* filter (`alpha: 0.2`) en een vloeiende transitie van 2 seconden om rustige overgangen te garanderen zonder wilde kleurwisselingen.
* **Lokale Webinterface (Versie 3)**: Volledig inzicht in alle meetwaarden, systeemstatus, IP-adres, Wi-Fi signaalsterkte, Uptime en ESPHome-versie. De status-LED is via de webserver ook handmatig te dimmen of uit te schakelen.
* **OTA (Over-The-Air) Updates**: Eenvoudig draadloos bij te werken na de eerste USB-flash.

## 🛠️ Hardware & Pinout

| Component | Pin / Aansluiting | Beschrijving |
| :--- | :--- | :--- |
| **Lolin ESP32-C3 Mini** | `GPIO8` | Onboard WS2812B RGB LED (aangestuurd via ESP-IDF RMT) |
| **UART RX** | `GPIO20` | Ontvangt data van het Makki P1-shield (inverteert het signaal via de boordtransistor) |
| **UART TX** | `GPIO21` | Niet actief gebruikt voor P1, gereserveerd voor busstructuur |
| **Voeding** | `5V / 3.3V` | Gevoed via het P1-shield of USB-C tijdens programmeren |

*Zie voor het volledige aansluitschema, de BOM (1K en 10K weerstanden) en het Mermaid-diagram de [HARDWARE.md](HARDWARE.md).*

## ⚙️ ESPHome Configuratie

De volledige ESPHome-firmware is te vinden in het bestand [`makkis-sms.yaml`](makkis-sms.yaml). 

Belangrijke configuratiedetails:
* **Framework:** ESP-IDF met `esp32_rmt_led_strip` driver voor de onboard LED op `GPIO8`.
* **DSMR:** Aangesloten op `GPIO20` (UART RX) op 115200 baud.
* **Webserver:** Geconfigureerd met `include_internal: true` voor weergave van netwerk- en systeeminformatie.

## 📊 LED Logica & Kleurenverloop

De onboard WS2812B RGB-LED op GPIO8 geeft direct visuele feedback:

* **Standby Mode (`P < 10W`)**: De LED brandt op 10% helderheid in een zachte witte kleur.
* **Verbruiksmodus (`P > 10W`)**: Helderheid schakelt naar 50%. Kleur verlopen van **Geel/Oranje** (laag verbruik) naar **Rood** (richting de 8000W limiet).
* **Teruglevermodus (`P > 10W`)**: Helderheid schakelt naar 50%. Kleur verlopen van **Groen** (lage zonne-opbrengst) naar **Blauw** (richting de 3680W limiet).

## 📥 Installatie & Snelle Start

1. **Secrets instellen**: Zorg dat `wifi_ssid` en `wifi_password` zijn gedefinieerd in je ESPHome `secrets.yaml`.
2. **Firmware Flashen**: Sluit de ESP32-C3 via USB-C aan en flash het bestand via het ESPHome Dashboard of met het CLI commando:
   ```bash
   esphome run makkis-sms.yaml
   ```
3. **P1-kabel Aansluiten**: Plug de RJ11-kabel in het Makki P1-shield. De LED zal direct zacht wit oplichten.
4. **Home Assistant**: Open Home Assistant; de P1-meter verschijnt automatisch bij de Ontdekte Integraties.

## 📚 Aanvullende Documentatie

* 📖 [**HARDWARE.md**](HARDWARE.md) — Bevat het uitgebreide onderdelenoverzicht (BOM met 1K/10K weerstanden), de stapsgewijze soldeerdocumentatie en het volledige **Mermaid-architectuurdiagram**.
* 📝 [**CHANGELOG.md**](CHANGELOG.md) — Bekijk alle releasedetails, versie-historie en toegevoegde functionaliteiten per versie.