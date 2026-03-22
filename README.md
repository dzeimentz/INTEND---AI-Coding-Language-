INTEND — Intent-Driven Programming Language

> **Eine KI-optimierte Hybridsprache für Human–AI Collaboration**
> **An AI-optimized hybrid language for Human–AI Collaboration**

[![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://python.org)
[![Spec](https://img.shields.io/badge/Spec-v1.6.2-0F2A44.svg)](docs/spec/)
[![Status](https://img.shields.io/badge/Status-Phase%200-lightgrey.svg)]()
[![CI](https://img.shields.io/badge/CI-GitHub%20Actions-green.svg)](.github/workflows/ci.yml)

---

## 🇩🇪 Deutsch | 🇬🇧 English

- [Deutsch](#deutsch)
- [English](#english)

---

<a name="deutsch"></a>

# 🇩🇪 Deutsch

## Was ist Intend?

**Intend** (Dateiendung `.itd`) ist eine deklarative, KI-optimierte Programmiersprache, die konsequent
die Ebene der *Absicht* (Was soll erreicht werden?) von der Ebene der *Implementierung* (Wie wird es
berechnet?) trennt. Eingebettete Python-Blöcke laufen in einer sicherheitsisolierten Sandbox.

Intend wurde entwickelt, damit **KI-Systeme und menschliche Entwickler gemeinsam** stabile, lesbare
und fehlerresistente Programme erstellen — mit Python als eingebetteter Ausführungsschicht.

> **Kerngedanke:** Intend ersetzt Python nicht — es strukturiert den Rahmen.
> Python liefert die Rechenleistung. Die Kombination macht beide Ebenen stabiler.

---

## Warum Intend?

Moderne LLMs generieren in klassischen Sprachen fehleranfälligen Code — nicht wegen schlechter
Modelle, sondern wegen struktureller Eigenschaften der Sprachen:

| Problem in Python / C++ | Lösung in Intend |
| --- | --- |
| Syntaktische Ambiguität (`*` als Pointer, Multiplikation oder Dereferenzierung) | Eindeutige Verb-Nomen-Keywords, keine Symbolüberlagerung |
| Fehlende Kontextpersistenz (Konfiguration vs. Laufzustand ununterscheidbar) | Vier explizite Scope-Typen: `Set-Fixed`, `Set-Config`, `Set-Local`, `Set-Out` |
| Off-by-One und Index-Fehler in `for`-Schleifen | `Run-Each` — kein manueller Index, automatisches Error-Handling |
| Kein eingebautes Fehler-Handling | `On-Error`-Pfad ist Pflicht bei Actions und Python-Blöcken |
| Crash bei `None`-Zugriff | Null-Safety: `?.` und `??` — kein NullPointerError möglich |
| Vermischung von Logik und Implementierung | Intend-Schicht (Was) + `>>>` Python-Hook (Wie) — strukturell getrennt |

---

## Schnellstart

### Installation

```bash
pip install intend-lang
```

### Erstes Programm

```yaml
# hello.itd
# intend: v1.6.2

Set-Fixed  "Begruessung": "Hallo von Intend!"

New-Task "Hauptprogramm":
  Intent: "Gibt eine Begrüßung auf der Konsole aus."
  Steps:
    - Run-Action: Log
      message: Set-Fixed.Begruessung
      level: info
```

```bash
intend run hello.itd
```

### KI-generierten Code validieren (mit JSON-Feedback)

```bash
intend check --json ki_generiert.itd
```

### Compile-Modus (kein Interpreter-Overhead)

```bash
intend compile script.itd    # erzeugt script.py
python script.py
```

---

## Keyword-System — Verb-Nomen-Prinzip

Alle Keywords folgen dem Schema `Verb-Nomen` mit einem kontrollierten Vokabular von
10 Verben (`Set`, `New`, `Run`, `Check`, `Find`, `Make`, `Open`, `Close`, `Read`, `Stop`).
Wer `Run` kennt, versteht `Run-Task`, `Run-Job`, `Run-Each`, `Run-Every`, `Run-All` sofort.

### Variablen-Scope-System

```yaml
Set-Fixed  "API_URL":    "https://api.example.com"       # Konstante — nicht veränderbar
Set-Config "Umgebung":   Env("APP_ENV", Refresh: 1_hour) # Konfiguration — Env-Variable, hot-reload
Set-Local  "Zaehler":    0                               # Task-lokal — nach Task-Ende freigegeben
Set-Out    "Ergebnis":   []                              # Explizite Übergabe zwischen Tasks
```

```yaml
# v1.6: Set-Persistent — State zwischen Run-Every-Zyklen
Set-Persistent "LetzterLauf": None   Default: 0
```

```yaml
# v1.6: Explizite Mutation (kein stilles Überschreiben)
Update Set-Local "Zaehler": Zaehler + 1
```

### Null-Safety (v1.6)

```yaml
# ?. — Safe-Access: gibt None statt Crash
Set-Local "Name":   Artikel?.name    ?? "Unbekannt"
Set-Local "Preis":  Artikel?.preis   ?? 0.0

# Safe-Chain über mehrere Ebenen
Set-Local "Stadt":  Adresse?.ort?.name ?? "Keine Stadt"
```

### Benutzerdefinierte Typen (v1.5+)

```yaml
Define-Type "Artikel":
  id:       Integer   Required: True
  name:     Text      Required: True
  menge:    Integer   Default: 0
  preis?:   Float                       # v1.6: optionales Feld
  verlauf?: List[Integer]               # v1.6: generische typisierte Liste
  status:   Text      Values: ["aktiv", "gesperrt", "auslauf"]
```

### Schleifen ohne Index-Fehler

```yaml
# Sequenzielle Iteration — automatisches try/except pro Element
Run-Each "Artikel" In Set-Local.Artikelliste:
  - Run-Action: Log
    message: Artikel?.name ?? "Unbekannt"
    level: info
  # Einzelfehler brechen die Schleife nicht ab
```

```yaml
# v1.6: Mit Zähler
Run-Each "Artikel" In Set-Local.Liste With-Index "i":
  - Run-Action: Log
    message: "Artikel {i}: {Artikel?.name}"
    level: debug
```

```yaml
# Zustandsschleife — kein Endlosloop möglich
Run-Loop Until Set-Local.IstLeer Max: 1000:
  - Run-Task "BestandPruefen"
```

### Zeitgesteuerte Ausführung

```yaml
Run-Every "1 hour":
  Steps:
    - Run-Task "BerichtErstellen"
  On-Error:
    - Run-Action: Log
      message: "Scheduler-Fehler aufgetreten"
      level: error
```

### Python-Integration — drei Formen

```yaml
# Form 1 — einzeilig
- Set-Local "Mittelwert": >>>  import statistics; statistics.mean(Set-Local.Werte)

# Form 2 — mehrzeiliger Block
>>>
import numpy as np
arr = np.array(Set-Local.Messwerte)
result = arr[arr > Set-Config.Schwellenwert].tolist()
<<<
Set-Local "KritischeWerte": result

# Form 3 — benannter Block (empfohlen)
>>> als "Vorhersage":
  import numpy as np
  verlauf = Set-Local.Artikel.verlauf or []
  tage = float(np.polyfit(range(len(verlauf)), verlauf, 1)[0]) if verlauf else 999.0
<<<
Set-Local "TagesBedarf": Vorhersage.result
```

> Python-Blöcke laufen in einer **Drei-Ebenen-Sandbox**:
> statischer AST-Scan → RestrictedPython → Prozess-Isolation mit CPU/RAM-Limits.
> Kein Netzwerkzugriff, kein Schreibzugriff auf das Host-Dateisystem.

### Muster-Matching

```yaml
Pick-Case Set-Local.Status:
  Case "aktiv":
    - Run-Task "Verarbeiten"
  Case "gesperrt":
    - Run-Action: Log
      message: "Artikel gesperrt"
      level: warn
  Other:
    - Stop-Now
      error: "UnbekannterStatus"
      message: "Status nicht erkannt"
```

### Async und Parallelausführung (v1.2+)

```yaml
New-Job "DatenLaden":
  Steps:
    - Run-Action: HTTP_Get
      url: Set-Config.API_URL

Run-All:
  - Run-Job "DatenLaden"
  - Run-Job "CacheAktualisieren"
```

### Robustheit — Retry, Timeout, Circuit-Breaker (v1.4+)

```yaml
New-Task "ExternerAufruf":
  Retry:    3
  Timeout:  30_seconds
  On-Error: Fallback
  Steps:
    - Run-Action: HTTP_Get
      url: Set-Config.ServiceURL
```

### Ressourcen-Management

```yaml
Open-Lock "datenbankverbindung":
  - Run-Task "DatenLesen"
  - Run-Task "DatenSchreiben"
Close-Lock "datenbankverbindung"
# Lock wird auch bei Fehler automatisch freigegeben
```

### IntendHub — Snippet-Bibliothek (v1.5+)

```yaml
# Snippet aus dem Hub laden
>>> from hub: "numpy/linreg-simple" version: "1.2.0"
Set-Local "Vorhersage": result
```

```bash
intend hub freeze        # intend.lock generieren (Produktionssicherheit)
intend hub audit         # Sicherheits-Scan aller Snippets
intend hub update        # Snippets auf neue Versionen prüfen
```

### Plugin-Actions

```yaml
Load-Plugin "intend-slack-action"

New-Task "Benachrichtigung":
  Steps:
    - Run-Action: Notify_Slack
      webhook_url: Set-Config.Slack_Webhook
      message: "Bericht fertig"
      channel: "#ops"
```

---

## Fehlertypen

| Fehlertyp | Zeitpunkt | Auslöser |
| --- | --- | --- |
| `FixedViolation` | Compile | Schreibversuch auf `Set-Fixed`-Variable |
| `TypeMismatch` | Compile | Typkonflikt bei Zuweisung oder Parameter |
| `ScopeViolation` | Compile | Zugriff auf nicht deklarierte Variable |
| `SecurityViolation` | Compile | Verbotenes Muster im Python-Block (`__class__`, `eval`, …) |
| `ValidationError` | Compile | `Check-That`-Bedingung schlägt fehl |
| `NullAccessError` | Compile | Direktzugriff auf möglicherweise `None`-Feld ohne `?.` |
| `ActionFailed` | Runtime | Action liefert Fehler — `On-Error`-Pfad wird aktiviert |
| `IterationLimit` | Runtime | `Run-Loop` überschreitet `Max:`-Limit |
| `PythonHookError` | Runtime | Exception im `>>>` Block — Prozess-Isolation verhindert Absturz |
| `ResourceLimit` | Runtime | Python-Block überschreitet CPU-Zeit oder Speicher-Limit |
| `TimeoutError` | Runtime | Task überschreitet konfiguriertes `Timeout:` |
| `CircuitOpen` | Runtime | Circuit-Breaker hat nach zu vielen Fehlern geöffnet |

---

## CLI-Referenz (v1.6.2 — vollständig)

### Kernbefehle

```bash
intend run    [--env E] [--role R] [--user U] <datei.itd>   # Ausführen
intend check  [--json]                        <datei.itd>   # Validieren — KI-Gate
intend compile                                <datei.itd>   # Zu .py kompilieren
intend migrate --from X --to Y               <datei.itd>   # Syntax-Migration
intend test   [testfile.itd]                               # Conformance-Tests
```

### KI-Befehle (v1.5+)

```bash
intend explain       [--task T]  <datei.itd>    # Natürlichsprachige Zusammenfassung
intend verify        [--task T]  <datei.itd>    # Semantische Intent-Verifikation via LLM
intend suggest       [--task T]  <datei.itd>    # Hub-Snippet-Empfehlung
intend export-schema [--format json|md|ebnf]    # Maschinenlesbare Spec für LLMs
```

### Neue Befehle v1.6

```bash
intend prompt   [--file F] [--append A]         # Natürlichsprache → .itd generieren
intend fix      [--preview] [--safe-only]        # Auto-Korrektur von check --json Fehlern
intend refactor --task T --ask 'msg'             # LLM-gesteuertes Refactoring
intend test-gen [--task T]                       # Automatische Test-Generierung
intend repl                                      # Interaktiver REPL-Modus
intend watch    [--env E] [--task T]             # Auto-Restart bei Dateiänderung
intend fmt      [--check] [*.itd]               # Automatische Formatierung
```

### Analyse und Debugging

```bash
intend bench    [--runs N] [--task T]            # Laufzeit-Benchmark
intend profile  [--task T]                       # Bottleneck-Analyse
intend viz      [--format svg|mermaid|dot]       # Task-Flow visualisieren
intend run      --debug --step                   # Schrittweise Ausführung
intend run      --verbose                        # Generierten Python-Code anzeigen
```

### Hub-Verwaltung

```bash
intend hub freeze                                # intend.lock generieren
intend hub update [snippet-id]                   # Snippets aktualisieren
intend hub search <query>                        # Hub-Snippets suchen
intend hub audit                                 # Sicherheits-Scan aller Snippets
```

---

## Vollständiges Beispielprogramm

```yaml
# intend: v1.6.2
# Lager-Monitor — alle v1.6 Features

Use-File "common/typen.itd"

Set-Fixed  "MinBestand":  10
Set-Fixed  "MwSt":        19 / 100
Set-Config "DB_URL":      Env("DATABASE_URL", Refresh: 1_hour, On-Change: Reconnect)
Set-Config "Email":       Env("OPS_EMAIL")

New-Task "Bestand_Pruefen":
  Intent: "Prüft Lagerbestand und meldet Artikel unter Mindestbestand per E-Mail."
  Parameters:
    - lager: List[Artikel]   Required: True
  Steps:
    - Find-All In Set-Local.lager
        Where Artikel?.menge ?? 0 < Set-Fixed.MinBestand
      As "kritisch"

    - If Set-Local.kritisch IS NOT Empty:
        - Run-Action: Notify_Email
          to:      Set-Config.Email
          subject: "Kritischer Lagerbestand"
          body:    Set-Local.kritisch

Run-Every "6 hours":
  Steps:
    - Run-Task "Bestand_Pruefen"
      lager: Set-Out.AktuellerBestand
  On-Error:
    - Run-Action: Log
      message: "Scheduler-Fehler"
      level:   error
```

---

## Projektstruktur

```
intend-lang/
├── src/intend/
│   ├── parser/          # Stufe 1: lark-Parser + AST-Knoten
│   ├── typechecker/     # Stufe 2: Typ-, Scope- und Null-Safety-Prüfung
│   ├── codegen/         # Stufe 3: Python-Code-Generierung (Jinja2)
│   ├── runtime/         # Stufe 4: Ausführung + RestrictedPython-Sandbox
│   ├── actions/         # Standard-Action-Bibliothek
│   ├── hub/             # IntendHub-Client + intend.lock
│   └── cli/             # 25 CLI-Befehle (v1.6.2)
├── packages/
│   └── vscode-intend/   # VS Code Extension (TypeScript + pygls)
├── tests/
│   ├── conformance/     # 200+ Conformance-Tests
│   └── property/        # Hypothesis Property-Tests
├── examples/            # Vollständige .itd-Beispielprogramme
├── docs/spec/           # Sprachspezifikation v1.6.2
├── LICENSE              # Apache License 2.0
├── NOTICE               # Drittanbieter-Komponenten
└── intend.lock          # Hub-Snapshot (nach intend hub freeze)
```

---

## Umsetzungsplan

| Phase | Zeitraum | Inhalt |
| --- | --- | --- |
| **Phase 0** — Vorbereitung | Q2 2026 | Grammatik, Repository, Lizenz, 10 Beispiele, LLM-System-Prompt, Keyword-Freeze |
| **Phase 1** — Interpreter-Kern | Q3 2026 | Parser, TypeChecker inkl. Null-Safety, CodeGen, Sandbox, `intend run/check` |
| **Phase 2** — Bibliothek & Qualität | Q4 2026 | Plugin-System, Hub-Client, Compile-Modus, 200+ Tests, Hypothesis |
| **Phase 3** — Tooling | Q1 2027 | VS Code Extension, `intend migrate`, SemVer, Docker-Image, `intend fmt/watch` |
| **Phase 4** — Pilot & Adoption | Q2 2027 | Piloteinsatz, KI-Benchmark, `intend prompt`, Release-Entscheidung |

---

## Lizenz

Dieses Projekt steht unter der **Apache License 2.0**.
Siehe [LICENSE](LICENSE) und [NOTICE](NOTICE) für Details.

Plugin-Actions dürfen unter eigenen Lizenzen — einschließlich proprietärer Lizenzen — veröffentlicht werden.

---

## Mitmachen

Beiträge sind willkommen. Bitte lies zuerst [CONTRIBUTING.md](CONTRIBUTING.md) (verfügbar in Phase 1).
Für Fragen und Diskussionen nutze die [GitHub Issues](../../issues).

**Adoption-KPIs:** 50 GitHub Stars und 5 externe Beiträge bis Ende Phase 4. ⭐

---

---

<a name="english"></a>

# 🇬🇧 English

## What is Intend?

**Intend** (file extension `.itd`) is a declarative, AI-optimized programming language that
consistently separates *intent* (What should be achieved?) from *implementation* (How is it computed?).
Embedded Python blocks run in a security-isolated sandbox.

Intend was designed for **AI systems and human developers to collaborate** on stable, readable,
fault-resistant programs — with Python as the embedded execution layer.

> **Core idea:** Intend does not replace Python — it structures the frame.
> Python delivers the computing power. The combination makes both layers more stable.

---

## Why Intend?

Modern LLMs generate error-prone code in classical languages — not because of bad models, but because
of structural properties of those languages:

| Problem in Python / C++ | Solution in Intend |
| --- | --- |
| Syntactic ambiguity (`*` as pointer, multiplication, or dereference) | Unambiguous Verb-Noun keywords, no symbol overloading |
| Missing context persistence (config vs. runtime state indistinguishable) | Four explicit scope types: `Set-Fixed`, `Set-Config`, `Set-Local`, `Set-Out` |
| Off-by-one and index errors in `for` loops | `Run-Each` — no manual index, automatic per-element error handling |
| No built-in error handling | `On-Error` path is mandatory for Actions and Python blocks |
| Crash on `None` access | Null-safety: `?.` and `??` — NullPointerError is impossible |
| Mixing logic and implementation | Intend layer (What) + `>>>` Python hook (How) — structurally separated |

---

## Quick Start

### Installation

```bash
pip install intend-lang
```

### First Program

```yaml
# hello.itd
# intend: v1.6.2

Set-Fixed  "Greeting": "Hello from Intend!"

New-Task "Main":
  Intent: "Prints a greeting to the console."
  Steps:
    - Run-Action: Log
      message: Set-Fixed.Greeting
      level: info
```

```bash
intend run hello.itd
```

### Validate AI-generated code (with JSON feedback)

```bash
intend check --json ai_generated.itd
```

### Compile mode (zero interpreter overhead)

```bash
intend compile script.itd    # produces script.py
python script.py
```

---

## Keyword System — Verb-Noun Principle

All keywords follow the `Verb-Noun` pattern with a controlled vocabulary of
10 verbs (`Set`, `New`, `Run`, `Check`, `Find`, `Make`, `Open`, `Close`, `Read`, `Stop`).
Anyone who knows `Run` immediately understands `Run-Task`, `Run-Job`, `Run-Each`, `Run-Every`, `Run-All`.

### Variable Scope System

```yaml
Set-Fixed  "API_URL":   "https://api.example.com"        # Constant — write-protected
Set-Config "Env":       Env("APP_ENV", Refresh: 1_hour)  # Config — env variable, hot-reload
Set-Local  "Counter":   0                                # Task-local — released after task end
Set-Out    "Result":    []                               # Explicit handoff between tasks
```

```yaml
# v1.6: Set-Persistent — state between Run-Every cycles
Set-Persistent "LastRun": None   Default: 0
```

```yaml
# v1.6: Explicit mutation (no silent overwriting)
Update Set-Local "Counter": Counter + 1
```

### Null-Safety (v1.6)

```yaml
# ?. — safe access: returns None instead of crashing
Set-Local "Name":  Item?.name   ?? "Unknown"
Set-Local "Price": Item?.price  ?? 0.0

# Safe chain across multiple levels
Set-Local "City":  Address?.location?.name ?? "No city"
```

### Custom Types (v1.5+)

```yaml
Define-Type "Item":
  id:       Integer   Required: True
  name:     Text      Required: True
  quantity: Integer   Default: 0
  price?:   Float                       # v1.6: optional field
  history?: List[Integer]               # v1.6: generic typed list
  status:   Text      Values: ["active", "blocked", "phaseout"]
```

### Loops Without Index Errors

```yaml
# Sequential iteration — automatic try/except per element
Run-Each "Item" In Set-Local.ItemList:
  - Run-Action: Log
    message: Item?.name ?? "Unknown"
    level: info
  # Single-item errors do not abort the loop
```

```yaml
# v1.6: With counter
Run-Each "Item" In Set-Local.List With-Index "i":
  - Run-Action: Log
    message: "Item {i}: {Item?.name}"
    level: debug
```

```yaml
# State loop — infinite loop is impossible
Run-Loop Until Set-Local.IsEmpty Max: 1000:
  - Run-Task "CheckStock"
```

### Scheduled Execution

```yaml
Run-Every "1 hour":
  Steps:
    - Run-Task "GenerateReport"
  On-Error:
    - Run-Action: Log
      message: "Scheduler error occurred"
      level: error
```

### Python Integration — Three Forms

```yaml
# Form 1 — single line
- Set-Local "Mean": >>>  import statistics; statistics.mean(Set-Local.Values)

# Form 2 — multi-line block
>>>
import numpy as np
arr = np.array(Set-Local.Measurements)
result = arr[arr > Set-Config.Threshold].tolist()
<<<
Set-Local "CriticalValues": result

# Form 3 — named block (recommended)
>>> as "Forecast":
  import numpy as np
  history = Set-Local.Item.history or []
  days = float(np.polyfit(range(len(history)), history, 1)[0]) if history else 999.0
<<<
Set-Local "DailyDemand": Forecast.result
```

> Python blocks run in a **three-layer sandbox**:
> static AST scan → RestrictedPython → process isolation with CPU/RAM limits.
> No network access, no write access to the host filesystem.

### Pattern Matching

```yaml
Pick-Case Set-Local.Status:
  Case "active":
    - Run-Task "Process"
  Case "blocked":
    - Run-Action: Log
      message: "Item is blocked"
      level: warn
  Other:
    - Stop-Now
      error: "UnknownStatus"
      message: "Status not recognized"
```

### Async and Parallel Execution (v1.2+)

```yaml
New-Job "LoadData":
  Steps:
    - Run-Action: HTTP_Get
      url: Set-Config.API_URL

Run-All:
  - Run-Job "LoadData"
  - Run-Job "UpdateCache"
```

### Robustness — Retry, Timeout, Circuit-Breaker (v1.4+)

```yaml
New-Task "ExternalCall":
  Retry:    3
  Timeout:  30_seconds
  On-Error: Fallback
  Steps:
    - Run-Action: HTTP_Get
      url: Set-Config.ServiceURL
```

### Resource Management

```yaml
Open-Lock "db_connection":
  - Run-Task "ReadData"
  - Run-Task "WriteData"
Close-Lock "db_connection"
# Lock is released automatically even on error
```

### IntendHub — Snippet Library (v1.5+)

```yaml
>>> from hub: "numpy/linreg-simple" version: "1.2.0"
Set-Local "Forecast": result
```

```bash
intend hub freeze        # generate intend.lock (production safety)
intend hub audit         # security scan of all snippets
intend hub update        # check snippets for new versions
```

---

## Error Types

| Error Type | Timing | Trigger |
| --- | --- | --- |
| `FixedViolation` | Compile | Write attempt on a `Set-Fixed` variable |
| `TypeMismatch` | Compile | Type conflict on assignment or parameter |
| `ScopeViolation` | Compile | Access to undeclared variable |
| `SecurityViolation` | Compile | Forbidden pattern in Python block (`__class__`, `eval`, …) |
| `ValidationError` | Compile | `Check-That` condition fails |
| `NullAccessError` | Compile | Direct access to possibly `None` field without `?.` |
| `ActionFailed` | Runtime | Action returns error — `On-Error` path is activated |
| `IterationLimit` | Runtime | `Run-Loop` exceeds `Max:` limit |
| `PythonHookError` | Runtime | Exception in `>>>` block — process isolation prevents crash |
| `ResourceLimit` | Runtime | Python block exceeds CPU time or memory limit |
| `TimeoutError` | Runtime | Task exceeds configured `Timeout:` |
| `CircuitOpen` | Runtime | Circuit-breaker opened after too many failures |

---

## CLI Reference (v1.6.2 — complete)

### Core Commands

```bash
intend run    [--env E] [--role R] [--user U] <file.itd>   # Execute
intend check  [--json]                        <file.itd>   # Validate — AI gate
intend compile                                <file.itd>   # Compile to .py
intend migrate --from X --to Y               <file.itd>   # Syntax migration
intend test   [testfile.itd]                              # Run conformance tests
```

### AI Commands (v1.5+)

```bash
intend explain       [--task T]  <file.itd>    # Natural language summary
intend verify        [--task T]  <file.itd>    # Semantic intent verification via LLM
intend suggest       [--task T]  <file.itd>    # Hub snippet recommendation
intend export-schema [--format json|md|ebnf]   # Machine-readable spec for LLMs
```

### New Commands v1.6

```bash
intend prompt   [--file F] [--append A]        # Natural language → .itd generation
intend fix      [--preview] [--safe-only]      # Auto-apply check --json corrections
intend refactor --task T --ask 'msg'           # LLM-driven refactoring
intend test-gen [--task T]                     # Automatic test generation
intend repl                                    # Interactive REPL mode
intend watch    [--env E] [--task T]           # Auto-restart on file change
intend fmt      [--check] [*.itd]             # Automatic formatting
```

### Analysis and Debugging

```bash
intend bench    [--runs N] [--task T]          # Runtime benchmark
intend profile  [--task T]                     # Bottleneck analysis
intend viz      [--format svg|mermaid|dot]     # Export task flow
intend run      --debug --step                 # Step-by-step execution
intend run      --verbose                      # Show generated Python code
```

### Hub Management

```bash
intend hub freeze                              # Generate intend.lock
intend hub update [snippet-id]                 # Update snippets
intend hub search <query>                      # Search hub snippets
intend hub audit                               # Security scan of all snippets
```

---

## Complete Example Program

```yaml
# intend: v1.6.2
# Inventory Monitor — all v1.6 features

Use-File "common/types.itd"

Set-Fixed  "MinStock":   10
Set-Fixed  "VatRate":    19 / 100
Set-Config "DB_URL":     Env("DATABASE_URL", Refresh: 1_hour, On-Change: Reconnect)
Set-Config "Email":      Env("OPS_EMAIL")

New-Task "Check_Stock":
  Intent: "Checks inventory levels. Emails ops for items below minimum stock."
  Parameters:
    - inventory: List[Item]   Required: True
  Steps:
    - Find-All In Set-Local.inventory
        Where Item?.quantity ?? 0 < Set-Fixed.MinStock
      As "critical"

    - If Set-Local.critical IS NOT Empty:
        - Run-Action: Notify_Email
          to:      Set-Config.Email
          subject: "Critical stock level"
          body:    Set-Local.critical

Run-Every "6 hours":
  Steps:
    - Run-Task "Check_Stock"
      inventory: Set-Out.CurrentInventory
  On-Error:
    - Run-Action: Log
      message: "Scheduler error"
      level:   error
```

---

## Project Structure

```
intend-lang/
├── src/intend/
│   ├── parser/          # Stage 1: lark parser + AST nodes
│   ├── typechecker/     # Stage 2: type, scope and null-safety checking
│   ├── codegen/         # Stage 3: Python code generation (Jinja2)
│   ├── runtime/         # Stage 4: execution + RestrictedPython sandbox
│   ├── actions/         # Standard action library
│   ├── hub/             # IntendHub client + intend.lock
│   └── cli/             # 25 CLI commands (v1.6.2)
├── packages/
│   └── vscode-intend/   # VS Code Extension (TypeScript + pygls)
├── tests/
│   ├── conformance/     # 200+ conformance tests
│   └── property/        # Hypothesis property-based tests
├── examples/            # Complete .itd example programs
├── docs/spec/           # Language specification v1.6.2
├── LICENSE              # Apache License 2.0
├── NOTICE               # Third-party components
└── intend.lock          # Hub snapshot (after intend hub freeze)
```

---

## Roadmap

| Phase | Timeline | Contents |
| --- | --- | --- |
| **Phase 0** — Preparation | Q2 2026 | Grammar, repository, license, 10 examples, LLM system prompt, keyword freeze |
| **Phase 1** — Interpreter Core | Q3 2026 | Parser, TypeChecker incl. null-safety, CodeGen, sandbox, `intend run/check` |
| **Phase 2** — Library & Quality | Q4 2026 | Plugin system, Hub client, compile mode, 200+ tests, Hypothesis |
| **Phase 3** — Tooling | Q1 2027 | VS Code Extension, `intend migrate`, SemVer, Docker image, `intend fmt/watch` |
| **Phase 4** — Pilot & Adoption | Q2 2027 | Pilot deployment, AI benchmark, `intend prompt`, release decision |

---

## License

This project is licensed under the **Apache License 2.0**.
See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details.

Plugin Actions may be published under their own licenses — including proprietary licenses.

---

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first (available in Phase 1).
For questions and discussions, use [GitHub Issues](../../issues).

**Adoption KPIs:** 50 GitHub Stars and 5 external contributions by end of Phase 4. ⭐

---

*INTEND Language Specification v1.6.2 — DC Software Engineering — 2026*
