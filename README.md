# 🔊 ESP32-S3-Box-3 - Custom Voice Assistant Firmware

Dies ist eine erweiterte ESPHome-Konfiguration für die **ESP32-S3-Box-3**, die auf der offiziellen ESPHome-Vorlage für sprachgesteuerte Assistenten basiert.

Diese Firmware verwandelt die S3-Box-3 in einen autarken, lokalen und zuverlässigen **Voice Assistant** für Home Assistant, ergänzt um erweiterte Sensorik und Komfortfunktionen.

## 🛠️ Installations- und Anpassungsanleitung

### 1\. Voraussetzungen

  * Ein installierter **Home Assistant Server** mit der **ESPHome Add-on** oder einer separaten ESPHome-Installation.
  * Eine aktive **Home Assistant Voice Assistant Pipeline**.

### 2\. Erweiterte Hardware-Anforderungen (Wichtig\!)

Die folgenden Funktionen **setzen das optionale SENSOR DOCK voraus**, da sie die dort verbauten Komponenten nutzen:

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

Die Konfiguration baut auf der offiziellen Vorlage auf und erweitert sie um essenzielle System- und Messdaten.

| Komponente | Sensor/Entität | Beschreibung |
| :--- | :--- | :--- |
| **System/Netzwerk** | **WiFi Signal**, **Uptime**, **WiFi SSID/BSSID**, **IP Address** | Diese Sensoren sind direkt auf der Hauptplatine implementiert und **funktionieren ohne das SENSOR DOCK**. Sie dienen der Überwachung der Netzwerkstabilität und des Gerätezustands. |
| **Benutzer-Eingabe** | **Home Button** (`gt911`) | Ein Touch-Flächensensor, der als programmierbarer "Home"- oder Zurück-Button auf dem Display dient. |
| **Präsenzerkennung** | **`binary_sensor.presence_detect`** | Erfasst Anwesenheit über den **AT581X-Radar-Sensor des Docks**. Steuert die Nachlaufzeit (`delayed_off`). |
| **Mute-Logik** | **`switch.mute_when_absent`** | Automatisches Stummschalten des Mikrofons bei Abwesenheit zum Schutz der Privatsphäre. |
| **Raumklima (AHT20)** | **`sensor.temperature`** / **`sensor.humidity`** | Temperatur- und Feuchtigkeitsmessung. Enthält eine **statische Kalibrierung** ($\mathbf{offset: -4.24 \text{ °C}}$) zur Korrektur der internen Erwärmung des Boards. |
| **Batterieanzeige** | **`sensor.battery_voltage` / `sensor.battery_percent`** | Zeigt Spannung und Ladezustand des internen Akkus an. |

## 🔗 Externe Abhängigkeiten

  * **Home Assistant API:** Für die Hauptkommunikation.
  * **WLAN-Verbindung:** Für den Zugriff auf das Home Assistant Backend.
  * **`micro_wake_word`:** Für die lokale Wake-Word-Erkennung.
