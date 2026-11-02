<div align="center">

# Heizlogik mit Override

### Manuell freundliche Heizungssteuerung für Home Assistant

Die Heiz-Saison wird automatisch berechnet, kann aber jederzeit bewusst per Override
aktiviert werden. Jeder Raum heizt nach Komfort- oder ECO-Temperatur – ohne erzwungene
Zeitpläne und ohne verpflichtende Anwesenheitslogik.

<p>
  <img src="https://img.shields.io/badge/Version-1.1.0-1E88C7?style=for-the-badge" alt="Version">
  <img src="https://img.shields.io/badge/Typ-Automation-1E88C7?style=for-the-badge" alt="Automation">
  <img src="https://img.shields.io/badge/Home%20Assistant-2024.10%2B-1E88C7?style=for-the-badge&logo=homeassistant&logoColor=white" alt="Home Assistant 2024.10+">
  <img src="https://img.shields.io/badge/License-MIT-1E88C7?style=for-the-badge" alt="License">
</p>

[![Blueprint importieren](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2FxXJimnyCricketXx%2Fha-blueprints%2Fblob%2Fmain%2Fheizlogik-mit-override%2Fheizlogik_mit_override.yaml)

**[Übersicht](#-übersicht)** ·
**[Funktionen](#-funktionen)** ·
**[So funktioniert es](#-so-funktioniert-es)** ·
**[Einrichtung](#-einrichtung)** ·
**[Zeitpläne](#-zeitpläne)** ·
**[Roadmap](#-roadmap)**

</div>

<br/><br/>

## <img src="https://api.iconify.design/lucide/book-open.svg?color=%231E88C7" width="20"> Übersicht

**Heizlogik mit Override** ist ein Automations-Blueprint, der pro Raum eingesetzt wird
und das Thermostat zwischen Komfort- und ECO-Temperatur umschaltet.

Er ist für Haushalte gedacht, in denen **manuelle Kontrolle wichtig** ist und die
verstehen wollen, **was ihre Heizung warum tut**. Die Temperatur wird nur bei klaren
Ereignissen gesetzt – etwa Saisonbeginn, Urlaub, Zeitplanwechsel oder Fenster.
Wer zwischendurch am Thermostat dreht, behält seine Einstellung bis zum nächsten Ereignis.

**Gut geeignet für:**

- Home-Assistant-Nutzer, die **keine aggressive Heizungsautomatik** möchten
- Smart-Home-Einsteiger, die eine **nachvollziehbare Lösung** suchen

**Nicht primär gedacht für:**

- vollständig automatische Anwesenheitssteuerung
- sehr komplexe Wochenzeitpläne mit vielen täglichen Umschaltpunkten
- Systeme, die permanent Temperaturen nachregeln sollen

<br/><br/>

## <img src="https://api.iconify.design/lucide/sparkles.svg?color=%231E88C7" width="20"> Funktionen

- **Heiz-Saison mit Override** — Die Saison wird automatisch anhand der Monate berechnet und lässt sich jederzeit manuell einschalten. Außerhalb der Saison wird das Thermostat ausgeschaltet.
- **Komfort und ECO pro Raum** — Beide Temperaturen sind eigene Helfer und lassen sich direkt im Dashboard ändern. Eine Änderung wirkt sofort, wenn der jeweilige Modus gerade aktiv ist.
- **Urlaub** — Ein globaler Schalter versetzt alle Räume in den ECO-Betrieb.
- **Optionaler Zeitplan** — Ein `schedule`-Helfer schaltet zwischen Komfort (*Ein*) und ECO (*Aus*).
- **Optionaler Fensterkontakt** — Nach einer einstellbaren Verzögerung wird auf ECO geschaltet (nicht aus). Kurzes Lüften wird ignoriert, beim Schließen geht es sofort zurück.
- **Manuelle Änderungen bleiben** — Kein ständiges Nachregeln. Die Automation greift nur bei Ereignissen ein.

<br/><br/>

## <img src="https://api.iconify.design/lucide/lightbulb.svg?color=%231E88C7" width="20"> So funktioniert es

Die Steuerung besteht aus **zwei klar getrennten Ebenen**.

### 1. Globale Heiz-Saison

Die Heiz-Saison gibt vor, **ob grundsätzlich geheizt werden soll**.
Sie wird von einer eigenen Automation berechnet (z. B. Oktober bis April) und kann über
einen Override jederzeit aktiviert werden. Der Saison-Schalter selbst wird **nicht direkt**
vom Benutzer geschaltet.

### 2. Raumbezogene Heizlogik

Jeder Raum bekommt eine eigene Automation aus diesem Blueprint. Sie bestimmt den Zielmodus:

| **Situation** | **Modus** |
| :--- | :---: |
| Heiz-Saison aus | Thermostat **aus** |
| Urlaub ein | ECO |
| Fenster länger als die Verzögerung offen | ECO |
| Zeitplan aktiviert und *Aus* | ECO |
| alle anderen Fälle | Komfort |

Ohne Zeitplan ist Komfort also der Standardbetrieb.

<br/><br/>

## <img src="https://api.iconify.design/lucide/rocket.svg?color=%231E88C7" width="20"> Einrichtung

### Voraussetzungen

- Home Assistant **2024.10** oder neuer
- Ein Thermostat als `climate`-Entität, das den Modus `heat` unterstützt
- Grundkenntnisse im Umgang mit Helfern und Automationen

Helfer werden unter **Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen** angelegt.

### Schritt 1: Globale Helfer anlegen

Diese Helfer werden **einmal zentral** benötigt:

| **Name** | **Typ** | **Entity-ID** | **Zweck** |
| :--- | :--- | :--- | :--- |
| Heiz-Saison | Schalter | `input_boolean.heiz_saison` | Gibt vor, ob geheizt wird. Wird durch die Automation aus Schritt 2 gesetzt. |
| Heiz-Saison Override | Schalter | `input_boolean.heiz_saison_override` | Manueller Eingriff: aktiviert die Saison unabhängig vom Monat. |
| Urlaub | Schalter | `input_boolean.urlaub` | Längere Abwesenheit: alle Räume wechseln auf ECO. |

### Schritt 2: Heiz-Saison automatisch berechnen

Die globale Heiz-Saison wird über eine eigene, bewusst einfache Automation gesetzt.
Sie ist **kein Blueprint**, sondern eine Vorlage zum Kopieren:

[`examples/heiz_saison_berechnen.yaml`](examples/heiz_saison_berechnen.yaml)

1. **Einstellungen → Automationen & Szenen → Automation erstellen**
2. Rechts oben im Drei-Punkte-Menü **In YAML bearbeiten** wählen
3. Inhalt der Datei einfügen, bei Bedarf die Monate anpassen und speichern

Die Automation prüft täglich, bei jedem Neustart und bei jeder Änderung des Overrides,
ob die Heiz-Saison aktiv sein soll.

### Schritt 3: Raumbezogene Helfer anlegen

Für **jeden Raum** zwei Helfer vom Typ **Zahl** (`input_number`):

| **Helfer** | **Beispiel** | **Einstellungen** |
| :--- | :--- | :--- |
| Komfort-Temperatur | `input_number.schlafzimmer_komfort` | Einheit °C, Min. `4`, Max. `30`, Schrittweite `0,5` |
| ECO-Temperatur | `input_number.schlafzimmer_eco` | Einheit °C, Min. `4`, Max. `30`, Schrittweite `0,5` |

Die Anzeige kann *Schieberegler* oder *Eingabe* sein. Ein realistischer Temperaturbereich
macht den Schieberegler im Dashboard besser nutzbar.

### Schritt 4: Optionale Helfer

**Fensterkontakte:** Ein einzelner Fensterkontakt oder eine **Gruppe** (`binary_sensor`).

> [!TIP]
> Hat ein Raum mehrere Fenster oder Fensterflügel, aus allen Kontakten eine Gruppe
> erstellen und diese im Blueprint auswählen. Die Gruppe gilt als offen, sobald ein
> Fenster offen ist.

**Zeitplan:** Ein Helfer vom Typ **Zeitplan** (`schedule`), z. B. `schedule.schlafzimmer_heizen`.
Siehe [Zeitpläne](#-zeitpläne).

### Schritt 5: Blueprint importieren und Automation erstellen

1. Blueprint über den Badge oben importieren oder die URL unter
   **Einstellungen → Automationen & Szenen → Blueprints → Blueprint importieren** einfügen:

   ```text
   https://github.com/xXJimnyCricketXx/ha-blueprints/blob/main/heizlogik-mit-override/heizlogik_mit_override.yaml
   ```

2. Aus dem Blueprint eine neue Automation erstellen und auswählen:

   | **Eingabe** | **Pflicht** | **Beschreibung** |
   | :--- | :---: | :--- |
   | Heiz-Saison | ✅ | `input_boolean.heiz_saison` |
   | Urlaub | ✅ | `input_boolean.urlaub` |
   | Thermostat | ✅ | `climate`-Entität des Raums |
   | Komfort-Temperatur | ✅ | `input_number` des Raums |
   | ECO-Temperatur | ✅ | `input_number` des Raums |
   | Zeitplan berücksichtigen? | ❌ | Standard: aus |
   | Zeitplan | ❌ | `schedule`-Helfer |
   | Fenster berücksichtigen? | ❌ | Standard: aus |
   | Fensterkontakt / Gruppe | ❌ | `binary_sensor` |
   | Fenster-Verzögerung | ❌ | Standard: `00:10:00` |

3. Der Automation einen **Bereich** zuweisen (empfohlen).

Der Blueprint steht danach dauerhaft zur Verfügung und wird für jeden weiteren Raum
wiederverwendet.

<br/><br/>

## <img src="https://api.iconify.design/lucide/calendar-clock.svg?color=%231E88C7" width="20"> Zeitpläne

Zeitpläne sind **nicht erforderlich**, können aber sinnvoll sein, z. B. für ECO tagsüber,
wenn niemand zu Hause ist, oder zum Vorheizen vor der Rückkehr.

Ein Home-Assistant-Zeitplan steuert **keine Temperaturen**. Er kennt nur zwei Zustände:

- **Ein** → Komfort-Temperatur
- **Aus** → ECO-Temperatur

Ein Zeitplan mit den Zeitfenstern **06:00–09:00** und **17:00–23:00** führt also dazu,
dass morgens und abends Komfort gilt und außerhalb dieser Zeiten ECO. Welche Temperaturen
das konkret sind, legen die Temperatur-Helfer des Raums fest.

<br/><br/>

## <img src="https://api.iconify.design/lucide/folder-tree.svg?color=%231E88C7" width="20"> Dateien

| **Datei** | **Inhalt** |
| :--- | :--- |
| [`heizlogik_mit_override.yaml`](heizlogik_mit_override.yaml) | Raumbezogener Heizungs-Blueprint |
| [`examples/heiz_saison_berechnen.yaml`](examples/heiz_saison_berechnen.yaml) | Beispiel-Automation zur Berechnung der Heiz-Saison |

<br/><br/>

## <img src="https://api.iconify.design/lucide/map.svg?color=%231E88C7" width="20"> Roadmap

| **Version** | **Änderungen** | **Status** |
| :---: | :--- | :---: |
| **1.1.0** | <ul><li>Fix: Zeitplan und Fensterkontakt lösen jetzt zuverlässig aus</li><li>Fix: Heizung wird am Saisonende korrekt ausgeschaltet</li><li>Fix: Änderung der ECO-Temperatur greift nicht mehr im Komfort-Modus</li><li>Kurzes Lüften überschreibt keine manuellen Änderungen mehr, Rückkehr beim Schließen ohne Verzögerung</li><li>Fensterstatus wird auch bei Zeitplanwechseln und Temperaturänderungen berücksichtigt</li><li>Robuster bei nicht verfügbaren Helfern und beim Neustart</li><li>Aktuelle Home-Assistant-Syntax, mindestens 2024.10</li><li>Beispiel-Automation berechnet die Saison auch nach einem Neustart</li></ul> | <img src="https://img.shields.io/badge/Released-1E88C7?style=for-the-badge" alt="Released"> |
| **1.0.0** | <ul><li>Heiz-Saison mit Override</li><li>Komfort/ECO pro Raum, Urlaub</li><li>Optionaler Zeitplan und Fensterkontakt</li></ul> | <img src="https://img.shields.io/badge/Released-1E88C7?style=for-the-badge" alt="Released"> |

<br/><br/>

---

<br>

<div align="center">

**[← Zurück zur Übersicht](../README.md)**

</div>
