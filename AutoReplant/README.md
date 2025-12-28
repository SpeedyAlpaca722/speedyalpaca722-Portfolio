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

## Bilder

Legende (Dateinamen in `AutoReplant/images/`):

- ![harvest_before](AutoReplant/images/auto-replant_worldguardflag.png) — Zustand vor der Ernte
- ![harvest_after](AutoReplant/images/harvest_after.png) — Zustand nach der Ernte / Wiederanpflanzung
- ![message_actionbar](AutoReplant/images/harvest.gif)
— Actionbar/Chat-Meldung ("Du hast erfolgreich ...")


---

## Sanitisierte Konfigurationsdateien
- [`sanitized_config.yml`](sanitized_config.yml) — Beispielkonfiguration (keine sensiblen Daten)  
- [`sanitized_messages.yml`](sanitized_messages.yml) — Beispielmeldungen (lokalisierbar)

