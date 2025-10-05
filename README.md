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


-------
# 🧩 ESP32-S3-Box-3 - Erweiterte Sensorik (Add-on Konfiguration)

Diese YAML-Datei enthält die **zusätzlichen Konfigurationsblöcke**, die zur Aktivierung der erweiterten Sensoren des **ESP32-S3-Box-3 Sensor Docks** in der offiziellen Voice Assistant Firmware benötigt werden.

Diese Datei dient dazu, die Haupt-Firmware zu ergänzen und das **Radar-Modul (AT581X)**, den **Raumklimasensor (AHT20)** und die **Batterieüberwachung** freizuschalten.

## ⚠️ Wichtige Voraussetzung: Sensor Dock

Die gesamte Logik dieses Codes setzt voraus, dass das **optionale SENSOR DOCK** angeschlossen ist.

## ⚙️ Detailierte Funktionsübersicht und Integration

Dieser Code-Auszug ergänzt die Basis-Firmware um folgende Hauptfunktionalitäten:

### 1. I²C-Bus Konfiguration

Da das Sensor Dock **zwei separate I²C-Busse** nutzt, werden beide konfiguriert.

| Bus-ID | SDA-Pin | SCL-Pin | Komponenten | Zweck |
| :---: | :---: | :---: | :---: | :--- |
| **`bus_a`** | GPIO08 | GPIO18 | Touchscreen, **Radar (AT581X)** | Bus der Hauptplatine, nun auch für Radar genutzt. |
| **`bus_b`** | GPIO41 | GPIO40 | **AHT20-Sensor** | Separater Bus für den Klimasensor. |

### 2. Präsenzerkennung und Mute-Logik (Radar)

Diese Logik nutzt den AT581X-Radar zur Anwesenheitserkennung und zur Steuerung der Mikrofonstummschaltung.

* **`at581x` / `switch.enable_radar`**: Aktiviert und steuert das Radar-Modul auf I²C-Bus A.
* **`binary_sensor.presence_detect` (GPIO21)**: Der Ausgang des Radars, der die Anwesenheit meldet.
    * **Filter:** Verwendet einen **`delayed_off`**-Filter, dessen Zeit von der `number.radar_delayed_off` Entität gesteuert wird (Nachlaufzeit).
* **`switch.mute_when_absent`**: Steuert die Funktion: Wenn dieser Schalter in Home Assistant auf `on` ist und keine Präsenz mehr erkannt wird (`on_release`), schaltet das Gerät das Mikrofon stumm (`switch.turn_on: mute`) und die Bildschirmbeleuchtung aus (`light.turn_off: led`).

### 3. Raumklima-Überwachung (AHT20)

* **`sensor.temperature` / `sensor.humidity`**: Liest den **AHT20**-Sensor über den separaten **`bus_b`** aus.
* **Kalibrierung:** Die Temperatur (`id: s3temp`) enthält einen **`offset: -4.24`**, um die Board-Erwärmung des ESP32 auszugleichen und eine präzisere Raumtemperatur zu liefern.

### 4. Batterie- und System-Überwachung

* **`sensor.adc` (GPIO10)**: Misst die Batteriespannung und leitet daraus den prozentualen Ladezustand (`sensor.battery_percent`) ab.
* **System-Sensoren:** Es wurden standardmäßige System-Sensoren wie **WiFi-Signal**, **Uptime**, **IP Address**, **SSID** und **BSSID** hinzugefügt, um die Stabilität und den Netzwerkstatus des Geräts besser überwachen zu können.

### 5. Touchscreen-Integration

* **`touchscreen` (GT911)**: Konfiguriert den Touch-Controller auf **`bus_a`**.
* **`binary_sensor.home_button`**: Definiert den programmierbaren Touch-Bereich des Displays als Home-Button.

---

## 🚀 Nutzung

Um diese Funktionen zu aktivieren, fügen Sie die Komponenten dieser Datei in Ihre Haupt-ESPHome-YAML-Konfiguration der ESP32-S3-Box-3 ein und kompilieren Sie die Firmware neu. Die neuen Entitäten werden automatisch in Home Assistant zur Verfügung stehen.
