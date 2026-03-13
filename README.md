# neo-cortex-mcp

> **Early Preview** — This project is in active development. APIs and features may change. We welcome feedback via [Issues](https://github.com/LadislavSopko/neo-cortex-mcp/issues).

**Persistent memory for AI agents. Remember everything, forget nothing.**

neo-cortex-mcp gives [Claude Code](https://docs.anthropic.com/en/docs/claude-code) (or any MCP-compatible AI agent) a **long-term memory** that survives across sessions. Every conversation starts where the last one ended.

```
Session 1: "We decided to use PostgreSQL for the auth service"
   ...3 weeks later...
Session 47: "What database did we pick for auth?"
   → "PostgreSQL — decided in session 1, based on..."
```

## Why?

Claude Code is brilliant but **amnesiac**. Every session starts from zero. You re-explain your architecture, your decisions, your preferences — over and over.

neo-cortex fixes this. It:

- **Captures** every conversation automatically (via hooks)
- **Distills** knowledge into atomic memories using Gemini LLM
- **Embeds** each memory as a 1024-dim vector (Jina AI)
- **Retrieves** relevant context via semantic search + full-text search
- **Consolidates** similar memories over time (dream cycles)
- **Evolves** — memories gain energy when recalled, decay when forgotten

## Quick Start

### 1. Get API Keys (free tiers available)

| Service | Purpose | Get it at |
|---------|---------|-----------|
| [Jina AI](https://jina.ai/) | Embeddings (1024-dim vectors) | [jina.ai](https://jina.ai/) |
| [Google Gemini](https://ai.google.dev/) | Knowledge distillation + classification | [ai.google.dev](https://ai.google.dev/) |

### 2. Configure Claude Code

Add this to your project's `.mcp.json`:

```json
{
  "mcpServers": {
    "neo-cortex": {
      "command": "uvx",
      "args": ["--from", "neo-cortex-mcp@latest", "neo-cortex-mcp"],
      "env": {
        "JINA_API_KEY": "your-jina-key",
        "GEMINI_API_KEY": "your-gemini-key"
      }
    }
  }
}
```

That's it. Two API keys. Everything else is auto-configured.

### 3. Or let Claude Code do it all

Copy this prompt into a Claude Code session. It guides you through a 3-step install with 2 restarts:

<details>
<summary><strong>Click to expand auto-install prompt</strong></summary>

```
# Install neo-cortex-mcp — Persistent Memory for Claude Code
# This installation requires 3 steps with 2 restarts.
# After each step, tell the user what you did and ask them to restart.

## STEP 1 — Check prerequisites and create config files

1a. Check Python >= 3.12
   Run: python3 --version
   If missing or < 3.12: install via system package manager
   (apt: sudo apt install python3.12, brew: brew install python@3.12)

1b. Check uv (Python package manager)
   Run: uv --version
   If missing: curl -LsSf https://astral.sh/uv/install.sh | sh
   Verify uvx command works after install.

1c. Get API keys
   Ask the user for two API keys:
   - JINA_API_KEY — from https://jina.ai/ (free tier, for embeddings)
   - GEMINI_API_KEY — from https://ai.google.dev/ (free tier, for distillation)
   Show the URLs and ask the user to provide the keys.

1d. Create .mcp.json
   Create .mcp.json in the project root with:
   {
     "mcpServers": {
       "neo-cortex": {
         "command": "uvx",
         "args": ["--from", "neo-cortex-mcp@latest", "neo-cortex-mcp"],
         "env": {
           "JINA_API_KEY": "<key-from-1c>",
           "GEMINI_API_KEY": "<key-from-1c>"
         }
       }
     }
   }
   If .mcp.json already exists, merge the neo-cortex entry into it.

1e. Add to CLAUDE.md
   Add this block to the project's CLAUDE.md (create if missing):

   # Memory
   This project uses neo-cortex for persistent memory.
   ## Rules
   - ALWAYS use `memory_query` before saying "I don't know"
   - Use `memory_search` to filter by project or activity
   - Use `memory_timeline` at session start for recent context
   - Cortex is the primary source of truth for project history
   - Conversations are captured automatically via hooks

1f. Tell the user:
   "Done! I created .mcp.json and CLAUDE.md.
   Now please RESTART Claude Code so the MCP server can start.
   After restart, type: Check neo-cortex installation"

## STEP 2 — After first restart: verify MCP and hooks
(The user will restart and type "Check neo-cortex installation")

2a. Test MCP connection
   Run: memory_stats
   If it works: the MCP server is running.
   If it fails: check .mcp.json syntax and API keys.

2b. Check hooks were auto-installed
   Read .claude/settings.json — it should now contain
   SessionStart and Stop hooks (auto-installed by neo-cortex).
   If the hooks are there, tell the user:
   "MCP server is working! Hooks were auto-installed.
   Please RESTART Claude Code one more time to activate the hooks.
   After restart, type: Verify neo-cortex memory"

   If hooks are NOT there, something went wrong with bootstrap.
   Check that .mcp.json has the correct neo-cortex entry.

## STEP 3 — After second restart: everything is active
(The user will restart and type "Verify neo-cortex memory")

3a. Confirm hooks fired
   You should see cortex timeline output at startup (session context).
   Run memory_stats to confirm connection.

3b. Tell the user:
   "All set! neo-cortex memory is fully active.
   - Sessions are captured automatically when you exit
   - Knowledge is distilled in the background
   - Use memory_query to search past conversations
   - 11 MCP tools are available
   Your AI now remembers everything."
```

</details>

### 4. Restart Claude Code

On first launch, neo-cortex **auto-bootstraps**:
- Installs session hooks (start + stop) into `.claude/settings.json`
- Creates `cortex_db/` directory for storage
- Requires one restart to activate hooks

After restart, every session automatically:
1. **Starts** with context from your recent memories
2. **Captures** the conversation when you're done
3. **Distills** knowledge in the background

### Session Hooks (auto-installed)

neo-cortex installs two hooks into your project's `.claude/settings.json`:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "uvx --from neo-cortex-mcp@latest neo-cortex-timeline --n 10",
            "timeout": 15
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "*",
        "hooks": [
          {
            "type": "command",
            "command": "uvx --from neo-cortex-mcp@latest neo-cortex-stop-hook",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

| Hook | What it does |
|------|-------------|
| **SessionStart** (`neo-cortex-timeline`) | Injects [MBEL v5 grammar](https://github.com/LadislavSopko/neo-cortex-mcp/blob/main/docs/mbel-v5.md) + last 10 memories into session context |
| **Stop** (`neo-cortex-stop-hook`) | Captures conversation turns into `conversation_log.db` for background distillation |

> **Note:** These hooks are installed automatically by the MCP server on first launch. If you already have hooks in `.claude/settings.json`, neo-cortex merges — it never overwrites existing hooks.

## What You Get

### 12 MCP Tools

These appear automatically in Claude Code:

| Tool | What it does |
|------|-------------|
| `memory_query` | Semantic search — "what did we decide about auth?" |
| `memory_search` | Filter by project, activity, text |
| `memory_timeline` | Recent memories, chronological |
| `memory_stats` | Total memories, sessions, energy levels |
| `memory_get` | Full details of specific memories by ID |
| `memory_ingest` | Manually store a memory |
| `memory_dream` | Run consolidation cycle (boost/decay) |
| `memory_spontaneous` | Rediscover forgotten high-energy memories |
| `memory_rebuild` | Wipe and rebuild from conversation log |
| `memory_identity_review` | Self-reflection — propose identity insights |
| `memory_dashboard` | Web UI for browsing memories + concept graph |
| `memory_graph` | Navigate the concept graph — top concepts, neighbors, paths, neighborhoods |

### Web Dashboard

A built-in web UI for exploring your memory:

- **3D concept graph** — interactive visualization of knowledge connections
- **Memory browser** — search, filter, view details
- **Energy distribution** — see which memories are strong/fading
- **Maintenance** — trigger dream cycles, rebuilds

Access it via `memory_dashboard` tool or directly at the dynamic port.

### Intelligent Memory Pipeline

```
Conversation → Stop Hook → Conversation Log
                              ↓
                    Gemini Distiller (extracts atomic facts)
                              ↓
                    Jina Embedder (1024-dim vectors)
                              ↓
              ChromaDB (vectors) + SQLite FTS5 (text search)
                              ↓
                    Concept Graph (NetworkX)
                              ↓
                  Idle Consolidator (merges similar)
```

**Key behaviors:**
- **Dual retrieval**: Vector similarity + full-text search, fused with Reciprocal Rank Fusion (RRF)
- **Concept graph**: NetworkX tracks relationships between ideas — navigate with `memory_graph`
- **Deduplication**: SimHash (64-bit, Hamming distance ≤ 3) prevents storing the same fact twice
- **Energy model**: Memories gain energy when recalled, decay over execution days (Ebbinghaus curve)
- **Noise filtering**: 7-category filter prevents storing ephemeral junk (test output, file listings, etc.)
- **Observation levels**: Direct, Inferred, Abstract, Reported, Metacognitive

## What Happens Automatically

neo-cortex handles three things without you doing anything:

1. **Session Start** — The hook injects the [MBEL v5 grammar](https://github.com/LadislavSopko/neo-cortex-mcp/blob/main/docs/mbel-v5.md) (a compression language with 27 operators) + a compact summary of your recent memories. Claude learns to read and write MBEL automatically.

2. **During Session** — The 11 MCP tools are available. Claude can query memories, search, view stats. Memory results come back in MBEL format (75% compression, 100% fidelity).

3. **Session End** — The stop hook captures the full conversation, queues it for background distillation.

**You don't need to teach Claude the MBEL grammar** — it's injected automatically at every session start.

## Add to Your CLAUDE.md

What you *should* add to your `CLAUDE.md` is behavioral guidance — telling Claude *when* and *how* to use its memory:

```markdown
# Memory

This project uses neo-cortex for persistent memory across sessions.

## Rules
- ALWAYS use `memory_query` before saying "I don't know" about past work
- Use `memory_search` to filter by project or activity type
- Use `memory_timeline` at session start to load recent context
- DO NOT ingest noise — only store genuinely useful knowledge via `memory_ingest`
- The cortex is the primary source of truth for project history and decisions

## How It Works
- Conversations are captured automatically via hooks (you don't need to save manually)
- Knowledge is distilled in the background by Gemini LLM
- Memories have energy (0.0-1.0) — recalled memories get stronger, unused ones fade
- Dream cycles consolidate and prune memories periodically
- The concept graph tracks relationships between ideas
- MBEL grammar is injected automatically at session start — you can read/write it natively
```

> **Tip:** The more specific your `CLAUDE.md` is about your project's context, the better Claude will use its memory. For example: "When debugging auth issues, always check cortex for past auth decisions."

## Configuration

### Required

| Variable | Description |
|----------|-------------|
| `JINA_API_KEY` | Jina AI embeddings API key |
| `GEMINI_API_KEY` | Google Gemini API key |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `CORTEX_DATA_DIR` | current directory | Root directory (creates `cortex_db/` inside) |
| `CLASSIFIER_PROVIDER` | `gemini` | LLM for query analysis (`gemini` or `groq`) |
| `DIGESTOR_MODE` | `distill` | Knowledge extraction mode |
| `DIGESTOR_BUDGET` | `32000` | Token budget per digestor batch |
| `CONSOLIDATION_SIMILARITY_THRESHOLD` | `0.82` | Cosine threshold for merging memories |

## Requirements

- **Python** >= 3.12
- **uv** (install from [astral.sh](https://docs.astral.sh/uv/getting-started/installation/))
- **Claude Code** with MCP support

## How It Stores Data

Everything lives in `cortex_db/` inside your project:

```
cortex_db/
├── chroma.sqlite3          # Vector embeddings (ChromaDB)
├── memory_index.db         # Full-text search index (SQLite FTS5)
├── conversation_log.db     # Raw conversation turns
├── concept_graph.json      # Knowledge graph (NetworkX)
├── dedup.db                # Duplicate detection hashes
└── neo-cortex.log          # Operation log
```

All SQLite. No external databases. No Docker. No network services.

## Debugging

Add `--debug` to hook commands for verbose logging:

```json
{
  "hooks": {
    "SessionStart": [{
      "command": "uvx --from neo-cortex-mcp@latest neo-cortex-timeline --n 10 --debug"
    }]
  }
}
```

Logs go to `cortex_db/neo-cortex.log`.

## FAQ

**Q: Does it work with other AI agents besides Claude Code?**
A: Any agent that supports MCP (Model Context Protocol) can use neo-cortex. The hooks are Claude Code specific, but the MCP tools work universally.

**Q: How much does it cost?**
A: The package is free. API costs depend on usage — both Jina AI and Gemini offer generous free tiers that cover normal development use.

**Q: Where is my data stored?**
A: Locally, in `cortex_db/` inside your project. Nothing leaves your machine except API calls to Jina (embeddings) and Gemini (distillation). No telemetry, no cloud storage.

**Q: Can I use it across multiple projects?**
A: Yes. Set `CORTEX_DATA_DIR` to a shared directory, or use separate `cortex_db/` per project.

**Q: How many memories can it handle?**
A: Tested with 900+ memories and a concept graph of 950+ nodes. ChromaDB and SQLite scale well beyond that.

**Q: What if I want to start fresh?**
A: Use the `memory_rebuild` tool — it wipes all memories and rebuilds from the conversation log.

## Status

**Current version: 5.21.0** (Early Preview)

This is an early preview release. The core functionality is stable and battle-tested in daily use, but:

- APIs may change between versions
- Documentation is evolving
- Some features are experimental (meta-reflection, identity review)

We welcome bug reports and feature requests via [Issues](https://github.com/LadislavSopko/neo-cortex-mcp/issues).

## Community

- **Issues**: [Report bugs, request features](https://github.com/LadislavSopko/neo-cortex-mcp/issues)
- **Discussions**: Use Issues with the `discussion` label
- **PyPI**: [neo-cortex-mcp](https://pypi.org/project/neo-cortex-mcp/)

## License

[CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — Free for non-commercial use. Commercial use requires a separate license. Contact: ladislav.sopko@gmail.com
