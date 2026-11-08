# Changelog

Alle belangrijke wijzigingen aan dit project worden in dit bestand gedocumenteerd. Het formaat is gebaseerd op [Keep a Changelog](https://keepachangelog.com/nl/1.0.0/) en dit project volgt [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-11-08

### Toegevoegd
- **DSMR 5.0+ Ondersteuning**: Volledige uitlezing van verbruik, teruglevering (Normaal/Piek en Dal), spanning en stroom voor o.a. de Landis+Gyr E360.
- **Dynamische LED-status (WS2812B)**: Onboard RGB-LED op GPIO8 met een `exponential_moving_average` filter (`alpha: 0.2`) en 2-seconden transitie tegen flickering.
  - Verbruik (Geel naar Rood, geschaald tot 8kW).
  - Teruglevering (Groen naar Blauw, geschaald tot 3.68kW).
  - Standby-modus (10% helderheid zacht wit bij <10W).
- **Lokale Webserver (v3)**: Ingebouwde webinterface inclusief handmatige LED-aansturing/dimmer.
- **Systeemmonitoring**: Interne sensoren toegevoegd aan de webinterface voor `uptime`, `wifi_signal`, `ip_address` en de `version` van ESPHome.
- **Documentatie**:
  - `README.md` met inhoudsopgave, features, ESPHome YAML-configuratie en snelstart-gids.
  - `HARDWARE.md` met volledige BOM (inclusief 1K/10K weerstanden, BC547 transistor), 5-stappen soldeerinstructies en gevalideerd Mermaid-architectuurdiagram.