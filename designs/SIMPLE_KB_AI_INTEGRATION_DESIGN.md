# Simple KB AI Integration Design

## Executive Summary

This document describes the architecture for integrating AI capabilities into simple_kb, transforming it from a pure FTS5 search tool into an intelligent knowledge assistant that can understand natural language questions and synthesize answers from the knowledge base.

**Key Principle**: Graceful degradation - all features work without AI; AI enhances but is never required.

---

## Architecture Overview

```
                    +------------------+
                    |   User Query     |
                    | "How do I parse  |
                    |  JSON in Eiffel?"|
                    +--------+---------+
                             |
                             v
                    +------------------+
                    |  KB_AI_ROUTER    |
                    | (Mode Detection) |
                    +--------+---------+
                             |
            +----------------+----------------+
            |                                 |
            v                                 v
   +------------------+              +------------------+
   |  NO AI MODE      |              |  AI MODE         |
   | (Current FTS5)   |              | (Two-Phase RAG)  |
   +------------------+              +--------+---------+
            |                                 |
            v                                 |
   +------------------+                       |
   | Direct FTS5      |                       |
   | Search Results   |                       |
   +------------------+                       |
                                              v
                              +-------------------------------+
                              |  PHASE 1: Query Generation    |
                              |  AI converts NL -> KB queries |
                              +---------------+---------------+
                                              |
                                              v
                              +-------------------------------+
                              |  KB Query Execution           |
                              |  FTS5 search, class/feature   |
                              |  lookups, pattern matching    |
                              +---------------+---------------+
                                              |
                                              v
                              +-------------------------------+
                              |  PHASE 2: Answer Synthesis    |
                              |  AI synthesizes answer from   |
                              |  raw data + user question     |
                              +---------------+---------------+
                                              |
                                              v
                              +-------------------------------+
                              |  Formatted Response           |
                              |  with citations/sources       |
                              +-------------------------------+
```

---

## Core Components

### 1. KB_AI_CONFIG

Manages AI provider configuration and credentials.

```eiffel
class KB_AI_CONFIG

feature -- Status
    has_ai_configured: BOOLEAN
        -- Is any AI provider configured?

    available_providers: ARRAYED_LIST [STRING_32]
        -- List of providers with valid credentials
        -- Checks: ANTHROPIC_API_KEY, OPENAI_API_KEY, GOOGLE_AI_KEY, etc.
        -- Always includes "ollama" if localhost:11434 responds

    active_provider: detachable STRING_32
        -- Currently selected provider (Void if none)

    is_ollama_available: BOOLEAN
        -- Is local Ollama running?

feature -- Configuration
    set_provider (a_name: STRING_32)
        -- Select active provider

    configure_claude (a_api_key: STRING_32)
        -- Set up Claude API access

    configure_openai (a_api_key: STRING_32)
        -- Set up OpenAI API access

    configure_gemini (a_api_key: STRING_32)
        -- Set up Google Gemini API access

    configure_ollama (a_base_url: STRING_32)
        -- Configure Ollama endpoint (default: localhost:11434)

feature -- Persistence
    save_config
        -- Save to ~/.simple_kb/ai_config.json

    load_config
        -- Load saved configuration
```

### 2. KB_AI_ROUTER

Routes queries based on AI availability and user preference.

```eiffel
class KB_AI_ROUTER

feature -- Routing
    process_query (a_query: STRING_32): KB_QUERY_RESULT
        -- Route query to appropriate handler
        do
            if ai_config.has_ai_configured and use_ai_mode then
                Result := process_with_ai (a_query)
            else
                Result := process_direct (a_query)
            end
        end

    use_ai_mode: BOOLEAN
        -- User preference for AI-assisted queries
        -- Toggled via: kb ai on/off

feature -- Direct Mode (No AI)
    process_direct (a_query: STRING_32): KB_QUERY_RESULT
        -- Current FTS5 search behavior
        -- Tries: search, class, feature, error, pattern

feature -- AI Mode (Two-Phase RAG)
    process_with_ai (a_query: STRING_32): KB_QUERY_RESULT
        -- Phase 1: Generate KB queries from NL
        -- Phase 2: Execute queries, synthesize answer
```

### 3. KB_QUERY_GENERATOR (Phase 1)

AI converts natural language to structured KB queries.

```eiffel
class KB_QUERY_GENERATOR

feature -- Query Generation
    generate_queries (a_natural_language: STRING_32): ARRAYED_LIST [KB_STRUCTURED_QUERY]
        -- Ask AI to convert user question into KB operations
        local
            l_prompt: STRING_32
            l_response: AI_RESPONSE
            l_queries: ARRAYED_LIST [KB_STRUCTURED_QUERY]
        do
            l_prompt := build_query_prompt (a_natural_language)
            l_response := ai_client.ask_with_system (System_prompt_query_gen, l_prompt)
            Result := parse_query_response (l_response.content)
        end

feature {NONE} -- Prompts
    System_prompt_query_gen: STRING_32 = "[
        You are a query translator for an Eiffel knowledge base.

        Available KB operations:
        - search:<terms> - Full-text search
        - class:<name> - Get class details
        - feature:<class>.<feature> - Get feature details
        - error:<code> - Look up compiler error (VEVI, VD89, etc.)
        - pattern:<name> - Get design pattern
        - example:<topic> - Find code examples
        - library:<name> - Get library info
        - ancestors:<class> - Get inheritance chain
        - descendants:<class> - Get child classes

        Convert the user's question into 1-5 KB queries.
        Return as TOON format:

        queries[N]{op,arg}:
          search,JSON parsing
          class,SIMPLE_JSON
          example,json

        Only output the TOON, no explanation.
    ]"
```

### 4. KB_ANSWER_SYNTHESIZER (Phase 2)

AI synthesizes final answer from KB data.

```eiffel
class KB_ANSWER_SYNTHESIZER

feature -- Synthesis
    synthesize (a_question: STRING_32; a_raw_data: KB_RAW_DATA): STRING_32
        -- Generate answer from question + KB results
        local
            l_context: STRING_32
            l_prompt: STRING_32
        do
            -- Encode raw data as TOON for token efficiency
            l_context := toon_encoder.encode_kb_data (a_raw_data)
            l_prompt := build_synthesis_prompt (a_question, l_context)

            Result := ai_client.ask_with_system (System_prompt_synthesis, l_prompt).content
        end

feature {NONE} -- Prompts
    System_prompt_synthesis: STRING_32 = "[
        You are an Eiffel programming expert answering developer questions.

        You have been given:
        1. The user's original question
        2. Raw data retrieved from the Eiffel Knowledge Base

        Guidelines:
        - Answer the question directly and concisely
        - Use code examples from the KB data when relevant
        - Cite sources: [CLASS_NAME] or [library:simple_json]
        - If KB data is insufficient, say so honestly
        - Follow Eiffel conventions: Design by Contract, void safety

        Format response as markdown with code blocks for Eiffel.
    ]"
```

### 5. KB_TOON_ADAPTER

Token-efficient encoding of KB data for AI context.

```eiffel
class KB_TOON_ADAPTER

feature -- Encoding
    encode_kb_data (a_data: KB_RAW_DATA): STRING_32
        -- Convert KB results to compact TOON format
        -- Achieves 30-60% token reduction vs JSON
        local
            l_builder: TOON_BUILDER
        do
            create l_builder.make

            -- Classes section
            if not a_data.classes.is_empty then
                l_builder.start_array ("classes", <<"name", "lib", "desc", "parents">>)
                across a_data.classes as c loop
                    l_builder.row (<<c.name, c.library, c.description, c.parents_csv>>)
                end
                l_builder.end_array
            end

            -- Features section
            if not a_data.features.is_empty then
                l_builder.start_array ("features", <<"class", "name", "sig", "desc">>)
                across a_data.features as f loop
                    l_builder.row (<<f.class_name, f.name, f.signature, f.description>>)
                end
                l_builder.end_array
            end

            -- Examples, errors, patterns similarly...

            Result := l_builder.to_string
        end

feature -- Token Estimation
    estimate_tokens (a_data: KB_RAW_DATA): TUPLE [json_tokens, toon_tokens, savings_percent: INTEGER]
        -- Compare JSON vs TOON token usage
```

---

## Onboarding: Users Without AI Access

### KB_AI_SETUP_GUIDE

For users without AI configured, simple_kb provides setup guidance.

```eiffel
class KB_AI_SETUP_GUIDE

feature -- Help Command
    show_setup_help
        -- Display when user runs: kb ai setup
        do
            io.put_string ("[
                ============================================
                SETTING UP AI ACCESS FOR SIMPLE_KB
                ============================================

                simple_kb works great without AI (FTS5 search).
                Adding AI enables natural language queries and
                intelligent answer synthesis.

                OPTION 1: Local AI (Free, Private)
                ---------------------------------
                Install Ollama: https://ollama.com/download

                Then run:
                  ollama pull llama3
                  ollama serve

                simple_kb will auto-detect Ollama at localhost:11434.

                OPTION 2: Claude API (Best Quality)
                -----------------------------------
                1. Get API key: https://console.anthropic.com/
                2. Set environment variable:
                   Windows: setx ANTHROPIC_API_KEY "sk-ant-..."
                   Linux:   export ANTHROPIC_API_KEY="sk-ant-..."
                3. Restart terminal

                OPTION 3: OpenAI API
                --------------------
                1. Get API key: https://platform.openai.com/
                2. Set environment variable:
                   Windows: setx OPENAI_API_KEY "sk-..."
                   Linux:   export OPENAI_API_KEY="sk-..."

                OPTION 4: Google Gemini API (Free Tier Available)
                -------------------------------------------------
                1. Get API key: https://makersuite.google.com/
                2. Set environment variable:
                   Windows: setx GOOGLE_AI_KEY "..."
                   Linux:   export GOOGLE_AI_KEY="..."

                VERIFY SETUP
                ------------
                Run: kb ai status

                This shows which providers are configured.
            ]")
        end

feature -- Prompt Templates
    get_prompt_for_provider (a_provider: STRING_32; a_question: STRING_32): STRING_32
        -- Generate a prompt the user can paste into their AI
        -- For users who have web access but not API access
        do
            Result := "[
                [EIFFEL KNOWLEDGE BASE QUERY]

                I'm working with Eiffel programming language.
                I need help with: $QUESTION

                Context about Eiffel:
                - Uses Design by Contract (require/ensure/invariant)
                - Void-safe (detachable/attached types)
                - SCOOP for concurrency
                - Libraries use simple_* naming convention

                Please provide:
                1. Direct answer with Eiffel code examples
                2. Relevant class/feature names to look up
                3. Any library dependencies needed
            ]"
            Result := Result.twin
            Result.replace_substring_all ("$QUESTION", a_question)
        end
```

---

## CLI Commands

### New AI-Related Commands

```
kb ai status          Show AI configuration status
kb ai setup           Show setup instructions
kb ai on              Enable AI-assisted mode
kb ai off             Disable AI (use direct FTS5)
kb ai provider <name> Switch active provider (claude/ollama/openai/gemini)
kb ai prompt <query>  Generate prompt for manual AI use (no API needed)
kb ai test            Test current AI connection
```

### Enhanced Query Experience

```
# Without AI (current behavior)
kb> search JSON parsing
  [Results from FTS5 search]

# With AI enabled
kb> How do I parse JSON in Eiffel?

  Based on the simple_json library, here's how to parse JSON:

  ```eiffel
  local
      json: SIMPLE_JSON
      value: SIMPLE_JSON_VALUE
  do
      create json.make
      value := json.parse ("{\"name\": \"Alice\"}")
      if attached value.object_value as obj then
          print (obj.string_item ("name"))  -- "Alice"
      end
  end
  ```

  Key classes:
  - SIMPLE_JSON: Main parser facade [library:simple_json]
  - SIMPLE_JSON_VALUE: Parsed value wrapper
  - SIMPLE_JSON_OBJECT: Object access methods

  Sources: [SIMPLE_JSON], [example:json_parsing]
```

---

## Token Efficiency Analysis

### TOON vs JSON for KB Data

**Example: Class lookup result**

JSON (68 tokens estimated):
```json
{"class":"SIMPLE_JSON","library":"simple_json","description":"JSON parser and generator","parents":["ANY"],"features":[{"name":"parse","signature":"(STRING_32): SIMPLE_JSON_VALUE","description":"Parse JSON string"}]}
```

TOON (41 tokens estimated, 40% reduction):
```
class: SIMPLE_JSON
library: simple_json
description: JSON parser and generator
parents[1]: ANY
features[1]{name,sig,desc}:
  parse,(STRING_32): SIMPLE_JSON_VALUE,Parse JSON string
```

### Estimated Token Usage Per Query

| Phase | Without TOON | With TOON | Savings |
|-------|-------------|-----------|---------|
| Query Generation | 500 | 500 | 0% (no data yet) |
| KB Results (avg) | 2000 | 1200 | 40% |
| Answer Synthesis | 800 | 800 | 0% (output) |
| **Total** | **3300** | **2500** | **24%** |

For complex queries with many results, savings can reach 40-50%.

---

## Implementation Phases

### Phase 1: Foundation (Current Sprint)
- [ ] KB_AI_CONFIG - credential management
- [ ] KB_AI_ROUTER - mode detection and routing
- [ ] CLI commands: ai status, ai setup, ai on/off
- [ ] Graceful degradation tests

### Phase 2: Query Generation
- [ ] KB_QUERY_GENERATOR with prompt engineering
- [ ] KB_STRUCTURED_QUERY parser
- [ ] Multi-query execution
- [ ] TOON integration for responses

### Phase 3: Answer Synthesis
- [ ] KB_ANSWER_SYNTHESIZER
- [ ] KB_TOON_ADAPTER for context encoding
- [ ] Citation formatting
- [ ] Response streaming (optional)

### Phase 4: Multi-Provider Support
- [ ] Expand simple_ai_client for OpenAI, Gemini, Grok
- [ ] Provider capability detection
- [ ] Fallback chains (Claude -> OpenAI -> Ollama)
- [ ] Cost tracking across providers

### Phase 5: Advanced Features
- [ ] Conversation memory (multi-turn)
- [ ] Query caching
- [ ] Embedding-based semantic search
- [ ] Specialist model routing

---

## Provider Support Matrix (simple_ai_client expansion)

| Provider | Status | API Key Env Var | Models |
|----------|--------|-----------------|--------|
| Ollama | Done | (none, local) | llama3, mistral, codellama |
| Claude | Done | ANTHROPIC_API_KEY | sonnet, opus, haiku |
| OpenAI | TODO | OPENAI_API_KEY | gpt-4, gpt-4-turbo, gpt-3.5 |
| Gemini | TODO | GOOGLE_AI_KEY | gemini-pro, gemini-ultra |
| Grok | TODO | XAI_API_KEY | grok-1 |

### New Classes Needed in simple_ai_client

```
src/providers/
  openai/
    openai_client.e        -- OpenAI chat completions
  gemini/
    gemini_client.e        -- Google Gemini API
  grok/
    grok_client.e          -- xAI Grok API

src/core/
  ai_provider_registry.e   -- Dynamic provider discovery
  ai_provider_config.e     -- Unified credential store
```

---

## Error Handling

### Graceful Degradation Scenarios

| Scenario | Behavior |
|----------|----------|
| No AI configured | Works as current FTS5 search |
| AI configured but offline | Falls back to FTS5 with warning |
| API rate limited | Falls back to FTS5, suggests retry later |
| API key invalid | Disables AI, shows setup instructions |
| Query generation fails | Falls back to keyword extraction heuristic |
| Synthesis fails | Returns raw KB data with apology |

---

## Security Considerations

1. **API Keys**: Stored in environment variables, never in code or config files
2. **Local Option**: Ollama provides fully offline operation
3. **Data Privacy**: KB queries go to external APIs - document this clearly
4. **Prompt Injection**: Sanitize user input before including in prompts
5. **Cost Control**: Warn users about API costs, support usage limits

---

## Summary

This design transforms simple_kb into an intelligent assistant while preserving its core value as a fast, offline-capable FTS5 search tool. Key principles:

1. **AI is optional** - Everything works without it
2. **Graceful degradation** - Failures fall back to working features
3. **Token efficiency** - TOON format reduces costs 30-40%
4. **Multi-provider** - Users choose their preferred AI
5. **Privacy-first** - Local Ollama option for sensitive work
6. **Self-documenting** - Built-in setup help and prompt templates

The two-phase RAG pattern (Query Generation -> Answer Synthesis) ensures accurate, grounded responses based on actual KB data rather than AI hallucinations.
