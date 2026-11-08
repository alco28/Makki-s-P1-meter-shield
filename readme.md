# P1-meter met ESPHome, LOLIN ESP32-C3 en Makki's SMS-shield

Deze ESPHome-configuratie leest de P1-poort uit van een **Landis+Gyr E360** of een vergelijkbare slimme meter die DSMR 5 ondersteunt. De hardware bestaat uit een **LOLIN ESP32-C3 mini** en het **Makki's SMS v1 P1-shield**.

Het P1-shield volgt de **LOLIN/Wemos D1 mini-shieldindeling**. De volgende LOLIN-boards hebben die fysieke indeling, maar vereisen niet allemaal dezelfde firmware:

* **[LOLIN C3 mini](https://docs.wemos.cc/en/latest/c3/c3_mini.html)** — aanbevolen; dit is het board waarvoor deze configuratie is gemaakt.
* **[LOLIN S2 mini](https://docs.wemos.cc/en/latest/s2/s2_mini.html)** — past fysiek op D1 mini-shields, maar controleer en pas de pinnen en LED-configuratie aan.
* **[LOLIN D1 mini](https://docs.wemos.cc/en/latest/d1/d1_mini.html)** en **[D1 mini Lite](https://docs.wemos.cc/en/latest/d1/d1_mini_lite.html)** — hebben dezelfde shield-indeling, maar gebruiken een ESP8266/ESP8285. Deze configuratie is voor ESP32 en werkt daar niet ongewijzigd op.
* **D1 mini Pro** — heeft een langere printplaat en is daarom geen directe mechanische vervanger voor de standaard D1 mini-indeling.

De huidige configuratie is direct bedoeld voor de C3 mini. Gebruik je een ander ESP32-board, stel dan onder `esp32:` het juiste `board` in en controleer of de shieldpinnen overeenkomen met de UART-instellingen: GPIO20 voor RX en GPIO21 voor TX. Pas zo nodig de `uart:`-sectie aan. De configuratie stuurt ook een WS2812B-led op GPIO8 aan. Ontbreekt die led op jouw board, verwijder dan in `makkis-sms.yaml` de volledige `light:`-sectie, het script `update_status_led` en de `script.execute`-acties bij de twee vermogenssensoren.


## 📋 Inhoudsopgave

* [🚀 Belangrijkste functies](#-belangrijkste-functies)
* [🛠️ Hardware & Pinout](#️-hardware--pinout)
* [⚙️ ESPHome Configuratie](#️-esphome-configuratie)
* [📊 Ledkleuren](#-ledkleuren)
* [📥 Installatie & Snelle Start](#-installatie--snelle-start)
* [📚 Aanvullende Documentatie](#-aanvullende-documentatie)

## 🚀 Belangrijkste functies

* **DSMR-metingen**: Leest meteridentificatie en DSMR-versie, verbruiks- en terugleveringsstanden per tarief, actueel vermogen, spanning en stroom op fase L1 uit.
* **Status-led (WS2812B)**: Geeft het nettovermogen weer: zacht wit rond nul, geel/oranje tot rood bij verbruik en groen tot blauw bij teruglevering.
* **Smoothing vermogensweergave**: Past op beide actuele vermogenssensoren een exponentieel voortschrijdend gemiddelde toe (`alpha: 0.2`) en laat de led vloeiend overgaan in 2 seconden.
* **Lokale webinterface**: Toont meetwaarden en systeeminformatie, waaronder IP-adres, wifi-signaalsterkte, uptime en ESPHome-versie. De led kan ook via de webinterface worden bediend.
* **OTA-updates**: Na de eerste installatie via USB kan de firmware via wifi worden bijgewerkt.

### Voorbeeld van de webinterface

<p align="center">
  <img src="webinterface-crop.png" alt="Voorbeeld van de ESPHome-webinterface met P1-meetwaarden en de status-led" width="560" style="max-width: 100%; height: auto;">
</p>
<p align="center"><em>Voorbeeld van de lokale ESPHome-webinterface.</em></p>

## 🛠️ Hardware & Pinout

### Hardware

<p align="center">
  <img src="pcb-lolinc3.png" alt="Makki's SMS P1-shield en LOLIN C3 mini naast elkaar" width="560" style="max-width: 100%; height: auto;">
</p>
<p align="center"><em>Makki's SMS P1-shield naast de LOLIN C3 mini.</em></p>

De opstelling bestaat uit een **LOLIN ESP32-C3 mini** en het **Makki's SMS v1 P1-shield**. De meter verstuurt seriële telegrammen via de P1-poort. Het shield past het datasignaal aan voor de UART-ingang van de ESP32, die de telegrammen uitleest en de ingestelde meetwaarden via ESPHome beschikbaar maakt. De onboard WS2812B-led van de C3 mini is aangesloten op GPIO8.

De P1-poort gebruikt een modulaire **6-posities connector (6P6C)**, meestal RJ12 genoemd. Een RJ11-stekker past vaak in deze aansluiting, maar een RJ11-kabel kan minder contacten hebben. Controleer of de kabel geschikt is voor jouw meter en shield.

| Component | Pin / Aansluiting | Beschrijving |
| :--- | :--- | :--- |
| **LOLIN ESP32-C3 mini** | `GPIO8` | Onboard WS2812B-statusled |
| **UART RX** | `GPIO20` | Ontvangt via het shield de P1-telegrammen |
| **UART TX** | `GPIO21` | Ingesteld in de UART-configuratie; voor deze uitlezing niet actief gebruikt |
| **Voeding** | USB-C / shield | USB-C voor programmeren; in gebruik volgens de shield-opbouw |

Zie [hardware.md](hardware.md) voor de opbouw, onderdelenlijst (BOM) en het signaalstroomschema.

## ⚙️ ESPHome Configuratie

De volledige ESPHome-configuratie staat in [`makkis-sms.yaml`](makkis-sms.yaml).

Belangrijke instellingen:
* **Board:** `lolin_c3_mini` met ESP32-C3.
* **P1-UART:** GPIO20 (RX), GPIO21 (TX) en 115200 baud.
* **Status-led:** WS2812 op GPIO8; de led toont het nettovermogen, berekend uit verbruik minus teruglevering.
* **Webserver:** HTTP op poort 80, versie 3. `include_internal: true` zorgt ervoor dat ook interne entiteiten in de webinterface worden getoond.
* **Wifi:** SSID en wachtwoord komen uit ESPHome-secrets `wifi_ssid` en `wifi_password`.

### DSMR en telegramfrequentie

DSMR beschrijft onder andere de gegevens en communicatie van Nederlandse slimme meters. Een meter stuurt P1-telegrammen met onder meer meterstanden en actuele waarden. DSMR 4 verstuurt doorgaans ongeveer elke 10 seconden een telegram; DSMR 5 doorgaans elke seconde. Daardoor worden waarden op DSMR 5 vaker bijgewerkt. De frequentie wordt door de meter bepaald: deze configuratie ontvangt en verwerkt telegrammen, maar vraagt de meter niet actief om extra telegrammen. De UART staat ingesteld op 115200 baud, passend bij de beoogde DSMR 5-meter; controleer de handleiding van je meter als de gegevens niet binnenkomen.

## 📊 Ledkleuren

De WS2812B-led op GPIO8 geeft het nettovermogen weer (afname minus teruglevering):

* **Rond nul (minder dan 10 W):** zacht wit op 10% helderheid.
* **Verbruik (meer dan 10 W):** geel/oranje naar rood; de kleurschaal loopt op tot 8000 W.
* **Teruglevering (meer dan 10 W):** groen naar blauw; de kleurschaal loopt op tot 3680 W.

## 📥 Installatie & Snelle Start

1. **Wifi-gegevens instellen:** Definieer `wifi_ssid` en `wifi_password` in `secrets.yaml` binnen je ESPHome-configuratiemap.
2. **Firmware flashen:** Sluit de LOLIN C3 mini via USB-C aan en flash de configuratie via het ESPHome-dashboard of met de opdracht:
   ```bash
   esphome run makkis-sms.yaml
   ```
3. **P1-kabel aansluiten:** Verbind het Makki's SMS-shield met de P1-poort van de meter. De aansluiting is 6P6C (vaak RJ12 genoemd); een RJ11-stekker past vaak, maar controleer of de kabel geschikt is. Na het opstarten licht de status-led rond nul zacht wit op.
4. **Home Assistant verbinden:** Voeg het gevonden ESPHome-apparaat toe via *Instellingen → Apparaten en diensten → Integraties*.

## 📚 Aanvullende Documentatie

* [hardware.md](hardware.md) — onderdelenlijst, opbouwstappen en signaalstroomschema.
* [changelog.md](changelog.md) — wijzigingen per versie.

## Licentie

Dit project is beschikbaar onder de **GNU General Public License v3.0**. Zie [LICENSE](LICENSE) voor de volledige licentievoorwaarden.