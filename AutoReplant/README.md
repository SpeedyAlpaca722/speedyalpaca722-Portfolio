# AutoReplant

**Kurzpitch**  
AutoReplant erntet Pflanzen per Rechtsklick und pflanzt sie automatisch wieder ein. Optimiert für Paper 1.21+; optional mit WorldGuard-Flag-Integration zur Gebietskontrolle. Ich veröffentliche keine JARs oder Quellcode öffentlich.

## Funktionen
- Rechtsklick-Ernte für Standardpflanzen (Weizen, Karotten, Kartoffeln, Rote Beete)  
- Automatisches Wiederanpflanzen nach Ernte  
- Anpassbares Wachstumssystem (`min-growth-time`, `growth-probability`)  
- Konfigurierbare Meldungen (`messages.yml`)  
- WorldGuard-Flag-basierte Berechtigungen für Regionen

## Kompatibilität & Abhängigkeiten
- **Getestet:** Paper 1.21+  
- **Optional:** WorldGuard (Flag-Integration), WorldEdit (falls benötigt)

## WorldGuard Flag

| Flag | Standard | Beschreibung |
|------|----------|--------------|
| `auto-replant` | `deny` | Erlaubt oder verbietet automatisches Wiederanpflanzen in einer Region |

---

## Demo

**So sieht AutoReplant in Aktion aus.**  
Klicken Sie auf das Bild, um das Demo-Video (YouTube — *unlisted*) zu öffnen.

<p align="center">
  <a href="https://youtu.be/DEIN_VIDEO_CODE" target="_blank" rel="noopener noreferrer">
    <img src="images/demo_thumb.png" alt="AutoReplant Demo" style="max-width:640px; width:100%; height:auto; border:1px solid #ddd;">
  </a>
</p>

**Kurzbeschreibung des Demos:**  
Das Video zeigt den Ablauf „Feld abbauen → Auto-Replant“ einschließlich der Actionbar/Chat-Meldung und des Verhaltens bei gesetzter WorldGuard-Flag (`allow` vs. `deny`). Dauer: ca. XX Sekunden.

---

## Bilder
Legende (Dateinamen in `AutoReplant/images/`):
- `harvest_before.png` — Zustand vor der Ernte  
- `harvest_after.png` — Zustand nach der Ernte / Wiederanpflanzung  
- `message_actionbar.png` — Actionbar/Chat-Meldung (z. B. „Du hast erfolgreich …“)

---

## Sanitisierte Konfigurationsdateien
- [`sanitized_config.yml`](sanitized_config.yml) — Beispielkonfiguration (keine sensiblen Daten)  
- [`sanitized_messages.yml`](sanitized_messages.yml) — Beispielmeldungen (lokalisierbar)

