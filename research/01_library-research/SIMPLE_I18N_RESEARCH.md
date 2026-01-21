# SIMPLE_I18N Research Report

**Date:** 2025-12-08
**Library:** simple_i18n
**Wraps:** ISE i18n library (`$ISE_LIBRARY/library/i18n/i18n.ecf`)

---

## Seven-Step Research Process

### Step 1: ISE Library Structure

**Location:** `$ISE_LIBRARY/library/i18n/`

**Key Components:**
```
i18n/
├── i18n_locale_manager.e     -- Main entry point for locale management
├── i18n_locale.e             -- Represents a locale with formatters
├── i18n_locale_id.e          -- Locale identifier (language_region)
├── datasource/               -- Translation file loading
│   └── file/
│       └── mo/               -- GNU gettext .mo file support
├── dictionary/               -- Translation storage
│   ├── i18n_dictionary.e     -- Abstract dictionary
│   ├── i18n_hash_table_dictionary.e  -- Hash-based impl
│   └── i18n_dictionary_entry.e       -- Translation entry
├── formatter/                -- Locale-aware formatting
│   ├── i18n_date_formatter.e
│   ├── i18n_value_formatter.e
│   ├── i18n_currency_formatter.e
│   └── i18n_string_formatter.e
├── locale/                   -- Host locale detection
│   ├── nls/                  -- Windows NLS implementation
│   └── posix/                -- POSIX implementation
└── locale_info/              -- Locale information structures
    ├── i18n_locale_info.e    -- Combines all info types
    ├── i18n_numeric_info.e   -- Number formatting
    ├── i18n_currency_info.e  -- Currency formatting
    └── i18n_date_time_info.e -- Date/time formatting
```

### Step 2: Core Classes Analysis

**I18N_LOCALE_MANAGER**
- Entry point for accessing locales
- `make (a_uri)` - Initialize with translation files directory
- `locale (a_locale_id)` - Get locale for specific ID
- `system_locale` - Get system default locale
- `available_locales` - List all available locales
- `has_translations (a_locale_id)` - Check translation availability

**I18N_LOCALE**
- Main interface for translation and formatting
- `translation (original)` - Translate a string
- `plural_translation (singular, plural, count)` - Pluralization
- `translation_in_context (original, context)` - Context-aware translation
- `formatted_string (template, values)` - String formatting with $1, $2 placeholders
- `date_formatter` - Locale-specific date formatting
- `value_formatter` - Number formatting
- `currency_formatter` - Currency formatting

**I18N_LOCALE_ID**
- Identifies a locale
- `make (language, region, script)` - Create from components
- `make_from_string (identifier)` - Parse "en_US", "fr_FR", etc.
- `language` - Language code (e.g., "en")
- `region` - Region code (e.g., "US")
- `name` - Full name (e.g., "en_US")

**I18N_LOCALE_INFO** (inherits from)
- I18N_NUMERIC_INFO
  - `value_decimal_separator` - e.g., "." or ","
  - `value_group_separator` - Thousands separator
- I18N_CURRENCY_INFO
  - `currency_symbol` - e.g., "$", "€"
  - `currency_decimal_separator`
- I18N_DATE_TIME_INFO
  - `long_date_format` - Date pattern
  - `long_time_format` - Time pattern

### Step 3: Translation System

**Dictionary Entry Structure:**
```eiffel
I18N_DICTIONARY_ENTRY
  make (original, translation)      -- Simple translation
  make_with_plural (singular, translation, plural)  -- With plural
  singular_translation              -- The translation
  plural_translations               -- Array of plural forms
  identifier                        -- Key for lookup
```

**Plural Forms (I18N_PLURAL_TOOLS):**
- `one_plural_form` - Languages with no plural distinction
- `two_plural_forms_singular_one` - English-like (n != 1)
- `three_plural_forms_special_slavic` - Russian-like
- `four_plural_forms_special_slovenian` - Slovenian

### Step 4: Host Locale Detection

**Windows (NLS):**
```eiffel
I18N_HOST_LOCALE_IMP (locale/nls/)
  system_locale_id                  -- Get Windows locale
  is_available (locale_id)          -- Check if locale exists
  create_locale_info (locale_id)    -- Get formatting info
```

**POSIX:**
```eiffel
I18N_HOST_LOCALE_IMP (locale/posix/)
  -- Uses LC_* environment variables
  -- Parses /etc/locale.conf
```

### Step 5: Design Decisions for simple_i18n

**Simplifications:**
1. **No .mo files required** - In-memory dictionary for programmatic translations
2. **Simple translation registration** - `register_translation(key, value)`
3. **Simplified plural handling** - Basic singular/plural (not full plural forms)
4. **Direct locale info access** - Through inherited features

**Key Features:**
- Translation registration and lookup
- String formatting with $1, $2 placeholders
- System locale detection
- Locale-aware date/time/number/currency formatting
- Locale ID parsing and creation

### Step 6: API Design

```eiffel
class SIMPLE_I18N

create
  make,                          -- System locale
  make_with_locale               -- Specific locale (e.g., "en_US")

feature -- Translation
  register_translation (key, value)
  register_plural_translation (singular_key, plural_key, singular, plural)
  register_translations (pairs)
  translate (key): STRING_32
  translate_plural (key, count): STRING_32
  has_translation (key): BOOLEAN

feature -- Formatting
  format (template, args): STRING_32       -- $1, $2 placeholders
  translate_format (key, args): STRING_32  -- Translate then format
  format_date (date): STRING_32
  format_time (time): STRING_32
  format_date_time (datetime): STRING_32
  format_number (value): STRING_32
  format_integer (value): STRING_32
  format_currency (value): STRING_32

feature -- Locale Info
  language: STRING_32            -- "en"
  region: STRING_32              -- "US"
  locale_name: STRING_32         -- "en_US"
  decimal_separator: STRING_32   -- "."
  thousands_separator: STRING_32 -- ","
  currency_symbol: STRING_32     -- "$"
  date_format: STRING_32
  time_format: STRING_32
  is_system_locale: BOOLEAN
```

### Step 7: Dependencies

**ISE Libraries:**
- `$ISE_LIBRARY/library/i18n/i18n.ecf`
- `$ISE_LIBRARY/library/time/time.ecf` (for date/time classes)
- `$ISE_LIBRARY/library/base/base.ecf`

**Key Classes Used:**
- I18N_HOST_LOCALE_IMP - Host locale detection
- I18N_LOCALE_ID - Locale identification
- I18N_LOCALE_INFO - Formatting information (via inheritance)
- I18N_DICTIONARY_ENTRY - Translation entries
- HASH_TABLE - In-memory translation storage

---

## Implementation Summary

**Files Created:**
- `simple_i18n.ecf` - ECF configuration
- `src/simple_i18n.e` - Main wrapper class
- `testing/i18n_test_app.e` - Test application

**Test Results:** All 10 tests passing
- test_create_with_system_locale
- test_create_with_specific_locale
- test_register_translation
- test_translate
- test_translate_missing
- test_plural_translation
- test_format_placeholders
- test_translate_format
- test_register_multiple
- test_locale_info

---

## Notes

1. **Complexity:** The ISE i18n library is designed for full gettext (.mo) file support with complex plural rules. Our wrapper simplifies to programmatic registration.

2. **Platform Differences:** Windows uses NLS, POSIX uses LC_* environment. I18N_HOST_LOCALE_IMP is platform-specific.

3. **Locale Info Inheritance:** I18N_LOCALE_INFO inherits from I18N_NUMERIC_INFO, I18N_CURRENCY_INFO, and I18N_DATE_TIME_INFO, so features are accessed directly (not through nested objects).

4. **Future Enhancement:** Could add .mo file loading via I18N_LOCALE_MANAGER for full gettext support.
