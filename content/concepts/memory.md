+++
title = "Memory System"
description = "How Kage stores and shares context between agents."
weight = 3
+++

# Memory System

Kage uses a two-tier memory system to store and share context between agents.

## Two-Tier Architecture

### Working Memory

- In-memory, fast access
- Current session context
- Cleared on agent termination

### Long-Term Memory

- Persisted to disk (append-only logs)
- Discoveries and learned patterns
- Survives restarts

## Memory Types

Kage stores different types of memories:

```rust
pub enum MemoryEntry {
    // Code understanding
    FileDiscovered { path, summary, structure },
    PatternLearned { pattern, examples, confidence },
    DependencyMapped { from, to, relationship },

    // Task context
    ErrorEncountered { error, resolution, worked },
    DecisionMade { decision, reasoning, alternatives },
    TaskCompleted { task_id, summary, artifacts },

    // Collaboration
    InsightShared { topic, content, from_agent },
    QuestionAsked { question, answer, from_agent },
}
```

## Memory Scopes

Memories can be scoped to different levels:

- `agent` - Private to one agent
- `namespace` - Shared within namespace
- `global` - Available to all agents (opt-in)

## Context Injection

Inject context from one agent to another:

```bash
kage context inject <target-agent> --from <source-agent>
```

Filter by type or topic:

```bash
kage context inject agent-2 --from agent-1 --type patterns
kage context inject agent-2 --from agent-1 --topic "auth"
```
