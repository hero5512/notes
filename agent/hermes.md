# Hermes Agent Full Research Document

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Tech Stack](#2-tech-stack)
3. [Directory Structure](#3-directory-structure)
4. [High-Level Architecture](#4-high-level-architecture)
5. [Sessions](#5-sessions)
6. [System Prompt Construction](#6-system-prompt-construction)
7. [Frozen Snapshot Mode](#7-frozen-snapshot-mode)
8. [Memory System](#8-memory-system)
9. [Context Compression](#9-context-compression)
10. [Tools and Skills System](#10-tools-and-skills-system)
11. [Multi-Provider LLM Support](#11-multi-provider-llm-support)
12. [Sandboxing and Execution Environments](#12-sandboxing-and-execution-environments)
13. [Security Mechanisms](#13-security-mechanisms)
14. [Multi-Platform Messaging Gateway](#14-multi-platform-messaging-gateway)
15. [Configuration System](#15-configuration-system)
16. [Data Storage](#16-data-storage)
17. [API and Interfaces](#17-api-and-interfaces)
18. [Scheduled Tasks (Cron)](#18-scheduled-tasks-cron)
19. [Build and Deployment](#19-build-and-deployment)
20. [Testing Methodology](#20-testing-methodology)
21. [Key Design Patterns](#21-key-design-patterns)
22. [Statistical Summary](#22-statistical-summary)

---

## 1. Project Overview

**Hermes Agent** is a self-improving, production-grade AI Agent framework built by Nous Research.

**Core Objectives**:
- Create programmatic skills from experience and continuously improve them during use.
- Support multiple deployment scenarios (Local, Cloud VPS, Serverless, Containers).
- Interface with any LLM provider, avoiding vendor lock-in.
- Connect to multiple messaging platforms (Telegram, Discord, Slack, WhatsApp, etc.).
- Maintain persistent memory, session continuity, and cross-session knowledge recall.
- Support scheduled automation (Cron-style with natural language expressions).
- Parallel execution and delegated isolation via sub-agents.

---

## 2. Tech Stack

| Layer | Technology |
|------|------|
| **Core Language** | Python 3.11+ |
| **Frontend/TUI** | TypeScript, Node.js 18+, React (Ink) |
| **LLM Clients** | `openai>=2.21.0`, `anthropic>=0.39.0` |
| **Web/HTTP** | `httpx[socks]>=0.28.1`, `requests>=2.33.0` |
| **Terminal UI** | `prompt_toolkit>=3.0.52`, `rich>=14.3.3` |
| **Config Validation** | `pydantic>=2.12.5`, `python-dotenv>=1.2.1` |
| **Web Search** | `firecrawl-py>=4.16.0`, `exa-py>=2.9.0` |
| **TTS** | `edge-tts>=7.2.7` (Free), `elevenlabs>=1.0` (Paid) |
| **MCP Protocol** | `mcp>=1.2.0` |
| **Speech Recognition** | `faster-whisper>=1.0.0`, `sounddevice>=0.4.6` |
| **Browser Automation**| `agent-browser>=0.13.0`, `playwright` |
| **Package Management**| `uv` (~10x faster than pip) |
| **Frontend Build** | Vite |
| **Containers** | Docker Multi-stage, NixOS flake |

---

## 3. Directory Structure

```
hermes-agent/
├── run_agent.py              # AIAgent class, core conversation loop
├── cli.py                    # Interactive TUI (prompt_toolkit)
├── batch_runner.py           # Parallel batch processing
├── mcp_serve.py              # MCP stdio server (for Claude Code integration)
│
├── agent/                    # Agent core modules
│   ├── anthropic_adapter.py       # Claude API adapter
│   ├── bedrock_adapter.py         # AWS Bedrock adapter
│   ├── gemini_native_adapter.py   # Google Gemini adapter
│   ├── auxiliary_client.py        # Multi-provider LLM routing
│   ├── context_compressor.py      # Automatic context summarization
│   ├── prompt_builder.py          # System prompt construction (1053 lines)
│   ├── model_metadata.py          # Model token limit metadata
│   ├── memory_manager.py          # External memory plugin orchestration
│   ├── memory_provider.py         # MemoryProvider ABC
│   └── prompt_caching.py          # Anthropic Prompt Cache control
│
├── tools/                    # Tool system (40+ tools)
│   ├── registry.py                # Tool registration, discovery, distribution
│   ├── terminal_tool.py           # Bash/command execution
│   ├── file_tools.py              # File read/write/search
│   ├── web_tools.py               # Web search/extraction/crawling
│   ├── browser_tool.py            # Browser automation (CDP)
│   ├── delegate_tool.py           # Sub-agent generation
│   ├── mcp_tool.py                # MCP server/client
│   ├── memory_tool.py             # Memory persistence
│   ├── cronjob_tools.py           # Scheduled task management
│   ├── skill_manager_tool.py      # Skill CRUD
│   ├── approval.py                # Dangerous command approval
│   ├── path_security.py           # Path traversal protection
│   ├── tirith_security.py         # External content security scanning
│   └── environments/              # Execution backends (6 types)
│       ├── base.py                # BaseEnvironment ABC
│       ├── local.py               # Local subprocess
│       ├── docker.py              # Docker container
│       ├── ssh.py                 # Remote SSH
│       ├── modal.py               # Modal Serverless
│       ├── daytona.py             # Daytona cloud dev environment
│       └── singularity.py         # HPC Singularity container
│
├── agent/
│   └── redact.py                  # Credential redaction (30+ patterns)
│
├── hermes_cli/               # CLI and Configuration
│   ├── main.py                    # Command routing entry point
│   ├── config.py                  # Config loading, migration, Schema validation
│   ├── default_soul.py            # Default SOUL.md content
│   ├── gateway.py                 # Gateway configuration and startup
│   ├── models.py                  # Model catalog (200+ model metadata)
│   ├── web_server.py              # FastAPI Web dashboard
│   └── profiles.py                # Profile management
│
├── gateway/                  # Multi-platform messaging gateway
│   ├── run.py                     # Main loop, authorization, Agent lifecycle
│   ├── pairing.py                 # DM pairing code mechanism
│   ├── session.py                 # Session storage
│   └── platforms/                 # 14+ Platform adapters
│       ├── telegram.py / discord.py / slack.py
│       ├── whatsapp.py / signal.py / matrix.py
│       ├── email.py / feishu.py / dingtalk.py
│       └── weixin.py / wecom.py / qqbot/
│
├── plugins/memory/           # External memory plugins (8 providers)
│   ├── honcho/  hindsight/  mem0/  retaindb/
│   └── byterover/  holographic/  openviking/  supermemory/
│
├── skills/                   # Built-in skills (30+ domains, 100+ skills)
├── cron/                     # Scheduled tasks (jobs.py + scheduler.py)
├── ui-tui/                   # Ink/React terminal UI
├── web/                      # Vite + React Web dashboard
├── tests/                    # ~3000 test cases
└── hermes_state.py           # SQLite session storage (with FTS5)
```

---

## 4. High-Level Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                       User Interface (3 Paths)                   │
├─────────────────────┬──────────────────┬────────────────────────┤
│  CLI (TUI)           │  Messaging Gateway│   Web Dashboard        │
│  cli.py             │  gateway/run.py  │   web_server.py        │
└──────────┬──────────┴────────┬─────────┴──────────┬────────────┘
           └───────────────────┴────────────────────┘
                               │
              ┌────────────────▼─────────────────┐
              │        AIAgent (run_agent.py)     │
              │   Core Loop + Tool Execution Engine│
              └────┬───────────┬─────────────┬───┘
                   │           │             │
          ┌────────▼──┐  ┌─────▼──────┐  ┌──▼────────────┐
          │ System      │  │  Memory    │  │  Tool         │
          │ Prompt      │  │  System    │  │  Registry     │
          │ Builder     │  │  Built-in  │  │  40+ Tools     │
          └───────────┘  └────────────┘  └───────────────┘
                               │
              ┌────────────────▼─────────────────┐
              │      Persistence & State Management│
              │  hermes_state.py (SQLite + FTS5) │
              └──────────────────────────────────┘
```

**Key Design Principles**:
- **Self-registering tools**: Tools call `registry.register()` automatically upon import, eliminating the need for a central maintenance list.
- **Provider-agnostic adapters**: Each LLM provider has a dedicated adapter, internally unified using OpenAI-style message formatting.
- **Async-Sync Bridge**: Supports both CLI (synchronous) and Gateway (asynchronous) contexts simultaneously.
- **Plugin Hook System**: Provides hooks for pre/post tool execution, LLM calls, and session lifecycles.

---

## 5. Sessions

A **Session** is the recording unit for a complete conversation from start to finish, stored in `~/.hermes/state.db` (SQLite).

### Session Fields

| Field | Meaning |
|------|------|
| `id` | Unique identifier (UUID) |
| `source` | Source (`cli`, `telegram`, `discord`, etc.) |
| `user_id` | User identifier |
| `model` | LLM used |
| `system_prompt` | Complete system prompt snapshot for this session |
| `parent_session_id`| Parent session ID (generated during context compression) |
| `started_at / ended_at` | Timestamps |
| `end_reason` | Reason for ending (`end_turn`, `error`, `timeout`, etc.) |
| `input/output_tokens` | Token usage |
| `actual_cost_usd` | Actual cost in USD |
| `title` | Auto-generated or user-defined title |

The `messages` table records all messages for the session (each containing `role`, `content`, timestamp, and tool call details).

### Session Lifecycle

```
User sends the first message
  → Create session (INSERT into sessions)
  → Append message for each turn (INSERT into messages)
  → Record tool call results
  → User /new, timeout, or error
  → Session closes (fill ended_at + end_reason)
```

### Session Splitting during Context Compression

When a conversation is too long and triggers compression, a **sub-session** is generated (`parent_session_id` points to the original). It continues from the compressed summary without losing history, forming a chained structure.

---

## 6. System Prompt Construction

`run_agent.py` `_build_system_prompt()` is called once during the **first turn** of a session. The result is cached in `_cached_system_prompt` and reused for subsequent turns. The only trigger for reconstruction is context compression.

### 7-Layer Construction Order

```
① Agent Identity (SOUL.md)
   If SOUL.md exists → Read file content.
   If not → Use hardcoded DEFAULT_AGENT_IDENTITY (content is identical).

② Tool Behavior Guidance (Dynamically determined based on loaded tools)
   memory tool     → MEMORY_GUIDANCE (when/how to write memory correctly).
   session_search  → SESSION_SEARCH_GUIDANCE.
   skill_manage    → SKILLS_GUIDANCE.
   GPT/Gemini      → Additional execution discipline prompts.

③ Optional system_message from User/Gateway
   Custom business instructions from messaging platforms.

④ Memory Snapshot (Frozen)
   MEMORY.md → "## Your Memory\n..."
   USER.md   → "## User Profile\n..."

⑤ Skill Index
   Scan all SKILL.md files under ~/.hermes/skills/.
   Generate <available_skills> list.
   Two-layer cache: Memory LRU + Disk Snapshot (invalidated by mtime).

⑥ Project Context Files (Highest priority existing file)
   .hermes.md / AGENTS.md / CLAUDE.md / .cursorrules.
   Scanned for Prompt Injection before injection.
   Limit: 20,000 characters (head/tail truncation if exceeded).

⑦ Current Time + Runtime Environment
   "Conversation started: Monday, April 21, 2026 09:00 AM"
   Session ID, Model, Provider.
   Specific hints for WSL/Termux environments.
   Platform formatting hints (telegram/discord/cli/email/cron...).
```

### Creation of SOUL.md

`SOUL.md` is the "personality file" of the Agent, determining the content of Layer ①. Users can modify it freely.

**Three Creation Paths**:

| Trigger | Code Location |
|---------|---------|
| First run of any command, `ensure_hermes_home()` auto-generates it | `hermes_cli/config.py:291` |
| `hermes profile create` creates a new profile | `hermes_cli/profiles.py:453` |
| `hermes doctor --fix` fills it if missing | `hermes_cli/doctor.py:540` |

Changes to `~/.hermes/SOUL.md` take effect at the start of the **next session**.

### Prompt Cache Strategy (`agent/prompt_caching.py`)

Anthropic supports up to 4 cache breakpoints (`cache_control: {"type": "ephemeral"}`). The `"system_and_3"` strategy:

```
Breakpoint 1 → Entire System Prompt (first system content block)
Breakpoint 2 → 3rd from last non-system message
Breakpoint 3 → 2nd from last non-system message
Breakpoint 4 → Last non-system message (latest user message)
```

Implementation uses deep copies to avoid mutating original message lists. System prompt is frozen within a session → Breakpoint 1 always hits → **Saves ~90% cost** and **reduces ~50% latency**.

---

## 7. Frozen Snapshot Mode

Resolves the contradiction: "Memory files must be writable at any time, but the system prompt cannot change mid-session."

### How it Works

```
Session Starts
  │
  ▼
Read MEMORY.md + USER.md from disk
  │
  ▼
Generate Snapshot (_system_prompt_snapshot) ────── Frozen for the entire session
  │
  ▼
Injected into Layer ④ of the System Prompt

Mid-Session:
  Agent calls memory.add("xxx")
    ├─ Immediate write to disk (Persistence) ✓
    └─ Do NOT modify the current session's system prompt ✗

Next Session Starts:
  Re-read disk → New Snapshot → New System Prompt (Previous memory takes effect)
```

**Key Code** (`tools/memory_tool.py:136`):

```python
self._system_prompt_snapshot = {
    "memory": self._render_block("memory", self.memory_entries),
    "user":   self._render_block("user",   self.user_entries),
}
# Subsequent mid-session writes only update disk, not this dict.
```

---

## 8. Memory System

### Built-in Memory Files

| File | Content |
|------|------|
| `~/.hermes/memories/MEMORY.md` | Agent's notes (env facts, tool quirks, project conventions) |
| `~/.hermes/memories/USER.md` | User info (preferences, communication style, habits) |
| `~/.hermes/SOUL.md` | Agent identity/personality (injected into Layer ①) |

The Agent operates via the `memory` tool: `add`, `replace`, `remove`, `read`.

### External Memory Plugins

The `plugins/memory/` directory provides 8 external providers: **Honcho, Hindsight, Mem0, RetainDB, Byterover, Holographic, OpenViking, Supermemory**. Only one can be active at a time (co-existing with built-in memory).

**External Memory Recall (e.g., Honcho)**:
Before every turn, a background thread asynchronously fetches context and injects it into the current API request (not the system prompt). Recall results are wrapped in `<memory-context>` tags and are **transient**.

---

## 9. Context Compression

Triggered when the conversation length approaches the model's token limit. Implemented in `agent/context_compressor.py`.

### Compression Strategy
1. **Protect Head**: System prompt and first turn are never compressed.
2. **Protect Tail**: Recent N turns are kept (configurable token budget).
3. **Compress Middle**: Call LLM to generate a summary.
4. **Iterative Updates**: Subsequent triggers update the summary rather than replacing it.
5. **Tool Output Pruning**: Large tool results are deleted before compression.

---

## 10. Tools and Skills System

### Tools

40+ built-in tools. Each tool self-registers to `tools/registry.py` upon import.

**Web Search** (`tools/web_tools.py`):
Auto-detects 4 backends: **Firecrawl → Parallel → Tavily → Exa**.

### Skills

Skills are reusable workflows stored as Markdown files.
- `SKILL.md`: Frontmatter + instructions + examples.
- `skill.yaml`: Metadata (version, tags, dependencies).

---

## 11. Multi-Provider LLM Support

### Native Adapters
- `anthropic_adapter.py`: Supports Extended Thinking, Prompt Cache.
- `bedrock_adapter.py`: AWS credentials and regional routing.
- `gemini_native_adapter.py`: Native Chat API.
- `gemini_cloudcode_adapter.py`: Google Cloud code execution.

---

## 12. Sandboxing and Execution Environments

### Shell State Persistence
Uses **Session Snapshots**: Before and after every command, environment variables are exported (`export -p`) to `hermes-snap-{id}.sh` and the CWD is saved. The next command `source`s the snapshot to restore state.

### 6 Execution Backends
- `local`, `docker`, `ssh`, `modal`, `daytona`, `singularity`.

---

## 13. Security Mechanisms

### 13a. Dangerous Command Approval (`tools/approval.py`)
Intercepts commands based on **35 dangerous patterns** (e.g., `rm -rf`, `mkfs`, `git reset --hard`).

### 13b. Tirith Security Scanning
External binary for content-level threats (homograph URLs, pipe-to-interpreter, terminal injection).

### 13c. Path Security (`tools/path_security.py`)
Prevents traversal outside the workspace root using `path.resolve()`.

### 13d. Prompt Injection Protection
Scans context files for 10 injection patterns before injection.

### 13e. Credential Redaction (`agent/redact.py`)
Automatically filters 30+ credential types from logs and Cron output.

---

## 14. Multi-Platform Messaging Gateway

Supports 14+ platforms. Features a **DM Pairing Code** mechanism for secure initial connection.

---

## 15-18. System Operations

- **Storage**: SQLite with **WAL mode**. FTS5 for full-text search.
- **Cron**: Natural language expressions. Cron-mode Agents act autonomously and skip memory contamination.
- **MCP**: Acts as both an MCP server and an MCP client.

---

## 21. Key Design Patterns

- **Self-Registering Tools**: Decentralized tool discovery.
- **Provider Adapter Pattern**: Unified message format with provider-specific conversion.
- **Async-Sync Bridge**: Handles persistent event loops across threads.
- **Frozen Snapshot Mode**: Maximizes Prompt Cache hits (90% savings).
- **Plugin Hook System**: Extensible lifecycle events.

---

## 22. Statistical Summary

| Metric | Value |
|------|------|
| Built-in Tools | 40+ |
| Messaging Platforms | 14+ |
| LLM Providers | 20+ |
| Execution Backends | 6 |
| Memory Plugins | 8 |
| Dangerous Patterns | 35 |
| Credential Redactors| 30+ |
| Test Cases | ~3000 |
| License | MIT |

---
*Generated based on source code analysis of `run_agent.py`, `agent/`, `tools/`, `gateway/`, and `hermes_state.py`.*
