<div align="center">

<img src="assets/banner.svg" width="900" alt="Habitus">

**Your AI agents should learn how you work — not the other way around.**

*Bottom-up behavioral profiling from file-system traces, screen recordings, and content deltas.*

[![Product Page](https://img.shields.io/badge/🌐_Product_Page-habitus.choiszt.com-38bdf8?style=flat-square)](https://habitus.choiszt.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg?style=flat-square)](https://www.python.org/downloads/)

[Features](#features) · [Architecture](#architecture) · [Quick Start](#quick-start) · [How It Works](#how-it-works) · [Comparison](#comparison) · [Paper](#academic-foundation)

</div>

---

## Why Habitus?

Every AI coding agent asks you to **manually describe your preferences** — write a CLAUDE.md, configure .cursorrules, set up AGENTS.md. But:

- **You don't know your own habits.** People are notoriously bad at self-reporting behavioral patterns.
- **Preferences evolve.** Your style today isn't your style 3 months ago. Static config files rot.
- **Behavior speaks louder than words.** What you *do* with files reveals more than what you *say* you prefer.

**Habitus watches how you work and automatically generates behavioral profiles that any AI agent can use.**

```
You work normally for a week
    → Habitus observes: file operations, content deltas, screen activity
    → Three-channel engine: statistical features + LLM interpretation
    → Outputs: "This user reads sequentially, writes comprehensively,
               organizes in deep hierarchies, iterates incrementally"
    → Exports as CLAUDE.md / .cursorrules / AGENTS.md
    → Every AI agent you use now understands your work style
```

---

## Features

### 🔍 Three Input Channels

| Channel | Signal | How It Works |
|---------|--------|-------------|
| **File System Watcher** | Real-time file operations | Monitors reads, writes, edits, moves, renames, deletes via OS-level hooks (FSEvents/inotify) |
| **Content Delta** | What changed and how | Shadow copies + unified diffs + content-addressable storage (CAS). Tracks every modification with full before/after state |
| **Screen Recording** | Visual behavior patterns | Captures screen activity → VLM extracts file operation sequences, reading patterns, navigation habits |

### 🧠 Three-Channel Profiling Engine

Based on the [FileGramOS](https://github.com/choiszt/FileGram) memory architecture:

| Channel | What It Captures | Method |
|---------|-----------------|--------|
| **Procedural** | How you work — reading strategies, navigation patterns, tool preferences | Statistical feature extraction → LLM classification and interpretation |
| **Semantic** | What you produce — writing style, content structure, edit motivations | Content delta analysis → LLM behavioral narrative encoding |
| **Episodic** | Behavioral consistency — stable habits vs. evolving patterns over time | Cross-session aggregation → LLM episode segmentation and drift analysis |

Each channel combines **bottom-up statistical grounding** with **LLM-powered interpretation**. Raw behavioral signals are never lost to premature abstraction.

### 📊 6-Dimension Behavioral Model

| Dim | Name | What It Captures | Spectrum |
|-----|------|-----------------|----------|
| **A** | Consumption | How you explore and read files | Sequential ↔ Targeted ↔ Breadth-first |
| **B** | Production | Your output style and detail level | Comprehensive ↔ Balanced ↔ Minimal |
| **C** | Organization | How you structure directories and name files | Deeply nested ↔ Adaptive ↔ Flat |
| **D** | Iteration | How you revise and refine work | Incremental ↔ Balanced ↔ Rewrite |
| **E** | Curation | How you manage workspace cleanliness | Selective cleanup ↔ Pragmatic ↔ Preservative |
| **F** | Cross-Modal | Whether you use visual materials | Visual-heavy ↔ Balanced ↔ Text-only |

Each dimension is classified as **Left / Middle / Right (L/M/R)** based on observable behavioral indicators.

### 📤 Universal Export

Generate preference files for any AI agent — no integration needed:

```bash
habitus export --format claude      # → CLAUDE.md
habitus export --format cursor      # → .cursorrules
habitus export --format agents      # → AGENTS.md
habitus export --format json        # → behavioral_profile.json
```

### 🧬 `.habitus` Memory — Knowledge That Compounds

Habitus maintains a persistent behavioral wiki at `~/.habitus/profile/` — a set of interlinked markdown pages that the LLM incrementally updates after each work session. This is not RAG. Nothing is re-derived from scratch.

```
Session 1: You organize files into deep hierarchies
  → organization.md: "Prefers 3+ level directory structures with date prefixes"

Session 5: Same pattern confirmed
  → organization.md: "Consistently uses deep nesting (5/5 sessions, 100%)"

Session 12: You suddenly flatten a project
  → organization.md: "⚠️ Behavioral shift: flat organization observed in session 12,
     contradicting established deep-nesting pattern. Monitoring."
```

**Pages**: index · organization · editing · reading · production · workflows · tools · evolution

Each page is a living document — observations accumulate, contradictions are flagged, and confidence grows with evidence. The wiki is the compiled artifact; raw events are the source of truth.

> Inspired by the [LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) pattern — behavioral knowledge that compounds over time, not retrieved from scratch.

### 📈 Profile Evolution

Your habits change over time. Habitus tracks this:

- **Behavioral drift detection** — alerts when your work patterns shift
- **Dimension-level evolution** — see which specific habits are changing
- **Temporal profile snapshots** — compare your profile from last month vs. today

### 🔒 Privacy-First

- All raw data stays on your device
- LLM calls only process behavioral features, never raw file content
- Screen recordings are processed locally; only extracted events are stored
- Full control over what gets analyzed

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    INPUT CHANNELS                         │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────┐  │
│  │ FS Watcher   │  │ Content      │  │ Screen        │  │
│  │ (watchdog +  │  │ Delta        │  │ Recording     │  │
│  │  fs_usage)   │  │ (shadow +    │  │ (capture +    │  │
│  │              │  │  CAS + diff) │  │  VLM extract) │  │
│  └──────┬───────┘  └──────┬───────┘  └──────┬────────┘  │
│         └─────────────┬───┘──────────────────┘           │
│                       ▼                                  │
│              events.json (FileGram schema)                │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                 PROFILING ENGINE                          │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Per-Session Encoding (EngramEncoder)               │ │
│  │  events → statistical features + LLM semantic       │ │
│  │         + fingerprint + episode segmentation        │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                         ▼                                │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Cross-Session Consolidation (EngramConsolidator)   │ │
│  │  N engrams → aggregated stats + consistency flags   │ │
│  │            + deviation detection + clustering       │ │
│  └──────────────────────┬──────────────────────────────┘ │
│                         ▼                                │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Three-Channel Retrieval (QueryAdaptiveRetriever)   │ │
│  │  Procedural │ Semantic │ Episodic → Profile         │ │
│  └─────────────────────────────────────────────────────┘ │
└───────────────────────┬─────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                    OUTPUT                                 │
│                                                          │
│  ┌────────────┐ ┌────────────┐ ┌───────────┐ ┌────────┐ │
│  │ CLAUDE.md  │ │.cursorrules│ │ AGENTS.md │ │  JSON  │ │
│  └────────────┘ └────────────┘ └───────────┘ └────────┘ │
│                                                          │
│  ┌─────────────────────────────────────────────────────┐ │
│  │  Dashboard — 6-dim radar, timeline, drift alerts    │ │
│  └─────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────┘
```

---

## Quick Start

### Installation

```bash
# Using uv (recommended)
uv pip install -e .

# Or from PyPI
uv pip install habitus
```

### Watch your workspace

```bash
# Start observing file operations in your project directory
uv run habitus watch ~/projects/my-project

# Work normally — Habitus runs silently in the background
# ... edit files, create directories, reorganize, write docs ...

# After a few work sessions, check your profile
uv run habitus profile

# Export for your favorite AI agent
uv run habitus export --format claude --output ~/projects/my-project/CLAUDE.md
```

### Analyze existing trajectories

```bash
# If you already have FileGram-format events.json files
uv run habitus analyze ./data/sessions/*/events.json

# View profile with full three-channel detail
uv run habitus profile --verbose
```

### Dashboard

```bash
# Launch the web dashboard
uv run habitus dashboard
# → http://localhost:8420
```

---

## How It Works

### Step 1: Observe

Habitus captures file-system events in real-time:

```
file_read    — what files you open, how deeply you read, revisit patterns
file_write   — what you create, content length, file types
file_edit    — how you modify files, diff size, edit frequency
file_move    — how you reorganize, destination depth
file_rename  — naming convention changes
dir_create   — directory structure preferences
file_delete  — cleanup behavior, what you discard
file_copy    — backup and versioning patterns
fs_snapshot  — periodic workspace state captures
```

### Step 2: Encode

Each work session becomes an **Engram** — a structured behavioral memory unit:

- **Procedural features**: Statistical extraction (read counts, edit sizes, directory depths, timing patterns) → LLM classifies into behavioral dimensions
- **Semantic encoding**: LLM analyzes content deltas to understand writing style, document structure, level of detail
- **Behavioral fingerprint**: Compact numerical vector for cross-session comparison
- **Episode segmentation**: LLM identifies logical work phases within a session

### Step 3: Consolidate

Across multiple sessions, Habitus builds a stable profile:

- **Feature aggregation**: Mean/median/std across sessions per dimension
- **Consistency analysis**: Which habits are stable vs. variable
- **Deviation detection**: Flag sessions that deviate from established patterns
- **Behavioral clustering**: Group similar work sessions

### Step 4: Export

The consolidated profile is rendered into preference files:

```markdown
# CLAUDE.md (auto-generated by Habitus)

## My Work Style
- I read files sequentially and thoroughly before making changes
- I prefer comprehensive output with multiple heading levels and data tables
- I organize files in deeply nested directory structures with descriptive names
- I iterate incrementally — small edits, frequent reviews, always keep backups
- I actively clean up temporary files after completing tasks
- I include visual materials (charts, diagrams) when data supports them
```

---

## Comparison

| | **Habitus** | MetaClaw | Mem0 | Zep | CLAUDE.md |
|---|:---:|:---:|:---:|:---:|:---:|
| **Signal source** | File operations + content deltas | Conversations | Conversations | Conversations | Manual writing |
| **Profiling method** | Stats + LLM interpretation | LLM + RL fine-tuning | LLM extraction | LLM → KG triples | Human self-report |
| **Structured dimensions** | 6-dim L/M/R model | Unstructured skills | Unstructured memory | Entity-relation graph | Free text |
| **Profile evolution** | ✅ Drift detection | ✅ Continuous RL | ❌ | ⚠️ Temporal edges | ❌ Manual update |
| **Captures non-verbal habits** | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Universal export** | ✅ Any agent format | ❌ Own ecosystem | ❌ API only | ❌ API only | ⚠️ One format |
| **Evaluated on benchmark** | ✅ FileGramBench | ❌ | ❌ | ❌ | ❌ |

---

## Project Structure

```
Habitus/
├── habitus/
│   ├── channels/          # Input: FS watcher, content delta, screen recording
│   ├── engine/            # Core: feature extraction, encoding, consolidation, retrieval
│   ├── export/            # Output: CLAUDE.md, .cursorrules, AGENTS.md, JSON
│   ├── storage/           # Persistence: SQLite session DB, event storage
│   └── server/            # API server for dashboard
├── dashboard/             # Web UI: 6-dim radar, event timeline, drift visualization
├── extensions/            # Agent-specific integrations
│   ├── claude-code/       #   Claude Code hooks + subagent
│   └── cursor/            #   Cursor rules auto-generation
├── tests/
├── docs/
└── assets/
```

---

## Configuration

```yaml
# habitus.yaml
watch_dirs:
  - ~/projects/my-project
  - ~/Documents/reports

ignore_patterns:
  - "**/.git/**"
  - "**/node_modules/**"
  - "**/__pycache__/**"

llm:
  provider: ollama           # openai | anthropic | azure | ollama
  model: gemma3:1b           # Local model via Ollama (or gpt-4.1 for cloud)
  
profiling:
  snapshot_interval: 300     # Directory snapshots every 5 min
  consolidation_interval: 10 # Consolidate every 10 sessions
  dedup_window: 2.0          # Merge rapid edits within 2 seconds

export:
  auto_update: true          # Re-export on profile change
  formats: [claude, cursor]  # Auto-export formats
  output_dir: .              # Where to write preference files
```

---

## Academic Foundation

Habitus is built on the research framework from the **FileGram** project:

- **FileGram**: Persona-driven behavioral data generation from file-system traces
- **FileGramBench**: Multimodal benchmark for evaluating behavioral memory systems (1058 MCQ questions, 4 evaluation tracks)
- **FileGramOS**: Three-channel memory architecture (Procedural + Semantic + Episodic) with deferred abstraction

The 6-dimension L/M/R behavioral model and three-channel profiling engine are validated on 640 trajectories across 20 user profiles and 32 tasks.

> **Core thesis**: User preferences are best inferred from what users *do* with files, not from what they *say*.

📄 Paper: *FileGram: Grounding Agent Memory in File-System Behavioral Traces* (2026)

---

## Roadmap

- [x] File system watcher with content delta tracking
- [x] Three-channel profiling engine (Procedural + Semantic + Episodic)
- [x] 6-dimension behavioral model with L/M/R classification
- [x] Export to CLAUDE.md, .cursorrules, AGENTS.md
- [ ] Web dashboard with behavioral radar chart
- [ ] Screen recording channel (VLM-based extraction)
- [ ] Profile evolution tracking and drift alerts
- [ ] Multi-workspace profiling (different profiles per project)
- [ ] Claude Code extension (hooks + subagent)
- [ ] Cursor Marketplace plugin

---

## Contributing

We welcome contributions! See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for guidelines.

---

## License

[MIT](LICENSE)

---

<div align="center">
<sub>Built on the FileGram research framework · HKU</sub>
</div>
