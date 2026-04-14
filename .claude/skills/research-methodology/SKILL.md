---
name: research-methodology
description: >-
  Structured research protocol for investigating technologies, patterns, and APIs
  before implementation. Use when exploring unfamiliar technologies, evaluating
  library options, or documenting technical findings. Covers multi-tool search
  strategy, research log conventions, and training data fallback honesty protocol.
---

# Research Methodology Skill

## Purpose

Ensure that code is informed by accurate, up-to-date knowledge — not stale training data assumptions. This skill provides a structured process for researching technologies, APIs, and patterns before writing code.

## When to Invoke

- Before implementing features using unfamiliar technologies
- When evaluating library or framework options
- When the Architect needs to inform design decisions with current best practices
- When any builder agent encounters an API or pattern they haven't used before

## Research Process

### 1. Topic Decomposition

Break the research need into specific, searchable topics (2-5 keywords each):

```
Example — "Task CRUD API with Supabase":
  Topics:
  1. Supabase client library JavaScript/TypeScript
  2. Supabase Row Level Security policies
  3. Supabase real-time subscriptions
  4. PostgreSQL UUID primary key patterns
  5. TypeScript type generation from Supabase
```

### 2. Multi-Tool Search Strategy

Search for each topic using the best available tool, in priority order:

| Priority | Tool | When Available | Best For |
|---|---|---|---|
| 1st | **Qurio** (`mcp_qurio_qurio_search`, `mcp_qurio_qurio_read_page`) | Qurio MCP is enabled | Deep documentation search, official docs |
| 2nd | **Context7** or similar MCP documentation servers | MCP server available | Library-specific API references |
| 3rd | **Supabase docs** (`mcp_supabase_search_docs`) | Supabase MCP enabled | Supabase-specific documentation |
| 4th | **`search_web`** | Always available | General queries, blog posts, Stack Overflow |
| 5th | **`read_url_content`** | Always available | Deep reading of specific documentation pages |

**Search strategy:**
```
# Start broad, then narrow:
search("Supabase RLS policies")           # Broad — find the right doc page
read_page("https://supabase.com/docs/...") # Deep — read the specific page

# Search for gotchas and edge cases:
search("Supabase RLS common mistakes")
search("Supabase RLS performance implications")
```

### 3. Document Findings

Create a research log for each feature or investigation:

**Path:** `docs/research_logs/{feature_name}.md`

**Naming convention:**
```
docs/research_logs/
├── epic1_auth.md           # Epic 1: Authentication
├── epic2_task_crud.md       # Epic 2: Task CRUD
├── supabase_rls_evaluation.md  # Technology evaluation
└── {feature_name}.md       # Pattern: one log per feature
```

**Research log template:**
```markdown
# Research: {Feature or Topic Name}
Date: {date}
Researcher: {agent name}

## Topics Investigated
1. {topic} — {search tool used} — {key finding}
2. {topic} — {search tool used} — {key finding}

## Key Patterns Discovered
- {Pattern name}: {description with code example}

## API Signatures & Options
```{language}
// Exact API signatures from documentation
```

## Gotchas & Edge Cases
- {Gotcha}: {how to avoid it}

## Code Examples
```{language}
// Working examples from documentation
```

## Sources
- [{doc title}]({url}) — {what was learned}

## Reliance on Training Data
<!-- Fill this section if any topic could not be verified externally -->
- {topic}: Relying on training data. No external verification available.
```

### 4. Training Data Fallback Protocol

If no documentation search tool yields results for a topic:

1. **Document what was searched** — list the queries attempted and tools used
2. **Explicitly state reliance:**
   > "I am relying on my training data for {topic}. External verification was unavailable."
3. **Proceed with caution** — flag the implementation for human review
4. **Prioritize verification** — if the feature is critical, ask the user for documentation links

**Never silently use training data when external verification is available.** The training data cutoff means APIs may have changed, libraries may have deprecated features, and patterns may have evolved.

### 5. Architecture Decision Records

If research reveals a decision involving:
- Choosing between 2+ viable approaches
- Introducing a new dependency or pattern
- Changing existing architecture

Then create an ADR using the **ADR Skill** at `docs/decisions/NNNN-short-title.md`.

## Integration with Agent Workflow

| Agent | How They Use This Skill |
|---|---|
| **Architect** | Research before design decisions, inform ADRs |
| **Backend Engineer** | Research unfamiliar APIs, library patterns |
| **Frontend Engineer** | Research component libraries, CSS patterns |
| **Mobile Engineer** | Research platform APIs, Flutter packages |
| **Database Expert** | Research query patterns, extension capabilities |

## Rule Compliance
- This skill supports the research phase of all builder agents
- Research logs persist in `docs/research_logs/` for cross-session knowledge
- ADRs follow the format defined in the `adr` skill
