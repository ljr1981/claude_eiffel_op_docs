# Task Manager AI-RAG Implementation Plan
## Version 1.0 - December 2025

---

## Executive Summary

Transform the basic task manager into an AI-enhanced productivity system with:
- Rich task data model (descriptions, due dates, subtasks, CPM dependencies)
- Multiple AI provider support (Claude, Grok, Gemini, Ollama)
- RAG-based intelligent features (smart search, task decomposition, NL input)
- **Works WITHOUT AI as default baseline** (AI is optional enhancement)
- **CPM-style dependency management** with AI-assisted block resolution
- **Provider-specific prompt engineering** (narrow prompts for Ollama, richer for Claude)

---

## Existing Libraries

| Library | What We Use |
|---------|------------|
| `simple_ai_client` | AI_CLIENT, CLAUDE_CLIENT, GROK_CLIENT, OLLAMA_CLIENT, AI_EMBEDDING_STORE |
| `simple_kb` | RAG patterns (KB_AI_ROUTER, KB_AI_CONFIG as templates) |
| `simple_sql` | TODO_ITEM, TODO_REPOSITORY (to extend) |
| `simple_tui` | TUI_QUICK, widgets (to extend) |

---

## Implementation Phases

### Phase 1: Data Model Enhancement
**Location:** `simple_sql/src/todo_app/`

- [x] **1.1** Extend TODO_ITEM with new fields:
  - `parent_id` (INTEGER) - self-reference for subtasks
  - `status` (STRING) - "pending", "in_progress", "waiting", "completed", "archived"
  - `context` (STRING) - "office", "home", "phone", "errands", or custom
  - `energy_level` (INTEGER 1-3) - focus required
  - `estimated_minutes` (INTEGER) - time estimate
  - `actual_minutes` (INTEGER) - time tracked

- [x] **1.2** Create TASK_DEPENDENCY class and table:
  ```sql
  CREATE TABLE task_dependency (
      id INTEGER PRIMARY KEY,
      predecessor_id INTEGER NOT NULL,
      successor_id INTEGER NOT NULL,
      dependency_type TEXT DEFAULT 'finish_to_start',
      FOREIGN KEY (predecessor_id) REFERENCES todo_item(id),
      FOREIGN KEY (successor_id) REFERENCES todo_item(id)
  );
  ```
  - Types: finish_to_start, start_to_start, finish_to_finish, start_to_finish

- [x] **1.3** Create TASK_TAG class and table:
  ```sql
  CREATE TABLE task_tag (
      task_id INTEGER NOT NULL,
      tag TEXT NOT NULL,
      PRIMARY KEY (task_id, tag),
      FOREIGN KEY (task_id) REFERENCES todo_item(id)
  );
  ```

- [x] **1.4** Extend TODO_REPOSITORY with new queries:
  - `find_children (parent_id)` - subtasks
  - `find_by_status (status)` - filter by status
  - `find_by_context (context)` - filter by context
  - `find_blocked` - tasks with incomplete predecessors
  - `find_blocking (task_id)` - what this task blocks
  - `find_predecessors (task_id)` - what blocks this task

---

### Phase 2: AI Provider Configuration
**Location:** `simple_tui/src/apps/task_manager/ai/`

- [x] **2.1** Create TASK_AI_CONFIG (modeled after KB_AI_CONFIG):
  - Provider selection: claude, grok, ollama, none
  - API key storage (per provider)
  - Model selection per provider
  - `is_ready: BOOLEAN` - is AI configured and available?
  - Persistence to `~/.task_manager/ai_config.json`

- [x] **2.2** Create TASK_PROMPT_TEMPLATES:
  - Provider-specific prompt engineering
  - Ollama: narrow, focused, single-task prompts
  - Claude/Grok: richer, multi-step prompts
  - Templates for: parse_task, suggest_subtasks, resolve_block, prioritize

---

### Phase 3: RAG Engine (TASK_AI_ROUTER)
**Location:** `simple_tui/src/apps/task_manager/ai/`

- [x] **3.1** Create TASK_AI_ROUTER (modeled after KB_AI_ROUTER):
  - 4-phase cascade with graceful no-AI fallback
  - Phase 1: Parse natural language input
  - Phase 2: Search similar past tasks
  - Phase 3: Synthesize from history if found
  - Phase 4: Raw AI generation + cache result

- [ ] **3.2** Create TASK_EMBEDDING_STORE:
  - Embed task title + description
  - Find similar tasks by semantic search
  - Track task patterns (what subtasks were created before)

- [ ] **3.3** Implement core AI features:
  - `parse_natural_input (text) -> TODO_ITEM` - NL to structured task
  - `suggest_subtasks (task) -> LIST[TODO_ITEM]` - decomposition
  - `find_similar_tasks (task) -> LIST[TODO_ITEM]` - history lookup
  - `suggest_estimate (task) -> INTEGER` - time estimate from history

---

### Phase 4: CPM/Dependency Features
**Location:** `simple_tui/src/apps/task_manager/`

- [ ] **4.1** Create TASK_CPM_ANALYZER:
  - `is_blocked (task) -> BOOLEAN`
  - `blocking_tasks (task) -> LIST[TODO_ITEM]`
  - `critical_path -> LIST[TODO_ITEM]`
  - `earliest_start (task) -> DATE`
  - `slack_time (task) -> INTEGER`

- [ ] **4.2** AI-Assisted Block Resolution:
  - When task is blocked, AI suggests:
    1. Alternative tasks to work on now
    2. Ways to unblock (parallel work, delegation)
    3. Rescheduling recommendations
  - Provider-specific prompts for block resolution

---

### Phase 5: TUI Enhancements
**Location:** `simple_tui/src/`

- [x] **5.1** Create TUI_INPUT_DIALOG widget:
  - Multi-field modal dialog
  - Text fields, text areas, selectors, date pickers
  - Tab navigation between fields
  - Submit/Cancel actions

- [x] **5.2** Create Task Creation Dialog:
  - Title (required)
  - Description (multi-line)
  - Priority (1-5 selector)
  - Due date
  - Context selector
  - Energy level selector
  - Parent task selector (for subtasks)

- [x] **5.3** Create Task Edit Dialog:
  - All creation fields
  - Status change
  - Add/remove dependencies
  - Time tracking (start/stop timer)

- [ ] **5.4** Enhanced List Display:
  - Indented subtasks
  - Status icons
  - Blocked indicator
  - Due date coloring (overdue=red, today=yellow)

---

### Phase 6: AI Integration in UI
**Location:** `simple_tui/src/apps/task_manager/`

- [x] **6.1** Add AI menu options:
  - "AI: Create from text..." (natural language)
  - "AI: Suggest subtasks" (for selected task)
  - "AI: What can I work on?" (unblocked, prioritized)
  - "AI: Help with blocked task"
  - "AI: Configure..." (provider selection)

- [x] **6.2** Implement graceful degradation:
  - All features work without AI
  - AI enhances but doesn't require
  - Clear indicators when AI is on/off
  - Error handling for AI failures

---

### Phase 7: Settings & Persistence
**Location:** `simple_tui/src/apps/task_manager/`

- [ ] **7.1** Create TASK_MANAGER_CONFIG:
  - AI settings (from TASK_AI_CONFIG)
  - Default context
  - Default energy level
  - UI preferences

- [ ] **7.2** Config file: `~/.task_manager/config.json`
  - Load on startup
  - Save on change
  - Encrypt API keys

---

## File Structure

```
simple_sql/src/todo_app/
├── todo_item.e              # EXTEND with new fields
├── todo_repository.e        # EXTEND with new queries
├── task_dependency.e        # NEW
└── task_tag.e               # NEW

simple_tui/src/apps/task_manager/
├── task_manager_app.e       # ENHANCE with AI
├── task_manager_config.e    # NEW
├── task_cpm_analyzer.e      # NEW
└── ai/
    ├── task_ai_config.e     # NEW
    ├── task_ai_router.e     # NEW
    ├── task_prompt_templates.e  # NEW
    └── task_embedding_store.e   # NEW

simple_tui/src/widgets/
└── tui_input_dialog.e       # NEW
```

---

## Provider-Specific Prompt Engineering

### Ollama (Local, Limited)
- Single-purpose prompts
- Explicit format requirements
- No multi-step reasoning
- Example:
  ```
  Extract task title and priority from this text.
  Format: TITLE: <title> | PRIORITY: <1-5>
  Text: "call bob tomorrow about the contract, it's urgent"
  ```

### Claude/Grok (API, Capable)
- Rich context allowed
- Multi-step reasoning
- Structured JSON output
- Example:
  ```
  Parse this natural language into a structured task.
  Consider: title, description, priority (1-5), due date, context.
  Return JSON: {"title": "...", "priority": N, ...}
  Input: "call bob tomorrow about the contract, it's urgent"
  ```

---

## Non-AI Baseline Features

When AI is disabled or unavailable:

| Feature | AI Mode | Non-AI Mode |
|---------|---------|-------------|
| Task creation | NL parsing | Manual form entry |
| Subtask suggestions | AI-generated | Manual creation |
| Similar task search | Semantic search | Keyword search |
| Block resolution | AI suggestions | Show blocking tasks only |
| Time estimates | History-based AI | Manual entry |
| Prioritization | AI-ranked | Manual sort |

---

## Success Criteria

1. [x] Task manager works fully without AI configured
2. [ ] User can select AI provider from UI
3. [x] Natural language task creation works
4. [x] Subtask suggestions work
5. [ ] CPM dependencies track correctly
6. [x] Blocked tasks show resolution suggestions
7. [x] All data persists to SQLite
8. [x] TUI remains responsive during AI calls

---

## Dependencies

```
simple_tui (task_manager target)
└── simple_ai_client
└── simple_sql
└── simple_logger
```

---

## Notes

- Start with non-AI features working first
- Add AI as enhancement layer
- Test each provider independently
- Cache AI responses to reduce API calls
- Log all AI interactions for debugging
