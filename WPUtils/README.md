```
===============================================================================
PLANNING.txt — ModerationCenter (Komplette, finale Fassung)
Version: 1.1
Projekt: ModerationCenter
Zweck: Vollständige Spezifikation für ein KI-gestütztes Chat-Moderations- und
       Sanktionensystem-Plugin (Paper / Spigot / Bukkit, Minecraft 1.21+)
Lizenz-Empfehlung: MIT oder Apache-2.0
Wichtig: KEIN Anti-Cheat. KEIN Web-Dashboard. Alle Admin/Mod-Interaktionen In-Game,
         per Config und optional Discord/Webhook. Lokal ausführbar (ONNX),
         keine externen API-Kosten erforderlich.
Autor: TRySpeedy (Discord: speedyalpaca722)
===============================================================================

INHALTSVERZEICHNIS
1. Ziele & Anforderungen
2. Design-Prinzipien
3. Vollständige Feature-Liste (inkl. Mehrsprachigkeit)
4. Architektur & Komponenten
5. Mehrsprachigkeit (detailliert, praktikabel)
6. KI-Design & Modelldetails
7. Regeln / Config-Formate (vollständig, kommentiert DE/EN)
8. Sanktionen / Infraction-System / Eskalation / Reputation
9. GUI, Commands & Permissions
10. Datenmodell / SQL-Schema
11. Integrationen & Erweiterungen
12. Betrieb, Skalierung, Fallbacks & Performance
13. Sicherheit, Datenschutz & DSGVO
14. Monitoring, Reporting & Analytics (in-game)
15. Teststrategie, CI/CD & Release-Checklist
16. Roadmap & Meilensteine
17. Implementierungs-Tasks (konkret)
18. Pseudocode & technische Hinweise (DE/EN kommentiert)
19. Anhang: plugin.yml, Beispiel-Configs, Hinweise (DE/EN)
===============================================================================

1) ZIELE & ANFORDERUNGEN
--------------------------------------------------------------------------------
Kurz:
- Ziel: Ein lokal ausführbares, performantes und erweiterbares Plugin zur
  Chat-Moderation und Sanktionierung, das semantische Regeln versteht, robuste
  Mehrsprachigkeit unterstützt, einfach konfigurierbar ist und mittlere bis große
  Server skaliert.
- Nicht-Ziele: Anti-Cheat, Web-Dashboard (nur In-Game + optionale Discord-Notifikation).

Anforderungen:
- 100 % lokale Ausführbarkeit möglich (ONNX-Modelle).
- Semantische Regeln (Rule-Embeddings) in natürlicher Sprache.
- Modernes Infraction-System mit Points, Decay, Probation, Appeals und Two-Mod-Approval.
- Sehr gute Admin-UX (In-Game GUIs, Rule-Editor, Moderation-Queue).
- Multilingualer Betrieb mit per-rule Language-Support, per-language thresholds und FP-Learning.

-------------------------------------------------------------------------------
2) DESIGN-PRINZIPIEN
--------------------------------------------------------------------------------
- Privacy-first / Offline-first: Standardmäßig minimale Speicherung, Evidence nur
  bei Vorfällen. Lokale Modelle bevorzugt.
- Simple first / Advanced next: Out-of-the-box nutzbar, Advanced Mode für Profis.
- Keine Main-Thread-Blocking: Alle schweren Aufgaben (Inference, DB IO) asynchron.
- Transparenz: Begründungen (Scores, Rule-Matches, Token-Highlights) für alle automatischen Aktionen.
- Erweiterbarkeit & Testbarkeit: Java API, Events, Unit-/Integration-Tests, Flyway/Liquibase Migrationen.

-------------------------------------------------------------------------------
3) VOLLSTÄNDIGE FEATURE-LISTE (inkl. Mehrsprachigkeit)
--------------------------------------------------------------------------------
A) Kernfeatures
- Echtzeit Chatmoderation (AsyncPlayerChatEvent).
- Hybrid-Moderation: Toxicity-Model (ONNX) + Rule-Embeddings + Regex/Blacklist + Kontext + Reputation.
- Aktionen: warn, tempmute, kick, tempban, permban, shadowmute, soft-ban.
- Infraction Points, Decay, Probation, Eskalation, automatische Strafen.

B) Admin- & Moderation-Tools
- In-Game Rule Editor (Inventory/Anvil), Rule-Templates (Best Practice).
- Moderation Queue GUI mit Evidence Viewer & One-Click-Aktionen.
- Appeal-Workflow (in-game), Appeal History.
- Moderator-Training (simulierte Fälle), Moderator-Statistiken & Reputation.
- Moderator-Makros & Vorlagen (mit PlaceholderAPI).

C) Mehrsprachige Unterstützung (zentral)
- Message-level Language Detection.
- Multilingual Toxicity-Model + Multilingual Embeddings (ONNX).
- Per-Rule `languages`-Feld, per-language thresholds (overrides).
- Fallbacks: Regex/Blacklist; optional Offline-Übersetzung (lokal) als letzter Schritt.
- FP-Collector & AutoTune (per language).

D) Operations & Erweiterungen
- Batching + bounded queue + fallback mode.
- Optional: local inference microservice (localhost HTTP) für große Instanzen.
- Model HotSwap & Safe Rollback.
- Pluggable Storage: SQLite default; MySQL/Postgres via HikariCP.
- Mass-Defense (raid detection), scheduled rules, cross-server ban sync (opt-in), PII Redaction, optional DB-Verschlüsselung.

-------------------------------------------------------------------------------
4) ARCHITEKTUR & KOMPONENTEN
--------------------------------------------------------------------------------
Hauptkomponenten:
- MainPlugin: lifecycle, config loader, model manager, scheduler, command registration.
- Event Layer: Async Chat Listener, optional Staff/Command hooks.
- Preprocessor: normalizeText, removeColors, leetNormalize, homoglyphNormalize, computeHash.
- ModelInferenceService: ONNX Runtime (Toxicity + Embeddings), batching engine, async API.
- RuleEngine: load rules.yml → precompute embeddings → match by language & scope.
- DecisionEngine: combine toxicityScore, ruleSim, blacklistHit, contextScore, reputation → decide or suggest.
- InfractionManager: persist infractions, points, decay, scheduling, appeals, audit log.
- Storage Layer: DAO pattern (SQLite / MySQL / Postgres), migrations.
- GUI Layer: Inventory GUIs, Anvil input, pagination, quick actions.
- API Layer: ModerationService Java API & Bukkit events.
- ModelManager: models/manifest.json, hotSwap, warmup, health checks.
- Monitoring: in-game stats and optional Prometheus exporter.

Message Flow (Kurz):
1. Message -> Preprocessor -> Language Detection.
2. Enqueue -> Batched inference (toxicity + embedding).
3. RuleEngine match -> DecisionEngine computes hybridScore.
4. Action -> InfractionManager persists & schedules -> notify mods.

-------------------------------------------------------------------------------
5) MEHRSPRACHIGKEIT (Detailliert, praktikabel)
--------------------------------------------------------------------------------
Ziel: sauber, transparent, skalierbar – Mehrsprachigkeit ist integraler Bestandteil.

Strategie:
- Language Detection pro Nachricht (z. B. fastText/langdetect).
- Multilingual Models: XLM-R / multilingual MiniLM (quantized ONNX) für Toxicity und Embeddings.
- Per-Rule Language Metadata: `languages` + `similarity_threshold.overrides`.
- Fallbacks:
  - `regex` fallback wenn Sprache unsupported oder low-confidence.
  - Optional `translate_to` (offline translator) → erhöhte Latenz; nutzbar nur wenn nötig.
- Operational Tools:
  - FP-Collector (per language).
  - AutoTune Assistant per language -> Vorschläge für thresholds.
  - Per-language evaluation sets (Precision/Recall).
- Normalizer: Leet mapping, unicode homoglyph normalization, transliteration helpers.
- Resource Management: Multilingual models benötigen mehr RAM/CPU => quantisieren, batchen, evtl. per-language lightweight models für Performance.

Beispielregel mit Sprach-Overrides:
```yaml
# DE: Regel mit sprachspezifischen Thresholds
# EN: Rule with per-language thresholds
- id: racist_insult
  text: "Beleidigungen aufgrund von Herkunft, Rasse oder Religion"
  languages: ["de", "en", "fr"]
  similarity_threshold:
    default: 0.85
    overrides:
      de: 0.82
      en: 0.86
  actions:
    - tempban: "7d"
````

Implementierungs-Schritte für Multi-Language:

1. Language Detector (fastText/langdetect) integrieren.
2. Default: multilingual toxicity model + multilingual embeddings.
3. Add per-rule language metadata and per-language thresholds.
4. Add fallback: regex + optional translation.
5. Build FP pipeline & AutoTune per language.

---

6. KI-DESIGN & MODELDETAILS

---

Empfohlene Modelle:

* Toxicity: quantized XLM-R / multilingual RoBERTa (ONNX 8-bit).
* Embeddings: paraphrase-multilingual-MiniLM (ONNX).
* Optional: sentiment/topic classifier, offline translator (Opus-MT/Marian).

Tokenisierung:

* Biete HuggingFace-exportierte Tokenizer-Artefakte oder ein Java-Wrapper; alternativ Python-Helper.

Inferenz-Pattern:

* Batch-window: 20–50 ms (konfigurierbar).
* ExecutorService mit `inference_threads`.
* Warmup-Requests beim Start (z. B. 5).
* Cache decisions by messageHash (TTL 300s).
* Quantisierung: 8-bit empfohlen; verifiziere Accuracy.

Explainability:

* Token-level contribution heuristics (perturbation/token-drop) → Top-K token/phrases in Evidence-View.

---

7. REGELN / CONFIG-FORMATE (vollständig, kommentiert DE/EN)

---

Dateien:

* config.yml (Simple Mode) — kommentiert DE/EN
* rules.yml — Regeln in natürlicher Sprache (DE/EN)
* advanced.yml — Performance/DB Tuning
* profiles/ — lenient/normal/strict
* scheduled_rules.yml — zeitgesteuerte Regeln

Beispiel: config.yml (DE/EN kommentiert)

```yaml
# DE: Hauptkonfiguration (Simple Mode)
# EN: Main configuration (Simple Mode)
server:
  platform: PAPER           # DE: Empfohlen / EN: Recommended
  api_version: "1.21"

easy_mode: true             # DE: importiert Rule-Templates bei erstem Start / EN: imports rule templates on first run

model:
  toxicity_model: "models/xlm_roberta_toxicity.onnx"       # DE: Toxicity-Model Pfad / EN: path to toxicity model
  embedding_model: "models/paraphrase_multilingual_mini.onnx"  # DE/EN: embeddings
  quantized: true            # DE: quantisierte Modelle verwenden / EN: use quantized models
  inference_threads: 4
  batch_window_ms: 50
  warmup_requests: 5

moderation:
  toxicity_weight: 0.6
  rule_weight: 0.35
  blacklist_weight: 0.05
  thresholds:
    warn: 0.45
    mute: 0.7
    ban: 0.95

infraction:
  warn_points: 1
  mute_points: 4
  tempban_points: 12
  decay_days_per_point: 7
  probation_days: 14

logging:
  messages_log_enabled: false   # DE: Default aus aus Datenschutzgründen / EN: default off for privacy
  retention_days: 90

integrations:
  discord_webhook_url: ""       # DE/EN optional
  vault_integration: true

language_detection:
  enabled: true
  min_confidence: 0.6
```

Beispiel: rules.yml (DE/EN kommentiert)

```yaml
# DE: Regeln mit Metadaten
# EN: Rules with metadata
version: 1
rules:
  - id: direct_threat
    text: "Direkte Androhung von körperlicher Gewalt oder Tötung"
    severity: 10
    similarity_threshold:
      default: 0.90
      overrides:
        de: 0.88
        en: 0.90
    languages: ["de","en","fr","*"]  # DE: '*' = alle Sprachen / EN: '*' = all languages
    actions:
      - tempban: "14d"
      - notify_mods: true
```

Hinweis: Kommentare in DE/EN in Konfiguration helfen internationalen Admins.

---

8. SANKTIONEN / INFRACTION-SYSTEM / ESKALATION / REPUTATION

---

* **Infraction-Record**: id, player_uuid, type, points, reason, moderator, evidence, created_at, expires_at, appeal_status.
* **Points & Escalation**: konfigurierbare Punkte pro Aktion; escalation map (z. B. >=8 -> tempban 7d).
* **Decay & Probation**: Punkteverfall (z. B. 1 Punkt / 7 Tage); Probation nach schweren Strafen.
* **Two-Mod Approval**: optional für permabans oder hohe Severity.
* **Reputation**: float Score pro Spieler, beeinflusst DecisionEngine (forgiveness / stricter handling).
* **Appeal-Workflow**: player command `/moderationcenter appeal <infraction-id> <reason>` → appears in mod queue.

Restorative Flow:

* Bei minor offenses: auto-apology prompt und points reduction on compliance.

---

9. GUI, COMMANDS & PERMISSIONS (kompakt)

---

Wichtige Commands (DE/EN kommentiert):

```
# DE: Hauptbefehle
# EN: Main commands
/moderationcenter help
/moderationcenter test <message>
/moderationcenter warn <player> <reason>
/moderationcenter mute <player> <duration> <reason>
/moderationcenter unmute <player>
/moderationcenter kick <player> <reason>
/moderationcenter ban <player> <duration|perm> <reason>
/moderationcenter unban <player>
/moderationcenter history <player>
/moderationcenter queue
/moderationcenter rules
/moderationcenter appeal <infraction-id> <reason>
/moderationcenter appeals
/moderationcenter autotune
/moderationcenter profile set <profileName>
/moderationcenter model hotswap <modelName>
/moderationcenter simulate <player/message>
/moderationcenter report daily|weekly
/moderationcenter lockdown enable|disable
```

Permissions (vollständig):

* moderationcenter.*
* moderationcenter.admin.*
* moderationcenter.mod.warn
* moderationcenter.mod.mute
* moderationcenter.mod.ban
* moderationcenter.mod.queue
* moderationcenter.rules.edit
* moderationcenter.user.appeal
* moderationcenter.mod.training
* moderationcenter.mod.approve_permanent_ban
* moderationcenter.reputation.manage
* moderationcenter.sync.manage

GUI-Layouts:

* Moderation Queue: Paginated list (player head, score, matched rules), Quick Actions.
* Rule Editor: list/add/edit/delete, threshold slider, test input.
* Player History: infractions & evidence, undo/reduce.
* Mod Training: interactive scenarios & scoring.

---

10. DATENMODELL / SQL-SCHEMA

---

Wesentliche Tabellen (kompakt):

Players:

```sql
CREATE TABLE IF NOT EXISTS players (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  uuid VARCHAR(36) NOT NULL,
  name VARCHAR(48) NOT NULL,
  points INTEGER DEFAULT 0,
  reputation REAL DEFAULT 0,
  last_active DATETIME,
  UNIQUE(uuid)
);
```

Infractions:

```sql
CREATE TABLE IF NOT EXISTS infractions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  player_uuid VARCHAR(36) NOT NULL,
  type VARCHAR(32) NOT NULL,
  points INTEGER NOT NULL,
  reason TEXT,
  moderator VARCHAR(48),
  evidence TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  expires_at DATETIME NULL,
  appeal_status VARCHAR(20) DEFAULT 'none'
);
```

Weitere Tabellen: rules, messages_log, appeals, reputation, mod_activity, scheduled_rules, audit_log.

Index-Empfehlungen: player_uuid, created_at, rule_id, mod_uuid.

---

11. INTEGRATIONEN & ERWEITERUNGEN

---

* Vault / LuckPerms: role based policies & temp roles for mutes.
* WorldGuard: region scopes.
* PlaceholderAPI: placeholders support.
* Discord: webhook/optional bot for notifications & appeal-bridge (secure token).
* Bungee/Waterfall: optional cross-server infractions & ban sync (opt-in).
* Flyway/Liquibase: DB migrations.
* Optional: local inference microservice with health checks.

---

12. BETRIEB, SKALIERUNG, FALLBACKS & PERFORMANCE

---

* Batching (20–50 ms), bounded queue, warmup, cache TTL.
* Fallback to regex/blacklist when overloaded.
* Hardware sizing:

  * Medium servers: 4 cores, 8 GB RAM.
  * Large servers: 8+ cores, 16+ GB RAM or dedicated inference server.
* HA: central DB for multiple instances; inference per-instance or central microservice.

---

13. SICHERHEIT, DATENSCHUTZ & DSGVO

---

* Default minimal logging; messages_log disabled.
* Evidence retention configurable (default 90 days).
* GDPR: export/delete commands (/moderationcenter export <player>, /moderationcenter deleteinfractions <player>).
* PII detection & redaction; optional encryption for evidence fields.
* Audit logs for moderator actions; secure token storage for sync.

---

14. MONITORING, REPORTING & ANALYTICS (IN-GAME)

---

* /moderationcenter stats -> queue length, avg inference latency, infractions/hour.
* /moderationcenter report daily -> top offenders, heatmap, category distribution.
* Scheduled reports via Discord optional.

---

15. TESTSTRATEGIE, CI/CD & RELEASE-CHECKLIST

---

* Unit tests: RuleEngine, DecisionEngine, InfractionManager.
* Integration: Embedded Paper tests.
* Load tests: big spikes & steady loads.
* CI: GitHub Actions build/test/package.
* Release artifacts: moderationcenter-<ver>.jar, config.yml, rules.yml, db_schema.sql, MODELS_README.md.

---

16. ROADMAP & MEILENSTEINE

---

Phase 0: Bootstrap (skeleton, ChatListener, SQLite).
Phase 1: ONNX integration & RuleEngine.
Phase 2: GUI & InfractionManager.
Phase 3: Batching, MySQL, Model HotSwap.
Phase 4: Mass Defense, AutoTune, cross-server sync.
Phase 5: Polish & release.

---

17. IMPLEMENTIERUNGS-TASKS (KONKRET)

---

Phase 0:

* Gradle init, MainPlugin skeleton, folders, config loader.
* ChatListener (Async), Preprocessor stub, SQLite DAO; regex fallback.

Phase 1:

* ONNX Runtime Java integration, ToxicityService & EmbeddingService.
* RuleEngine with multilingual support, DecisionEngine, InfractionManager.

Phase 2:

* GUI: moderation queue, rule editor, player history.
* Appeals & Evidence snapshotting.

Phase 3:

* Batching & caching, fallback logic.
* MySQL/Postgres support (HikariCP), Model HotSwap, Reputation & Mod Activity.

Phase 4:

* Mass defense, cross-server sync, AutoTune & FP tools.

Phase 5:

* Tests, docs, packaging & release.

---

18. PSEUDOCODE & TECHNISCHE HINWEISE (DE/EN)

---

Java ONNX Inference (DE/EN):

```java
// DE: ONNX Inferenzskizze (Java) - Toxicity-Modell
// EN: ONNX inference sketch (Java) - toxicity model
OrtEnvironment env = OrtEnvironment.getEnvironment();
OrtSession.SessionOptions opts = new OrtSession.SessionOptions();
opts.setIntraOpNumThreads(config.model.inference_threads);
OrtSession session = env.createSession("models/xlm_roberta_toxicity.onnx", opts);

public CompletableFuture<Map<String, Float>> evaluateToxicityAsync(String text) {
  return CompletableFuture.supplyAsync(() -> {
    long[] inputIds = tokenizer.encode(text); // DE: Tokenizer verwenden / EN: use tokenizer
    OnnxTensor inputTensor = OnnxTensor.createTensor(env, new long[][]{inputIds});
    Map<String, OnnxTensor> inputs = Map.of("input_ids", inputTensor);
    try (OrtSession.Result result = session.run(inputs)) {
      float[] logits = ((float[][]) result.get(0).getValue())[0];
      Map<String, Float> scores = logitsToProbabilities(logits);
      return scores;
    }
  }, inferenceExecutor);
}
```

Batching Sketch:

```
# DE: Batching-Mechanismus
# EN: Batching mechanism
# 1) LinkedBlockingQueue<InferenceRequest>
# 2) Scheduler every batch_window_ms: drain up to maxBatchSize
# 3) Run batched inference, map results to futures
# 4) If queue > fallback_threshold -> regex fallback
```

Decision Engine:

```java
double hybrid = config.moderation.toxicityWeight * toxScore
              + config.moderation.ruleWeight * maxRuleSim
              + config.moderation.blacklistWeight * blacklistHit
              - reputationAdjustment;

if (hybrid >= config.moderation.thresholds.ban) {
  applyBan(player, reason);
} else if (hybrid >= config.moderation.thresholds.mute) {
  applyMute(player, duration, reason);
} else if (hybrid >= config.moderation.thresholds.warn) {
  applyWarn(player, reason);
} else {
  allowMessage();
}
```

---

19. ANHANG: plugin.yml, Beispiel-Configs, Hinweise (DE/EN)

---

plugin.yml:

```yaml
name: ModerationCenter
main: de.paparuntz420.moderationcenter.core.MainPlugin
version: "1.0.0"
api-version: "1.21"
authors:
  - "TRySpeedy (Discord: speedyalpaca722)"
commands:
  moderationcenter:
    description: ModerationCenter Hauptbefehl
    usage: /moderationcenter help
    aliases: [mc, modcenter]
permissions:
  moderationcenter.*:
    description: Vollzugriff auf ModerationCenter Befehle
    default: op
```

Model-Hinweis:

* DE: Lege ONNX-Modelle in `plugins/ModerationCenter/models/` oder nutze `models/manifest.json`.
* EN: Place ONNX models into `plugins/ModerationCenter/models/` or use `models/manifest.json`.

---

## LETZTE HINWEISE & EMPFEHLUNGEN

* Start with quantized multilingual models (XLM-R + multilingual MiniLM).
* Collect FP/TP per language and use AutoTune to refine thresholds.
* Default minimal logging; enable messages_log only for debugging/beta.
* Balance performance & accuracy: quantize + batch; optional local microservice for heavy load.
* Provide README, ADMIN.md, DEV.md, MODELS_README.md and CONTRIBUTING.md.

===============================================================================
ENDE DER KOMPLETTEN PLANUNG (ModerationCenter)
==============================================

```

