# Application JSON Specification Protocol v1.0

A comprehensive, machine-readable format for defining complete applications from GUI to persistence. Provides top-down, bottom-up, and middle-out views of the entire system.

## Design Philosophy

1. **Single Source of Truth**: One spec defines the entire application
2. **Full Stack**: GUI → Business Logic → Data → Persistence
3. **Bidirectional Traceability**: Every UI element traces to data, every data field traces to UI
4. **Testable at Every Layer**: Use cases, unit tests, integration tests, persistence tests
5. **Generatable**: Produces complete, working Eiffel code

---

## Schema Overview

```json
{
  "app": { },              // Application metadata
  "architecture": { },     // Layer definitions and dependencies

  "// TOP-DOWN (User → System)": "",
  "windows": [ ],          // GUI windows and dialogs
  "widgets": { },          // Reusable widget definitions
  "state_machines": [ ],   // UI state management
  "user_flows": [ ],       // End-to-end user journeys

  "// MIDDLE (Business Logic)": "",
  "use_cases": [ ],        // Business use cases with full logic
  "services": [ ],         // Service layer contracts
  "domain_rules": [ ],     // Business rules and validations
  "workflows": [ ],        // Multi-step business processes

  "// BOTTOM-UP (Data → User)": "",
  "entities": [ ],         // Domain entities with full schema
  "repositories": [ ],     // Data access patterns
  "data_sources": [ ],     // External data connections

  "// PERSISTENCE": "",
  "persistence": {
    "tables": [ ],         // Database tables
    "migrations": [ ],     // Schema migrations
    "queries": [ ],        // Named queries
    "indexes": [ ]         // Performance indexes
  },

  "// CROSS-CUTTING": "",
  "events": [ ],           // Application events
  "errors": [ ],           // Error definitions
  "security": { },         // Permissions and auth
  "config": { }            // Runtime configuration
}
```

---

# SECTION 1: APPLICATION METADATA

```json
{
  "app": {
    "name": "speech_studio",
    "title": "Speech Studio",
    "version": "1.0.0",
    "description": "GUI front-end for simple_speech TTS engine",
    "author": "Larry Rix",
    "license": "MIT",

    "libraries": {
      "gui": "simple_vision",
      "speech": "simple_speech",
      "data": "simple_repository",
      "storage": "simple_sql"
    },

    "entry_point": {
      "window": "main_window",
      "on_start": ["load_config", "initialize_speech_engine", "load_voices"]
    }
  }
}
```

---

# SECTION 2: ARCHITECTURE

Defines the layers and their relationships.

```json
{
  "architecture": {
    "layers": [
      {
        "name": "presentation",
        "description": "GUI layer - windows, widgets, user interaction",
        "depends_on": ["application"],
        "technologies": ["simple_vision", "EiffelVision2"]
      },
      {
        "name": "application",
        "description": "Use cases, workflows, orchestration",
        "depends_on": ["domain", "infrastructure"],
        "technologies": []
      },
      {
        "name": "domain",
        "description": "Business entities, rules, domain logic",
        "depends_on": [],
        "technologies": []
      },
      {
        "name": "infrastructure",
        "description": "Repositories, external services, persistence",
        "depends_on": ["domain"],
        "technologies": ["simple_repository", "simple_sql", "simple_speech"]
      }
    ],

    "modules": [
      {
        "name": "speech",
        "description": "Text-to-speech functionality",
        "layers": {
          "presentation": ["speech_panel", "voice_selector", "playback_controls"],
          "application": ["speak_use_case", "voice_management_use_case"],
          "domain": ["speech_request", "voice", "speech_history"],
          "infrastructure": ["speech_engine_adapter", "speech_history_repository"]
        }
      },
      {
        "name": "settings",
        "description": "User preferences and configuration",
        "layers": {
          "presentation": ["settings_dialog"],
          "application": ["settings_use_case"],
          "domain": ["user_settings"],
          "infrastructure": ["settings_repository"]
        }
      }
    ]
  }
}
```

---

# SECTION 3: ENTITIES (Domain Layer)

Complete domain model with relationships, validations, and behaviors.

```json
{
  "entities": [
    {
      "name": "SpeechRequest",
      "description": "A request to synthesize speech from text",
      "table": "speech_requests",

      "fields": [
        {
          "name": "id",
          "type": "uuid",
          "primary_key": true,
          "generated": true
        },
        {
          "name": "text",
          "type": "string",
          "max_length": 10000,
          "required": true,
          "validations": [
            { "type": "not_empty", "message": "Text is required" },
            { "type": "max_length", "value": 10000, "message": "Text too long (max 10000 chars)" }
          ],
          "ui_hints": {
            "widget": "text_area",
            "label": "Text to Speak",
            "placeholder": "Enter text here...",
            "rows": 5
          }
        },
        {
          "name": "voice_id",
          "type": "string",
          "required": true,
          "references": { "entity": "Voice", "field": "id" },
          "ui_hints": {
            "widget": "dropdown",
            "label": "Voice",
            "options_source": "available_voices"
          }
        },
        {
          "name": "rate",
          "type": "integer",
          "default": 0,
          "range": { "min": -10, "max": 10 },
          "ui_hints": {
            "widget": "slider",
            "label": "Speed",
            "show_value": true
          }
        },
        {
          "name": "volume",
          "type": "integer",
          "default": 100,
          "range": { "min": 0, "max": 100 },
          "ui_hints": {
            "widget": "slider",
            "label": "Volume",
            "suffix": "%"
          }
        },
        {
          "name": "pitch",
          "type": "integer",
          "default": 0,
          "range": { "min": -10, "max": 10 },
          "ui_hints": {
            "widget": "slider",
            "label": "Pitch"
          }
        },
        {
          "name": "created_at",
          "type": "datetime",
          "default": "now",
          "immutable": true
        },
        {
          "name": "status",
          "type": "enum",
          "values": ["pending", "speaking", "completed", "failed", "cancelled"],
          "default": "pending"
        },
        {
          "name": "error_message",
          "type": "string",
          "nullable": true
        },
        {
          "name": "audio_file_path",
          "type": "string",
          "nullable": true,
          "description": "Path to saved audio file, if requested"
        }
      ],

      "computed_fields": [
        {
          "name": "word_count",
          "type": "integer",
          "expression": "text.split(' ').count"
        },
        {
          "name": "estimated_duration_seconds",
          "type": "real",
          "expression": "word_count * 0.4 * (1.0 - rate * 0.05)"
        }
      ],

      "behaviors": {
        "can_cancel": {
          "condition": "status IN ['pending', 'speaking']"
        },
        "can_replay": {
          "condition": "status = 'completed'"
        }
      },

      "invariants": [
        "text.count > 0",
        "rate >= -10 AND rate <= 10",
        "volume >= 0 AND volume <= 100"
      ]
    },

    {
      "name": "Voice",
      "description": "An available TTS voice",
      "table": null,
      "source": "speech_engine",

      "fields": [
        { "name": "id", "type": "string", "primary_key": true },
        { "name": "name", "type": "string" },
        { "name": "language", "type": "string" },
        { "name": "gender", "type": "enum", "values": ["male", "female", "neutral"] },
        { "name": "age", "type": "enum", "values": ["child", "teen", "adult", "senior"] }
      ],

      "queries": {
        "by_language": "language = ?",
        "by_gender": "gender = ?"
      }
    },

    {
      "name": "SpeechHistory",
      "description": "Record of past speech requests",
      "table": "speech_history",

      "fields": [
        { "name": "id", "type": "integer", "primary_key": true, "auto_increment": true },
        { "name": "text", "type": "string", "max_length": 10000 },
        { "name": "voice_name", "type": "string" },
        { "name": "rate", "type": "integer" },
        { "name": "volume", "type": "integer" },
        { "name": "created_at", "type": "datetime" },
        { "name": "duration_ms", "type": "integer" },
        { "name": "saved_audio_path", "type": "string", "nullable": true }
      ],

      "ui_hints": {
        "list_columns": ["text", "voice_name", "created_at"],
        "sort_default": { "field": "created_at", "order": "desc" }
      }
    },

    {
      "name": "UserSettings",
      "description": "User preferences",
      "table": "user_settings",
      "singleton": true,

      "fields": [
        { "name": "default_voice_id", "type": "string", "nullable": true },
        { "name": "default_rate", "type": "integer", "default": 0 },
        { "name": "default_volume", "type": "integer", "default": 100 },
        { "name": "auto_save_audio", "type": "boolean", "default": false },
        { "name": "audio_output_folder", "type": "string", "default": "./audio" },
        { "name": "audio_format", "type": "enum", "values": ["wav", "mp3"], "default": "wav" },
        { "name": "keep_history", "type": "boolean", "default": true },
        { "name": "max_history_items", "type": "integer", "default": 100 },
        { "name": "theme", "type": "enum", "values": ["light", "dark", "system"], "default": "system" }
      ]
    }
  ]
}
```

---

# SECTION 4: REPOSITORIES (Data Access Layer)

Define data access patterns for each entity.

```json
{
  "repositories": [
    {
      "name": "SpeechHistoryRepository",
      "entity": "SpeechHistory",
      "storage": "sqlite",
      "table": "speech_history",

      "operations": {
        "create": {
          "type": "insert",
          "returns": "SpeechHistory"
        },
        "find_by_id": {
          "type": "select_one",
          "params": ["id"],
          "query": "SELECT * FROM speech_history WHERE id = ?"
        },
        "find_recent": {
          "type": "select_many",
          "params": ["limit"],
          "query": "SELECT * FROM speech_history ORDER BY created_at DESC LIMIT ?",
          "default_params": { "limit": 50 }
        },
        "search_text": {
          "type": "select_many",
          "params": ["search_term"],
          "query": "SELECT * FROM speech_history WHERE text LIKE '%' || ? || '%' ORDER BY created_at DESC"
        },
        "delete_by_id": {
          "type": "delete",
          "params": ["id"],
          "query": "DELETE FROM speech_history WHERE id = ?"
        },
        "delete_older_than": {
          "type": "delete",
          "params": ["days"],
          "query": "DELETE FROM speech_history WHERE created_at < datetime('now', '-' || ? || ' days')"
        },
        "count": {
          "type": "scalar",
          "query": "SELECT COUNT(*) FROM speech_history"
        },
        "prune_to_limit": {
          "type": "delete",
          "params": ["max_items"],
          "query": "DELETE FROM speech_history WHERE id NOT IN (SELECT id FROM speech_history ORDER BY created_at DESC LIMIT ?)"
        }
      },

      "indexes": [
        { "name": "idx_history_created", "columns": ["created_at"], "order": "desc" },
        { "name": "idx_history_text", "columns": ["text"], "type": "fulltext" }
      ]
    },

    {
      "name": "UserSettingsRepository",
      "entity": "UserSettings",
      "storage": "sqlite",
      "table": "user_settings",
      "singleton": true,

      "operations": {
        "load": {
          "type": "select_one",
          "query": "SELECT * FROM user_settings LIMIT 1",
          "create_if_missing": true
        },
        "save": {
          "type": "upsert",
          "description": "Insert or update the singleton settings row"
        }
      }
    },

    {
      "name": "VoiceRepository",
      "entity": "Voice",
      "storage": "memory",
      "source": "speech_engine.get_all_voices()",

      "operations": {
        "get_all": {
          "type": "method",
          "call": "speech_engine.get_all_voices"
        },
        "find_by_id": {
          "type": "filter",
          "predicate": "voice.id = ?"
        },
        "find_by_language": {
          "type": "filter",
          "predicate": "voice.language = ?"
        }
      },

      "caching": {
        "enabled": true,
        "ttl_seconds": 3600,
        "invalidate_on": ["speech_engine_changed"]
      }
    }
  ]
}
```

---

# SECTION 5: PERSISTENCE

Database schema, migrations, and named queries.

```json
{
  "persistence": {
    "database": {
      "type": "sqlite",
      "path": "${app_data}/speech_studio.db",
      "version": 1
    },

    "tables": [
      {
        "name": "speech_history",
        "columns": [
          { "name": "id", "type": "INTEGER", "primary_key": true, "auto_increment": true },
          { "name": "text", "type": "TEXT", "not_null": true },
          { "name": "voice_name", "type": "TEXT", "not_null": true },
          { "name": "rate", "type": "INTEGER", "default": 0 },
          { "name": "volume", "type": "INTEGER", "default": 100 },
          { "name": "created_at", "type": "DATETIME", "default": "CURRENT_TIMESTAMP" },
          { "name": "duration_ms", "type": "INTEGER" },
          { "name": "saved_audio_path", "type": "TEXT" }
        ]
      },
      {
        "name": "user_settings",
        "columns": [
          { "name": "id", "type": "INTEGER", "primary_key": true, "default": 1 },
          { "name": "default_voice_id", "type": "TEXT" },
          { "name": "default_rate", "type": "INTEGER", "default": 0 },
          { "name": "default_volume", "type": "INTEGER", "default": 100 },
          { "name": "auto_save_audio", "type": "INTEGER", "default": 0 },
          { "name": "audio_output_folder", "type": "TEXT", "default": "'./audio'" },
          { "name": "audio_format", "type": "TEXT", "default": "'wav'" },
          { "name": "keep_history", "type": "INTEGER", "default": 1 },
          { "name": "max_history_items", "type": "INTEGER", "default": 100 },
          { "name": "theme", "type": "TEXT", "default": "'system'" }
        ],
        "constraints": [
          "CHECK (id = 1)"
        ]
      }
    ],

    "migrations": [
      {
        "version": 1,
        "description": "Initial schema",
        "up": [
          "CREATE TABLE IF NOT EXISTS speech_history (...)",
          "CREATE TABLE IF NOT EXISTS user_settings (...)",
          "CREATE INDEX idx_history_created ON speech_history(created_at DESC)"
        ],
        "down": [
          "DROP TABLE speech_history",
          "DROP TABLE user_settings"
        ]
      }
    ],

    "named_queries": {
      "recent_history": "SELECT * FROM speech_history ORDER BY created_at DESC LIMIT :limit",
      "history_stats": "SELECT COUNT(*) as count, AVG(duration_ms) as avg_duration FROM speech_history"
    }
  }
}
```

---

# SECTION 6: SERVICES (Application Layer)

Service contracts that orchestrate use cases.

```json
{
  "services": [
    {
      "name": "SpeechService",
      "description": "Orchestrates text-to-speech operations",
      "dependencies": ["speech_engine", "speech_history_repository", "settings_repository"],

      "operations": [
        {
          "name": "speak",
          "description": "Synthesize and play speech",
          "params": [
            { "name": "request", "type": "SpeechRequest" }
          ],
          "returns": "SpeechResult",
          "preconditions": [
            "request.text.count > 0",
            "request.voice_id is valid"
          ],
          "steps": [
            { "validate": "request" },
            { "call": "speech_engine.speak", "with": ["request.text", "request.voice_id", "request.rate", "request.volume"] },
            { "if": "settings.keep_history", "then": { "call": "save_to_history", "with": ["request"] } },
            { "if": "settings.auto_save_audio", "then": { "call": "save_audio_file", "with": ["request"] } }
          ],
          "events": {
            "on_start": "speech_started",
            "on_progress": "speech_progress",
            "on_complete": "speech_completed",
            "on_error": "speech_failed"
          }
        },
        {
          "name": "stop",
          "description": "Stop current speech",
          "steps": [
            { "call": "speech_engine.stop" }
          ],
          "events": {
            "on_complete": "speech_stopped"
          }
        },
        {
          "name": "pause",
          "description": "Pause current speech",
          "preconditions": ["is_speaking"],
          "steps": [
            { "call": "speech_engine.pause" }
          ]
        },
        {
          "name": "resume",
          "description": "Resume paused speech",
          "preconditions": ["is_paused"],
          "steps": [
            { "call": "speech_engine.resume" }
          ]
        }
      ],

      "state": {
        "is_speaking": { "type": "boolean", "default": false },
        "is_paused": { "type": "boolean", "default": false },
        "current_progress": { "type": "real", "default": 0.0 }
      }
    },

    {
      "name": "HistoryService",
      "description": "Manages speech history",
      "dependencies": ["speech_history_repository", "settings_repository"],

      "operations": [
        {
          "name": "get_recent",
          "params": [{ "name": "limit", "type": "integer", "default": 50 }],
          "returns": "List[SpeechHistory]"
        },
        {
          "name": "search",
          "params": [{ "name": "query", "type": "string" }],
          "returns": "List[SpeechHistory]"
        },
        {
          "name": "delete",
          "params": [{ "name": "id", "type": "integer" }]
        },
        {
          "name": "clear_all",
          "confirm": "Are you sure you want to delete all history?"
        },
        {
          "name": "prune",
          "description": "Remove excess history items based on settings",
          "steps": [
            { "get": "settings.max_history_items", "as": "limit" },
            { "call": "repository.prune_to_limit", "with": ["limit"] }
          ]
        }
      ]
    }
  ]
}
```

---

# SECTION 7: USE CASES (Business Logic)

Complete business use cases with full logic, not just UI actions.

```json
{
  "use_cases": [
    {
      "id": "UC001",
      "name": "Speak Text",
      "actor": "User",
      "description": "User enters text and hears it spoken aloud",

      "trigger": "User clicks Speak button",

      "preconditions": [
        { "id": "PRE1", "description": "Text field is not empty" },
        { "id": "PRE2", "description": "A voice is selected" },
        { "id": "PRE3", "description": "Speech engine is initialized" }
      ],

      "main_flow": [
        {
          "step": 1,
          "actor": "User",
          "action": "Enters text in the text area"
        },
        {
          "step": 2,
          "actor": "User",
          "action": "Selects voice from dropdown"
        },
        {
          "step": 3,
          "actor": "User",
          "action": "Optionally adjusts rate/volume/pitch sliders"
        },
        {
          "step": 4,
          "actor": "User",
          "action": "Clicks 'Speak' button"
        },
        {
          "step": 5,
          "actor": "System",
          "action": "Validates input",
          "service": "SpeechService",
          "operation": "validate",
          "on_failure": "EF1"
        },
        {
          "step": 6,
          "actor": "System",
          "action": "Transitions to 'speaking' state",
          "state_machine": "app_states",
          "event": "speak"
        },
        {
          "step": 7,
          "actor": "System",
          "action": "Sends text to speech engine",
          "service": "SpeechService",
          "operation": "speak"
        },
        {
          "step": 8,
          "actor": "System",
          "action": "Updates progress bar as speech plays"
        },
        {
          "step": 9,
          "actor": "System",
          "action": "On completion, saves to history if enabled",
          "condition": "settings.keep_history = true"
        },
        {
          "step": 10,
          "actor": "System",
          "action": "Transitions to 'idle' state",
          "state_machine": "app_states",
          "event": "complete"
        }
      ],

      "alternative_flows": [
        {
          "id": "AF1",
          "name": "Save Audio",
          "branch_from": 4,
          "condition": "User has 'Save audio' checked",
          "steps": [
            { "step": "4a", "action": "System prompts for save location if not configured" },
            { "step": "4b", "action": "System saves audio file after speech completes" }
          ],
          "rejoins_at": 10
        }
      ],

      "exception_flows": [
        {
          "id": "EF1",
          "name": "Validation Failed",
          "trigger": "Step 5 validation fails",
          "steps": [
            { "step": "5a", "action": "System displays validation error" },
            { "step": "5b", "action": "System highlights invalid fields" }
          ],
          "outcome": "Use case ends, user corrects and retries"
        },
        {
          "id": "EF2",
          "name": "Speech Engine Error",
          "trigger": "Step 7 throws exception",
          "steps": [
            { "step": "7a", "action": "System transitions to 'error' state" },
            { "step": "7b", "action": "System displays error message" },
            { "step": "7c", "action": "System logs error for debugging" }
          ],
          "outcome": "Use case ends with error"
        }
      ],

      "postconditions": [
        { "id": "POST1", "description": "Speech has been played (success) or error displayed (failure)" },
        { "id": "POST2", "description": "History updated if enabled and successful" },
        { "id": "POST3", "description": "App is in 'idle' or 'error' state" }
      ],

      "test_scenarios": [
        {
          "name": "Happy path - basic speech",
          "inputs": {
            "text": "Hello world",
            "voice": "david",
            "rate": 0,
            "volume": 100
          },
          "steps": [
            { "action": "type", "target": "speech_text", "value": "Hello world" },
            { "action": "select", "target": "voice_select", "value": "david" },
            { "action": "click", "target": "speak_button" },
            { "action": "wait_for_state", "machine": "app_states", "state": "idle", "timeout": 10000 }
          ],
          "assertions": [
            { "type": "state", "machine": "app_states", "expected": "idle" },
            { "type": "count", "target": "speech_history", "expected": 1 }
          ]
        },
        {
          "name": "Empty text validation",
          "inputs": { "text": "" },
          "steps": [
            { "action": "clear", "target": "speech_text" },
            { "action": "click", "target": "speak_button" }
          ],
          "assertions": [
            { "type": "visible", "target": "validation_error" },
            { "type": "state", "machine": "app_states", "expected": "idle" }
          ]
        }
      ]
    },

    {
      "id": "UC002",
      "name": "Stop Speech",
      "actor": "User",
      "description": "User stops speech that is currently playing",

      "preconditions": [
        { "id": "PRE1", "description": "Speech is currently playing (app in 'speaking' state)" }
      ],

      "main_flow": [
        { "step": 1, "actor": "User", "action": "Clicks 'Stop' button" },
        { "step": 2, "actor": "System", "action": "Stops speech engine" },
        { "step": 3, "actor": "System", "action": "Transitions to 'idle' state" }
      ],

      "postconditions": [
        { "id": "POST1", "description": "Speech has stopped" },
        { "id": "POST2", "description": "App is in 'idle' state" }
      ]
    },

    {
      "id": "UC003",
      "name": "Replay from History",
      "actor": "User",
      "description": "User selects a past speech and replays it",

      "preconditions": [
        { "id": "PRE1", "description": "History contains at least one entry" }
      ],

      "main_flow": [
        { "step": 1, "actor": "User", "action": "Double-clicks history item" },
        { "step": 2, "actor": "System", "action": "Populates text field with history text" },
        { "step": 3, "actor": "System", "action": "Restores voice/rate/volume settings from history" },
        { "step": 4, "actor": "System", "action": "Triggers speak action" }
      ],

      "extends": "UC001",
      "extension_point": "After step 3"
    }
  ]
}
```

---

# SECTION 8: DOMAIN RULES

Business rules that apply across the application.

```json
{
  "domain_rules": [
    {
      "id": "DR001",
      "name": "Text Length Limit",
      "description": "Speech text must not exceed 10,000 characters",
      "entity": "SpeechRequest",
      "field": "text",
      "rule": "length <= 10000",
      "message": "Text exceeds maximum length of 10,000 characters",
      "severity": "error"
    },
    {
      "id": "DR002",
      "name": "Rate Range",
      "description": "Speech rate must be between -10 and +10",
      "entity": "SpeechRequest",
      "field": "rate",
      "rule": "value >= -10 AND value <= 10",
      "message": "Rate must be between -10 and +10",
      "severity": "error"
    },
    {
      "id": "DR003",
      "name": "History Pruning",
      "description": "History automatically prunes when exceeding max items",
      "trigger": "after_insert",
      "entity": "SpeechHistory",
      "condition": "count > settings.max_history_items",
      "action": "delete_oldest_entries"
    },
    {
      "id": "DR004",
      "name": "Default Voice Selection",
      "description": "If no voice selected, use default from settings",
      "entity": "SpeechRequest",
      "field": "voice_id",
      "condition": "value is null or empty",
      "action": "set_to settings.default_voice_id",
      "fallback": "use first available voice"
    }
  ]
}
```

---

# SECTION 9: EVENTS

Application-wide events for decoupled communication.

```json
{
  "events": [
    {
      "name": "speech_started",
      "description": "Fired when speech synthesis begins",
      "payload": {
        "request_id": "uuid",
        "text_length": "integer",
        "voice_id": "string"
      },
      "handlers": [
        { "target": "progress_bar", "action": "show" },
        { "target": "analytics", "action": "track_speech_start" }
      ]
    },
    {
      "name": "speech_progress",
      "description": "Fired periodically during speech",
      "payload": {
        "request_id": "uuid",
        "progress": "real",
        "word_index": "integer"
      },
      "handlers": [
        { "target": "progress_bar", "action": "update", "value": "event.progress" }
      ]
    },
    {
      "name": "speech_completed",
      "description": "Fired when speech finishes successfully",
      "payload": {
        "request_id": "uuid",
        "duration_ms": "integer"
      },
      "handlers": [
        { "target": "history_service", "action": "save_if_enabled" },
        { "target": "state_machine", "action": "trigger", "event": "complete" }
      ]
    },
    {
      "name": "speech_failed",
      "description": "Fired when speech synthesis fails",
      "payload": {
        "request_id": "uuid",
        "error_code": "string",
        "error_message": "string"
      },
      "handlers": [
        { "target": "error_display", "action": "show", "message": "event.error_message" },
        { "target": "state_machine", "action": "trigger", "event": "error" }
      ]
    },
    {
      "name": "voice_changed",
      "description": "User selected a different voice",
      "payload": {
        "old_voice_id": "string",
        "new_voice_id": "string"
      }
    },
    {
      "name": "settings_changed",
      "description": "User modified settings",
      "payload": {
        "setting": "string",
        "old_value": "any",
        "new_value": "any"
      }
    }
  ]
}
```

---

# SECTION 10: ERRORS

Structured error definitions.

```json
{
  "errors": [
    {
      "code": "E001",
      "name": "EMPTY_TEXT",
      "message": "Please enter some text to speak",
      "severity": "warning",
      "user_action": "Enter text in the text area"
    },
    {
      "code": "E002",
      "name": "NO_VOICE_SELECTED",
      "message": "Please select a voice",
      "severity": "warning",
      "user_action": "Select a voice from the dropdown"
    },
    {
      "code": "E003",
      "name": "SPEECH_ENGINE_ERROR",
      "message": "Speech engine encountered an error: {details}",
      "severity": "error",
      "user_action": "Try again or restart the application"
    },
    {
      "code": "E004",
      "name": "AUDIO_SAVE_FAILED",
      "message": "Could not save audio file: {details}",
      "severity": "error",
      "user_action": "Check disk space and permissions"
    },
    {
      "code": "E005",
      "name": "DATABASE_ERROR",
      "message": "Database error: {details}",
      "severity": "error",
      "user_action": "Restart the application",
      "log": true
    }
  ]
}
```

---

# SECTION 11: GUI (Windows, Layouts, Widgets)

Now with explicit data bindings to entities.

```json
{
  "windows": [
    {
      "id": "main_window",
      "type": "window",
      "title": "Speech Studio",
      "size": { "width": 900, "height": 700 },
      "state_machine": "app_states",

      "data_context": {
        "current_request": { "type": "SpeechRequest", "create": true },
        "settings": { "type": "UserSettings", "load_on_start": true },
        "history": { "type": "List[SpeechHistory]", "source": "history_service.get_recent" },
        "voices": { "type": "List[Voice]", "source": "voice_repository.get_all" }
      },

      "layout": {
        "type": "vertical",
        "padding": 16,
        "spacing": 12,
        "children": [
          {
            "type": "panel",
            "title": "Text to Speak",
            "layout": {
              "type": "vertical",
              "children": [
                {
                  "id": "speech_text",
                  "type": "text_area",
                  "rows": 6,
                  "bind": "current_request.text",
                  "placeholder": "Enter text to speak...",
                  "validation": { "entity": "SpeechRequest", "field": "text" }
                },
                {
                  "type": "horizontal",
                  "align": "right",
                  "children": [
                    {
                      "id": "char_count",
                      "type": "label",
                      "style": "caption",
                      "text": "{current_request.text.count} / 10000 characters"
                    }
                  ]
                }
              ]
            }
          },

          {
            "type": "panel",
            "title": "Voice Settings",
            "layout": {
              "type": "grid",
              "columns": 2,
              "children": [
                {
                  "type": "label",
                  "text": "Voice:"
                },
                {
                  "id": "voice_select",
                  "type": "dropdown",
                  "options_source": "voices",
                  "label_field": "name",
                  "value_field": "id",
                  "bind": "current_request.voice_id",
                  "on_change": "preview_voice"
                },

                { "type": "label", "text": "Speed:" },
                {
                  "id": "rate_slider",
                  "type": "slider",
                  "min": -10, "max": 10, "step": 1,
                  "bind": "current_request.rate",
                  "show_value": true,
                  "labels": { "-10": "Slow", "0": "Normal", "10": "Fast" }
                },

                { "type": "label", "text": "Volume:" },
                {
                  "id": "volume_slider",
                  "type": "slider",
                  "min": 0, "max": 100, "step": 5,
                  "bind": "current_request.volume",
                  "show_value": true,
                  "suffix": "%"
                },

                { "type": "label", "text": "Pitch:" },
                {
                  "id": "pitch_slider",
                  "type": "slider",
                  "min": -10, "max": 10, "step": 1,
                  "bind": "current_request.pitch",
                  "show_value": true
                }
              ]
            }
          },

          {
            "type": "horizontal",
            "spacing": 8,
            "children": [
              {
                "id": "speak_button",
                "type": "button",
                "text": "Speak",
                "style": "primary",
                "icon": "play",
                "on_click": "do_speak",
                "enabled_when": "can_speak",
                "keyboard_shortcut": "Ctrl+Enter"
              },
              {
                "id": "pause_button",
                "type": "button",
                "text": "Pause",
                "icon": "pause",
                "on_click": "do_pause",
                "visible_when": "state:speaking"
              },
              {
                "id": "stop_button",
                "type": "button",
                "text": "Stop",
                "icon": "stop",
                "on_click": "do_stop",
                "enabled_when": "state:speaking OR state:paused"
              },
              { "type": "expander" },
              {
                "id": "save_audio_check",
                "type": "checkbox",
                "label": "Save audio to file",
                "bind": "settings.auto_save_audio"
              }
            ]
          },

          {
            "id": "progress_bar",
            "type": "progress",
            "bind": "speech_service.current_progress",
            "visible_when": "state:speaking"
          },

          {
            "id": "error_label",
            "type": "label",
            "style": "error",
            "bind": "last_error.message",
            "visible_when": "last_error != null"
          },

          {
            "type": "panel",
            "title": "History",
            "collapsible": true,
            "layout": {
              "type": "vertical",
              "children": [
                {
                  "id": "history_grid",
                  "type": "grid",
                  "data_source": "history",
                  "columns": [
                    { "field": "text", "label": "Text", "width": 400, "truncate": 50 },
                    { "field": "voice_name", "label": "Voice", "width": 100 },
                    { "field": "created_at", "label": "Date", "width": 150, "format": "relative" }
                  ],
                  "on_double_click": "replay_from_history",
                  "context_menu": [
                    { "label": "Replay", "action": "replay_from_history" },
                    { "label": "Copy Text", "action": "copy_history_text" },
                    { "type": "separator" },
                    { "label": "Delete", "action": "delete_history_item", "confirm": true }
                  ]
                }
              ]
            }
          }
        ]
      },

      "menu": {
        "items": [
          {
            "label": "File",
            "children": [
              { "label": "Save Audio As...", "action": "save_audio_as", "shortcut": "Ctrl+S" },
              { "type": "separator" },
              { "label": "Exit", "action": "exit", "shortcut": "Alt+F4" }
            ]
          },
          {
            "label": "Edit",
            "children": [
              { "label": "Clear Text", "action": "clear_text", "shortcut": "Ctrl+Delete" },
              { "label": "Paste & Speak", "action": "paste_and_speak", "shortcut": "Ctrl+Shift+V" }
            ]
          },
          {
            "label": "Settings",
            "children": [
              { "label": "Preferences...", "action": "show_settings", "shortcut": "Ctrl+," }
            ]
          },
          {
            "label": "Help",
            "children": [
              { "label": "About", "action": "show_about" }
            ]
          }
        ]
      }
    }
  ]
}
```

---

# SECTION 12: STATE MACHINES

UI state management with full widget state mapping.

```json
{
  "state_machines": [
    {
      "id": "app_states",
      "initial": "idle",

      "states": [
        {
          "name": "idle",
          "description": "Ready for user input",
          "on_enter": ["clear_progress", "clear_error"],
          "widgets": {
            "speak_button": { "enabled": true, "text": "Speak" },
            "pause_button": { "visible": false },
            "stop_button": { "enabled": false },
            "progress_bar": { "visible": false },
            "speech_text": { "enabled": true },
            "voice_select": { "enabled": true },
            "rate_slider": { "enabled": true },
            "volume_slider": { "enabled": true }
          }
        },
        {
          "name": "speaking",
          "description": "Speech in progress",
          "on_enter": ["start_progress_updates"],
          "on_exit": ["stop_progress_updates"],
          "widgets": {
            "speak_button": { "enabled": false },
            "pause_button": { "visible": true, "enabled": true },
            "stop_button": { "enabled": true },
            "progress_bar": { "visible": true },
            "speech_text": { "enabled": false },
            "voice_select": { "enabled": false }
          }
        },
        {
          "name": "paused",
          "description": "Speech paused",
          "widgets": {
            "speak_button": { "enabled": true, "text": "Resume" },
            "pause_button": { "visible": false },
            "stop_button": { "enabled": true },
            "progress_bar": { "visible": true }
          }
        },
        {
          "name": "error",
          "description": "Error state",
          "on_enter": ["show_error_notification"],
          "widgets": {
            "speak_button": { "enabled": true },
            "stop_button": { "enabled": false },
            "error_label": { "visible": true }
          }
        }
      ],

      "transitions": [
        { "event": "speak", "from": "idle", "to": "speaking", "guard": "can_speak", "action": "start_speech" },
        { "event": "speak", "from": "paused", "to": "speaking", "action": "resume_speech" },
        { "event": "pause", "from": "speaking", "to": "paused", "action": "pause_speech" },
        { "event": "stop", "from": ["speaking", "paused"], "to": "idle", "action": "stop_speech" },
        { "event": "complete", "from": "speaking", "to": "idle", "action": "on_speech_complete" },
        { "event": "error", "from": "*", "to": "error", "action": "handle_error" },
        { "event": "dismiss_error", "from": "error", "to": "idle", "action": "clear_error" }
      ]
    }
  ]
}
```

---

# SECTION 13: TRACEABILITY MATRIX

Maps requirements to implementation.

```json
{
  "traceability": {
    "requirements_to_use_cases": {
      "REQ001_speak_text": ["UC001"],
      "REQ002_stop_speech": ["UC002"],
      "REQ003_save_history": ["UC001"],
      "REQ004_replay_history": ["UC003"]
    },

    "use_cases_to_services": {
      "UC001": ["SpeechService.speak", "HistoryService.save"],
      "UC002": ["SpeechService.stop"],
      "UC003": ["HistoryService.get", "SpeechService.speak"]
    },

    "services_to_repositories": {
      "SpeechService": ["VoiceRepository"],
      "HistoryService": ["SpeechHistoryRepository", "UserSettingsRepository"]
    },

    "entities_to_tables": {
      "SpeechRequest": null,
      "SpeechHistory": "speech_history",
      "UserSettings": "user_settings",
      "Voice": null
    }
  }
}
```

---

# CODE GENERATION

The spec generates complete, layered code:

```
generated/
├── domain/
│   ├── speech_request.e
│   ├── voice.e
│   ├── speech_history.e
│   └── user_settings.e
├── repositories/
│   ├── speech_history_repository.e
│   ├── user_settings_repository.e
│   └── voice_repository.e
├── services/
│   ├── speech_service.e
│   └── history_service.e
├── presentation/
│   ├── main_window.e
│   └── settings_dialog.e
├── state/
│   └── app_states_machine.e
└── tests/
    ├── uc001_speak_text_test.e
    ├── uc002_stop_speech_test.e
    └── integration/
        └── speech_flow_test.e
```

---

This expanded protocol provides complete top-down, bottom-up, and middle-out coverage of the entire application.
