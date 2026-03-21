[README.md](https://github.com/user-attachments/files/26158180/README.md)
# INTEND — Intent-Driven Programming Language

> **Eine KI-optimierte Hybridsprache für Human–AI Collaboration**
> **An AI-optimized hybrid language for Human–AI Collaboration**

[![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.11%2B-blue.svg)](https://python.org)
[![Version](https://img.shields.io/badge/Spec-v1.1-navy.svg)](docs/spec/)
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
berechnet?) trennt. Die Implementierungsebene wird durch eingebettete Python-Blöcke abgedeckt, die in
einer sicherheitsisolierten Sandbox laufen.

Intend wurde entwickelt, damit **KI-Systeme und menschliche Entwickler gemeinsam** stabile, lesbare
und fehlerresistente Programme erstellen — mit Python als eingebetteter Ausführungsschicht.

> **Kerngedanke:** Intend ersetzt Python nicht — es strukturiert den Rahmen.
> Python liefert die Rechenleistung. Die Kombination macht beide Ebenen stabiler.

---

## Warum Intend?

Moderne LLMs wie Claude, GPT oder Gemini generieren in klassischen Sprachen wie Python fehleranfälligen
Code — nicht wegen schlechter Modelle, sondern wegen struktureller Eigenschaften der Sprachen:

| Problem in Python / C++ | Lösung in Intend |
|---|---|
| Syntaktische Ambiguität (`*` als Pointer, Multiplikation oder Dereferenzierung) | Eindeutige Keywords, keine Symbolüberlagerung |
| Fehlende Kontextpersistenz (Konfiguration vs. Laufzustand ununterscheidbar) | Vier explizite Scope-Typen: `Immutable`, `Context`, `State`, `Shared` |
| Off-by-One und Index-Fehler in `for`-Schleifen | `Process Every` — kein manueller Index, automatisches Error-Handling |
| Kein eingebautes Fehler-Handling | `On_Error`-Pfad ist Pflicht bei Actions und Python-Hooks |
| Vermischung von Logik und Implementierung | Intend-Schicht (Was) + Python-Hook (Wie) — strukturell getrennt |

---

## Schnellstart

### Installation

```bash
pip install intend-lang
```

### Erstes Programm

```yaml
# hello.itd
# intend: v1.1

Immutable Begruessung: Text = "Hallo von Intend!"

Define Task Hauptprogramm:
  Action Log:
    message: Begruessung
    level: info
```

```bash
intend run hello.itd
```

### KI-generierter Code validieren

```bash
intend check ki_generiert.itd
```

### Compile-Modus (kein Interpreter-Overhead)

```bash
intend compile script.itd   # erzeugt script.py
python script.py
```

---

## Sprachkonstrukte — Übersicht

### Variablen-Scope-System

```yaml
Immutable API_URL: Text = "https://api.example.com"   # Konstante — nicht veränderbar
Context  Umgebung: Text = "produktion"                 # Konfiguration — global sichtbar
State    Zaehler:  Number = 0                          # Task-lokal — wird nach Task-Ende freigegeben
Shared   Ergebnis: List  = []                          # Explizite Übergabe zwischen Tasks
```

### Schleifen ohne Index-Fehler

```yaml
Process Every Artikel in Artikelliste:
  Action Log:
    message: Artikel.Name
    level: info
  # Einzelfehler brechen die Schleife nicht ab — automatisches try/except
```

```yaml
Repeat Until Lager.IstLeer Max_Iterations: 1000:
  Call Task Bestand_Pruefen
# Kein Endlosloop möglich — Pflichtlimit
```

### Zeitgesteuerte Ausführung

```yaml
Schedule Every 1 hour:
  Call Task Bericht_Erstellen
  On_Error:
    Action Log:
      message: "Scheduler-Fehler aufgetreten"
      level: error
```

### Python-Hook (sicher isoliert)

```yaml
Execute Python:
  Import: [numpy as np, pandas as pd]
  Inject: [State.Messwerte as 'raw_data', Context.Schwellenwert as 'threshold']
  Code: |
    df = pd.DataFrame(raw_data)
    result = df[df['wert'] > threshold]['id'].tolist()
  Export_As: 'KritischeArtikel'
  On_Error: Log_And_Continue
```

> Der Python-Hook läuft in einer **Drei-Ebenen-Sandbox**: statischer AST-Scan →
> RestrictedPython → Prozess-Isolation mit Ressourcenlimits. Kein Netzwerkzugriff,
> kein Schreibzugriff auf das Dateisystem des Hosts.

### Externe Actions einbinden (Plugin-System)

```yaml
Use Action 'intend-slack-action' as Notify_Slack

Define Task Benachrichtigung:
  Action Notify_Slack:
    webhook_url: Context.Slack_Webhook
    message: "Bericht fertig"
    channel: "#ops"
```

---

## Fehlertypen

| Fehlertyp | Zeitpunkt | Auslöser |
|---|---|---|
| `ImmutableViolation` | Compile | Schreibversuch auf `Immutable`-Variable |
| `TypeMismatch` | Compile | Typkonflikt bei Zuweisung |
| `ScopeViolation` | Compile | Zugriff auf nicht deklarierte Variable |
| `SecurityViolation` | Compile | Verbotenes Muster im Python-Hook (`__class__`, `eval`, …) |
| `ActionFailed` | Runtime | Action liefert Fehler — `On_Error`-Pfad wird aktiviert |
| `IterationLimit` | Runtime | `Repeat Until` überschreitet `Max_Iterations` |
| `PythonHookError` | Runtime | Exception im Hook — Prozess-Isolation verhindert Absturz |
| `ResourceLimit` | Runtime | Hook überschreitet CPU-Zeit oder Speicher-Limit |

---

## CLI-Referenz

```bash
intend run    <datei.itd>                   # Ausführen
intend check  <datei.itd>                   # Validieren (ohne Ausführung) — KI-Gate
intend compile <datei.itd>                  # Zu .py kompilieren
intend migrate --from 1.0 --to 1.1 <datei> # Syntax automatisch migrieren
intend run --verbose <datei.itd>            # Generierten Python-Code anzeigen
```

---

## Projektstruktur

```
intend-lang/
├── src/intend/
│   ├── parser/          # Stufe 1: lark-Parser + AST-Knoten
│   ├── typechecker/     # Stufe 2: Typ- und Scope-Prüfung
│   ├── codegen/         # Stufe 3: Python-Code-Generierung (Jinja2)
│   ├── runtime/         # Stufe 4: Ausführung + RestrictedPython-Sandbox
│   ├── actions/         # Standard-Action-Bibliothek
│   └── cli/             # intend run / check / compile / migrate
├── packages/
│   └── vscode-intend/   # VS Code Extension (TypeScript + pygls)
├── tests/
│   ├── conformance/     # 150+ Conformance-Tests
│   └── property/        # Hypothesis Property-Tests
├── examples/            # Vollständige .itd-Beispielprogramme
├── docs/spec/           # Sprachspezifikation
├── LICENSE              # Apache License 2.0
└── NOTICE               # Drittanbieter-Komponenten
```

---

## Umsetzungsplan

| Phase | Zeitraum | Inhalt |
|---|---|---|
| **Phase 0** — Vorbereitung | Q2 2026 | Grammatik, Repository, Lizenz, 10 Beispiele, LLM-System-Prompt |
| **Phase 1** — Interpreter-Kern | Q3 2026 | Parser, TypeChecker, CodeGen, Sandbox, 6 Standard-Actions, `intend run/check` |
| **Phase 2** — Bibliothek & Qualität | Q4 2026 | Plugin-System, Compile-Modus, 150+ Tests, Hypothesis |
| **Phase 3** — Tooling | Q1 2027 | VS Code Extension, `intend migrate`, SemVer, Docker-Image |
| **Phase 4** — Pilot & Adoption | Q2 2027 | Piloteinsatz, KI-Benchmark, Release-Entscheidung |

---

## Lizenz

Dieses Projekt steht unter der **Apache License 2.0**.
Siehe [LICENSE](LICENSE) und [NOTICE](NOTICE) für Details.

Plugin-Actions dürfen unter eigenen Lizenzen — einschließlich proprietärer Lizenzen — veröffentlicht werden.

---

## Mitmachen

Beiträge sind willkommen. Bitte lies zuerst [CONTRIBUTING.md](CONTRIBUTING.md) (folgt in Phase 1).
Für Fragen und Diskussionen nutze die [GitHub Issues](../../issues).

**Adoption-KPIs:** 50 GitHub Stars und 5 externe Beiträge bis Ende Phase 4.
Hilf mit, dieses Ziel zu erreichen — ⭐ Star das Repository!

---
---

<a name="english"></a>
# 🇬🇧 English

## What is Intend?

**Intend** (file extension `.itd`) is a declarative, AI-optimized programming language that
consistently separates the level of *intent* (What should be achieved?) from the level of
*implementation* (How is it computed?). The implementation layer is covered by embedded Python blocks
running in a security-isolated sandbox.

Intend was designed to allow **AI systems and human developers to collaborate** on creating stable,
readable, and fault-resistant programs — with Python as the embedded execution layer.

> **Core idea:** Intend does not replace Python — it structures the frame.
> Python delivers the computing power. The combination makes both layers more stable.

---

## Why Intend?

Modern LLMs like Claude, GPT, or Gemini generate error-prone code in classical languages like Python —
not because of bad models, but because of structural properties of those languages:

| Problem in Python / C++ | Solution in Intend |
|---|---|
| Syntactic ambiguity (`*` as pointer, multiplication, or dereference) | Unambiguous keywords, no symbol overloading |
| Missing context persistence (configuration vs. runtime state indistinguishable) | Four explicit scope types: `Immutable`, `Context`, `State`, `Shared` |
| Off-by-one and index errors in `for` loops | `Process Every` — no manual index, automatic error handling |
| No built-in error handling | `On_Error` path is mandatory for Actions and Python hooks |
| Mixing logic and implementation | Intend layer (What) + Python hook (How) — structurally separated |

---

## Quick Start

### Installation

```bash
pip install intend-lang
```

### First Program

```yaml
# hello.itd
# intend: v1.1

Immutable Greeting: Text = "Hello from Intend!"

Define Task Main:
  Action Log:
    message: Greeting
    level: info
```

```bash
intend run hello.itd
```

### Validate AI-generated code

```bash
intend check ai_generated.itd
```

### Compile mode (zero interpreter overhead)

```bash
intend compile script.itd   # produces script.py
python script.py
```

---

## Language Constructs — Overview

### Variable Scope System

```yaml
Immutable API_URL:   Text   = "https://api.example.com"  # Constant — write-protected
Context   Env:       Text   = "production"                # Config — globally visible
State     Counter:   Number = 0                           # Task-local — released after task end
Shared    Result:    List   = []                          # Explicit handoff between tasks
```

### Loops Without Index Errors

```yaml
Process Every Item in ItemList:
  Action Log:
    message: Item.Name
    level: info
  # Single-item errors do not abort the loop — automatic try/except
```

```yaml
Repeat Until Inventory.IsEmpty Max_Iterations: 1000:
  Call Task Check_Stock
# No infinite loop possible — limit is mandatory
```

### Scheduled Execution

```yaml
Schedule Every 1 hour:
  Call Task Generate_Report
  On_Error:
    Action Log:
      message: "Scheduler error occurred"
      level: error
```

### Python Hook (security-isolated)

```yaml
Execute Python:
  Import: [numpy as np, pandas as pd]
  Inject: [State.Measurements as 'raw_data', Context.Threshold as 'threshold']
  Code: |
    df = pd.DataFrame(raw_data)
    result = df[df['value'] > threshold]['id'].tolist()
  Export_As: 'CriticalItems'
  On_Error: Log_And_Continue
```

> The Python hook runs in a **three-layer sandbox**: static AST scan →
> RestrictedPython → process isolation with resource limits. No network access,
> no write access to the host filesystem.

### External Actions via Plugin System

```yaml
Use Action 'intend-slack-action' as Notify_Slack

Define Task SendNotification:
  Action Notify_Slack:
    webhook_url: Context.Slack_Webhook
    message: "Report complete"
    channel: "#ops"
```

---

## Error Types

| Error Type | Timing | Trigger |
|---|---|---|
| `ImmutableViolation` | Compile | Write attempt on an `Immutable` variable |
| `TypeMismatch` | Compile | Type conflict on assignment |
| `ScopeViolation` | Compile | Access to undeclared variable |
| `SecurityViolation` | Compile | Forbidden pattern in Python hook (`__class__`, `eval`, …) |
| `ActionFailed` | Runtime | Action returns error — `On_Error` path is activated |
| `IterationLimit` | Runtime | `Repeat Until` exceeds `Max_Iterations` |
| `PythonHookError` | Runtime | Exception in hook — process isolation prevents crash |
| `ResourceLimit` | Runtime | Hook exceeds CPU time or memory limit |

---

## CLI Reference

```bash
intend run    <file.itd>                      # Execute
intend check  <file.itd>                      # Validate (without execution) — AI gate
intend compile <file.itd>                     # Compile to .py
intend migrate --from 1.0 --to 1.1 <file>    # Auto-migrate syntax
intend run --verbose <file.itd>               # Show generated Python code
```

---

## Project Structure

```
intend-lang/
├── src/intend/
│   ├── parser/          # Stage 1: lark parser + AST nodes
│   ├── typechecker/     # Stage 2: type and scope checking
│   ├── codegen/         # Stage 3: Python code generation (Jinja2)
│   ├── runtime/         # Stage 4: execution + RestrictedPython sandbox
│   ├── actions/         # Standard action library
│   └── cli/             # intend run / check / compile / migrate
├── packages/
│   └── vscode-intend/   # VS Code Extension (TypeScript + pygls)
├── tests/
│   ├── conformance/     # 150+ conformance tests
│   └── property/        # Hypothesis property-based tests
├── examples/            # Complete .itd example programs
├── docs/spec/           # Language specification
├── LICENSE              # Apache License 2.0
└── NOTICE               # Third-party components
```

---

## Roadmap

| Phase | Timeline | Contents |
|---|---|---|
| **Phase 0** — Preparation | Q2 2026 | Grammar, repository, license, 10 examples, LLM system prompt |
| **Phase 1** — Interpreter Core | Q3 2026 | Parser, TypeChecker, CodeGen, sandbox, 6 standard actions, `intend run/check` |
| **Phase 2** — Library & Quality | Q4 2026 | Plugin system, compile mode, 150+ tests, Hypothesis |
| **Phase 3** — Tooling | Q1 2027 | VS Code Extension, `intend migrate`, SemVer, Docker image |
| **Phase 4** — Pilot & Adoption | Q2 2027 | Pilot deployment, AI benchmark, release decision |

---

## License

This project is licensed under the **Apache License 2.0**.
See [LICENSE](LICENSE) and [NOTICE](NOTICE) for details.

Plugin Actions may be published under their own licenses — including proprietary licenses.

---

## Contributing

Contributions are welcome. Please read [CONTRIBUTING.md](CONTRIBUTING.md) first (available in Phase 1).
For questions and discussions, use [GitHub Issues](../../issues).

**Adoption KPIs:** 50 GitHub Stars and 5 external contributions by end of Phase 4.
Help us reach that goal — ⭐ Star the repository!

---

*INTEND Language Specification v1.1 — DC Software Engineering — 2026*
