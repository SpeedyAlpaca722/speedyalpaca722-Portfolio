# AutoReplant

**Kurzpitch**  
AutoReplant erntet Pflanzen per Rechtsklick und pflanzt sie automatisch wieder ein. Optimiert für Paper 1.21+; optional mit WorldGuard-Flag-Integration zur Gebietskontrolle. Ich veröffentliche keine JARs oder Quellcode öffentlich — JAR auf Anfrage.

## Funktionen
- Rechtsklick-Ernte für Standardpflanzen (Weizen, Karotten, Kartoffeln, Rote Beete)
- Automatisches Wiederanpflanzen nach Ernte
- Anpassbares Wachstumssystem (min-growth-time, growth-probability)
- Konfigurierbare Nachrichten (messages.yml)
- WorldGuard-Flag-basierte Berechtigungen für Regionen

## Kompatibilität & Abhängigkeiten
- **Getestet:** Paper 1.21+  
- **Optional:** WorldGuard (Flag-Integration), WorldEdit (falls benötigt)

## WorldGuard Flag

| Flag | Standard | Beschreibung |
|------|----------|--------------|
| `auto-replant` | `deny` | Erlaubt oder verbietet automatisches Wiederanpflanzen in einer Region |

**Beispiel (WorldGuard):**
```bash
/rg flag <region> auto-replant allow  # Erlauben
/rg flag <region> auto-replant deny   # Verbieten
