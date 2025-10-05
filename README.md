# 🔊 ESP32-S3-Box-3 - Custom Voice Assistant Firmware

Dies ist eine erweiterte ESPHome-Konfiguration für die **ESP32-S3-Box-3**, die auf der offiziellen ESPHome-Vorlage für sprachgesteuerte Assistenten basiert.

Diese Firmware verwandelt die S3-Box-3 in einen autarken, lokalen und zuverlässigen **Voice Assistant** für Home Assistant, ergänzt um erweiterte Sensorik und Komfortfunktionen.

## 🛠️ Installations- und Anpassungsanleitung

### 1\. Voraussetzungen

  * Ein installierter **Home Assistant Server** mit der **ESPHome Add-on** oder einer separaten ESPHome-Installation.
  * Eine aktive **Home Assistant Voice Assistant Pipeline**.

### 2\. Erweiterte Hardware-Anforderungen (Wichtig\!)

Die folgenden Zusatzfunktionen dieser Konfiguration setzen voraus, dass das optionale **SENSOR DOCK** (die Basisplatine, die der Box beiliegt) angeschlossen ist, da es den **AT581X-Radar-Sensor** und die Anschlüsse für den **AHT20-Sensor** bereitstellt:

  * **Radar-Präsenzerkennung** (`at581x`)
  * **Raumklima-Überwachung** (`aht10` / AHT20)
  * **Batteriespannungsmessung** (`adc` / Batterieanschluss am Dock)

### 3\. Konfigurationsdatei

Die Konfiguration basiert auf der Datei `esp32-s3-box-3.yaml`.

**Wichtig:** Vor dem Hochladen müssen Sie die `secrets.yaml` konfigurieren.

```yaml
# Beispiel für secrets.yaml
wifi_ssid: "Ihr WLAN Name"
wifi_password: "Ihr WLAN Passwort"
api_encryption_key: "Ein_langer_zufälliger_Schlüssel" 
```

### 4\. Hochladen

Fügen Sie die YAML-Datei in Ihr ESPHome Dashboard ein und flashen Sie die Firmware auf Ihre ESP32-S3-Box-3.

-----

## ⚙️ Detailierte Funktionsübersicht und Erweiterungen

Die Konfiguration baut auf der offiziellen Vorlage auf und erweitert sie um Messungen und Steuerungslogik.

| Komponente | Beschreibung | Anpassungen |
| :--- | :--- | :--- |
| **Präsenzerkennung** | Erfasst Anwesenheit über den integrierten **AT581X-Radar-Sensor des Docks** (`GPIO21`). | **`number.presence_duration`**: Steuert die Nachlaufzeit (`delayed_off`) des Präsenzsensors, bevor Abwesenheit gemeldet wird. |
| **Mute-Logik** | Schaltet das Mikrofon (`switch.mute`) automatisch **ON** und die Beleuchtung **OFF**, wenn keine Anwesenheit erkannt wird (Abwesenheits-Mute). | **`switch.mute_when_absent`**: Template-Schalter zur Deaktivierung dieser Automatisierung über Home Assistant. |
| **Raumklima (AHT20)** | Implementiert den AHT20-Temperatur- und Feuchtigkeitssensor (über I²C-Bus B). | **`offset: -4.24`**: Statische Kalibrierung von **$-4.24 \text{ °C}$** zur Korrektur der Board-Erwärmung. |
| **Batterieanzeige** | Misst den Ladezustand über ADC (`GPIO10`). | **`sensor.battery_voltage` / `sensor.battery_percent`**: Zeigt Spannung und Ladezustand des internen Akkus an. |

## 🔗 Externe Abhängigkeiten

  * **Home Assistant API:** Für die Hauptkommunikation.
  * **WLAN-Verbindung:** Für den Zugriff auf das Home Assistant Backend.
  * **`micro_wake_word`:** Für die lokale Wake-Word-Erkennung.
