# SIMPLE_FILE Research Report

**Date:** 2025-12-08
**Library:** simple_file
**Wraps:** ISE base library file classes (`$ISE_LIBRARY/library/base/base.ecf`)

---

## Seven-Step Research Process

### Step 1: ISE Library Structure

**Location:** `$ISE_LIBRARY/library/base/elks/kernel/` and `$ISE_LIBRARY/library/base/ise/kernel/`

**Key Classes:**
```
base/elks/kernel/
├── file.e                    -- Deferred base class for all files
├── plain_text_file.e         -- Text file with encoding support
├── raw_file.e                -- Binary file operations
├── file_info.e               -- File metadata (size, dates, permissions)
├── file_name.e               -- Path construction (obsolete, use PATH)
├── directory.e               -- Directory operations
└── std_files.e               -- Standard input/output/error

base/ise/kernel/
├── path.e                    -- Modern path handling
├── file_utilities.e          -- Expanded class with file/directory helpers
└── execution_environment.e   -- Environment variables, paths
```

### Step 2: Core Classes Analysis

**FILE (deferred)**
- Abstract base class for file operations
- `make_with_name (a_name)` - Create file object with name
- `make_with_path (a_path)` - Create file object with PATH
- `make_open_read (a_name)` - Open for reading
- `make_open_write (a_name)` - Open for writing (creates if needed)
- `make_open_append (a_name)` - Open for appending
- `make_open_read_write (a_name)` - Open for read/write
- `make_open_temporary` - Create unique temporary file
- `exists: BOOLEAN` - Does file exist?
- `is_readable, is_writable, is_executable: BOOLEAN` - Permissions
- `is_plain, is_directory, is_symlink: BOOLEAN` - File type
- `count: INTEGER` - File size in bytes
- `date: INTEGER` - Modification timestamp
- `access_date: INTEGER` - Last access timestamp
- `path: PATH` - Associated path object
- `open_read, open_write, open_append` - Open operations
- `close` - Close file
- `delete` - Delete file
- `rename_file (new_name)` - Rename file
- `copy_to (other_file)` - Copy to another file

**PLAIN_TEXT_FILE**
- Text file with encoding support
- Inherits from FILE
- `put_string (s)` - Write string
- `put_string_32 (s)` - Write STRING_32
- `put_integer (i)` - Write integer as text
- `put_real (r)` - Write real as text
- `put_boolean (b)` - Write boolean as text
- `put_new_line` - Write newline
- `read_line` - Read a line into last_string
- `read_word` - Read a word
- `read_character` - Read single character
- `read_integer` - Read integer from text
- `read_real` - Read real from text
- `last_string: STRING` - Last line read
- `last_string_32: STRING_32` - Unicode version
- `encoding: ENCODING` - Character encoding
- `set_encoding (enc)` - Set encoding
- `set_utf8_encoding` - Use UTF-8
- `detect_encoding` - Auto-detect BOM
- `put_encoding_bom` - Write BOM

**RAW_FILE**
- Binary file operations
- Inherits from FILE
- `put_data (p, size)` - Write raw bytes
- `read_data (p, size)` - Read raw bytes
- `put_managed_pointer (mp, start, nb)` - Write from MANAGED_POINTER
- `read_to_managed_pointer (mp, start, nb)` - Read to MANAGED_POINTER
- `put_integer_8, put_integer_16, put_integer_32, put_integer_64` - Write integers
- `put_natural_8, ..., put_natural_64` - Write naturals
- `put_real_32, put_real_64` - Write floats
- `read_integer_8, ..., read_integer_64` - Read integers
- `read_natural_8, ..., read_natural_64` - Read naturals
- `read_real_32, read_real_64` - Read floats

**DIRECTORY**
- Directory operations
- `make (dn)` - Create directory object
- `make_with_path (a_path)` - Create with PATH
- `make_open_read (dn)` - Open for reading entries
- `exists: BOOLEAN` - Does directory exist?
- `is_readable, is_writable, is_executable: BOOLEAN` - Permissions
- `create_dir` - Create directory
- `recursive_create_dir` - Create with parents
- `delete` - Delete empty directory
- `recursive_delete` - Delete with contents
- `readentry` - Read next entry
- `lastentry: STRING` - Last entry name
- `last_entry_32: STRING_32` - Unicode version
- `has_entry (name)` - Check for entry
- `start` - Go to first entry
- `close` - Close directory
- `count: INTEGER` - Number of entries
- `linear_representation: ARRAYED_LIST` - All entries as list

**PATH**
- Modern path handling (supports Unicode)
- `make_from_string (s)` - Create from string
- `make_current` - Current directory
- `extended (name)` - Append component
- `parent: PATH` - Parent directory
- `entry: PATH` - Final component
- `name: STRING_32` - Full path string
- `is_empty, is_absolute, is_relative: BOOLEAN` - Path type
- `canonical_path: PATH` - Resolved path
- `same_as (other): BOOLEAN` - Compare paths

**FILE_UTILITIES (expanded)**
- Helper functions (no instance needed)
- `file_exists (n): BOOLEAN` - Check file
- `directory_exists (n): BOOLEAN` - Check directory
- `file_path_exists (p): BOOLEAN` - Check file by PATH
- `directory_path_exists (p): BOOLEAN` - Check directory by PATH
- `copy_file (old_name, new_name)` - Copy file
- `rename_file (old_name, new_name)` - Rename file
- `create_directory (path)` - Create directory
- `create_directory_path (path)` - Create with parents
- `file_names (dir): LIST` - Files in directory
- `directory_names (dir): LIST` - Subdirectories
- `compact_path (path): STRING_32` - Resolve . and ..

**FILE_INFO**
- File metadata
- `size: INTEGER` - File size
- `date: INTEGER` - Modification timestamp
- `access_date: INTEGER` - Access timestamp
- `protection: INTEGER` - Unix permissions
- `owner_name: STRING` - Owner name
- `group_name: STRING` - Group name
- `is_readable, is_writable, is_executable: BOOLEAN`
- `is_plain, is_directory, is_symlink, is_fifo, is_socket: BOOLEAN`

### Step 3: File Open Modes

The FILE class uses mode constants:
- `Closed_file` - File is closed
- `Read_file` - Open for reading only
- `Write_file` - Open for writing only
- `Append_file` - Open for appending
- `Read_write_file` - Open for read and write
- `Append_read_file` - Open for read and append

### Step 4: Path Handling

PATH class is the modern way to handle file paths:
- Supports Unicode filenames
- Cross-platform separator handling
- Path composition without string concatenation
- Canonical path resolution

Example:
```eiffel
local
  l_path: PATH
do
  create l_path.make_from_string ("C:\Users")
  l_path := l_path.extended ("test")
  l_path := l_path.extended ("file.txt")
  -- Result: C:\Users\test\file.txt
end
```

### Step 5: Design Decisions for simple_file

**Simplifications:**
1. **One-liner operations** - Read/write entire file contents
2. **Auto-close** - Functions that open, operate, close
3. **Path-based API** - Accept strings, handle PATH internally
4. **Default UTF-8** - Modern text handling
5. **Error handling** - has_error, last_error pattern
6. **Stream operations** - Append, prepend, clear
7. **Directory helpers** - List files, ensure directory exists

**Key Features:**
- Read entire file to string
- Write string to file (create/overwrite)
- Append to file
- Line-by-line reading
- Binary read/write
- File existence/type checks
- File metadata (size, dates)
- File operations (copy, move, delete)
- Directory operations (create, list, delete)
- Path utilities

### Step 6: API Design

```eiffel
class SIMPLE_FILE

create
  make,                          -- Create for path string
  make_with_path                 -- Create for PATH object

feature -- File Status
  exists: BOOLEAN                -- Does file exist?
  is_readable: BOOLEAN           -- Can read?
  is_writable: BOOLEAN           -- Can write?
  is_executable: BOOLEAN         -- Can execute?
  is_directory: BOOLEAN          -- Is it a directory?
  is_file: BOOLEAN               -- Is it a regular file?
  is_symlink: BOOLEAN            -- Is it a symlink?

feature -- File Metadata
  size: INTEGER_64               -- Size in bytes
  modified_date: DATE_TIME       -- Last modification
  accessed_date: DATE_TIME       -- Last access
  extension: STRING              -- File extension (e.g., "txt")
  base_name: STRING              -- Name without extension
  file_name: STRING              -- Full name with extension
  parent_path: STRING            -- Parent directory path
  path: PATH                     -- Full PATH object

feature -- Read Operations
  read_text: STRING_32           -- Read entire file as text
  read_bytes: ARRAY [NATURAL_8]  -- Read entire file as bytes
  read_lines: LIST [STRING_32]   -- Read all lines
  each_line (action: PROCEDURE)  -- Iterate lines with callback

feature -- Write Operations
  write_text (a_content: READABLE_STRING_GENERAL): BOOLEAN
  write_bytes (a_bytes: ARRAY [NATURAL_8]): BOOLEAN
  write_lines (a_lines: LIST [READABLE_STRING_GENERAL]): BOOLEAN
  append_text (a_content: READABLE_STRING_GENERAL): BOOLEAN
  append_line (a_line: READABLE_STRING_GENERAL): BOOLEAN
  prepend_text (a_content: READABLE_STRING_GENERAL): BOOLEAN
  clear: BOOLEAN                 -- Empty the file

feature -- File Operations
  copy_to (a_destination: READABLE_STRING_GENERAL): BOOLEAN
  move_to (a_destination: READABLE_STRING_GENERAL): BOOLEAN
  rename_to (a_new_name: READABLE_STRING_GENERAL): BOOLEAN
  delete: BOOLEAN
  touch: BOOLEAN                 -- Update modification time
  create_if_missing: BOOLEAN     -- Create empty file

feature -- Directory Operations (when is_directory)
  entries: LIST [STRING_32]      -- All entries
  files: LIST [STRING_32]        -- Only files
  directories: LIST [STRING_32]  -- Only directories
  files_with_extension (ext): LIST [STRING_32]
  create_directory: BOOLEAN
  create_directory_recursive: BOOLEAN
  delete_directory: BOOLEAN
  delete_directory_recursive: BOOLEAN

feature -- Error Handling
  has_error: BOOLEAN
  last_error: STRING

feature -- Encoding
  encoding: ENCODING
  set_encoding (a_encoding: ENCODING)
  set_utf8_encoding

end

-- Convenience class for common operations
expanded class SIMPLE_FILES

feature -- Existence Checks
  file_exists (a_path: READABLE_STRING_GENERAL): BOOLEAN
  directory_exists (a_path: READABLE_STRING_GENERAL): BOOLEAN

feature -- Quick Read
  read_text (a_path: READABLE_STRING_GENERAL): STRING_32
  read_lines (a_path: READABLE_STRING_GENERAL): LIST [STRING_32]
  read_bytes (a_path: READABLE_STRING_GENERAL): ARRAY [NATURAL_8]

feature -- Quick Write
  write_text (a_path, a_content: READABLE_STRING_GENERAL): BOOLEAN
  write_lines (a_path: READABLE_STRING_GENERAL; a_lines: LIST): BOOLEAN
  write_bytes (a_path: READABLE_STRING_GENERAL; a_bytes: ARRAY): BOOLEAN
  append_text (a_path, a_content: READABLE_STRING_GENERAL): BOOLEAN

feature -- Quick Operations
  copy_file (a_source, a_dest: READABLE_STRING_GENERAL): BOOLEAN
  move_file (a_source, a_dest: READABLE_STRING_GENERAL): BOOLEAN
  delete_file (a_path: READABLE_STRING_GENERAL): BOOLEAN
  ensure_directory (a_path: READABLE_STRING_GENERAL): BOOLEAN

feature -- Directory Listing
  list_files (a_path: READABLE_STRING_GENERAL): LIST [STRING_32]
  list_directories (a_path: READABLE_STRING_GENERAL): LIST [STRING_32]
  list_all (a_path: READABLE_STRING_GENERAL): LIST [STRING_32]

feature -- Path Utilities
  join_path (parts: ARRAY [READABLE_STRING_GENERAL]): STRING_32
  parent_of (a_path: READABLE_STRING_GENERAL): STRING_32
  extension_of (a_path: READABLE_STRING_GENERAL): STRING_32
  base_name_of (a_path: READABLE_STRING_GENERAL): STRING_32

feature -- Temporary Files
  temp_file: STRING_32           -- Create and return temp file path
  temp_directory: STRING_32      -- Get temp directory path

end

-- Path builder for fluent API
class SIMPLE_PATH

create
  make,                          -- Empty path
  make_from (a_path)             -- From string
  make_current,                  -- Current directory
  make_temp                      -- Temp directory

feature -- Building
  add (a_component): like Current
  up: like Current               -- Go to parent
  with_extension (ext): like Current

feature -- Access
  to_string: STRING_32
  to_path: PATH
  exists: BOOLEAN
  is_file: BOOLEAN
  is_directory: BOOLEAN
  is_absolute: BOOLEAN
  parent: like Current
  file_name: STRING_32
  extension: STRING_32

end
```

### Step 7: Dependencies

**ISE Libraries:**
- `$ISE_LIBRARY/library/base/base.ecf`
- `$ISE_LIBRARY/library/time/time.ecf` (for DATE_TIME conversion)

**Key Classes Used:**
- FILE, PLAIN_TEXT_FILE, RAW_FILE - File I/O
- DIRECTORY - Directory operations
- PATH - Path handling
- FILE_INFO - File metadata
- FILE_UTILITIES - Helper functions (expanded class)
- ENCODING - Character encoding

---

## Implementation Summary

**Files to Create:**
- `simple_file.ecf` - ECF configuration
- `src/simple_file.e` - Main file wrapper class
- `src/simple_files.e` - Expanded convenience class
- `src/simple_path.e` - Path builder class
- `testing/file_test_app.e` - Test application

**Test Strategy:**
Tests can run fully without external dependencies:
1. Create temp files for testing
2. Write content, read back, verify
3. Test directory operations in temp location
4. Clean up temp files after tests

---

## Notes

1. **No External Dependencies:** Unlike simple_mongo, simple_file has no external database requirements. All testing can be done with temporary files.

2. **Unicode Support:** PATH and STRING_32 provide full Unicode filename support. The wrapper should use STRING_32 for all path/filename operations.

3. **Encoding:** PLAIN_TEXT_FILE supports various encodings with BOM detection. Default to UTF-8 for modern compatibility.

4. **Error Handling:** File operations can fail for many reasons (permissions, disk full, file in use). Capture errors in has_error/last_error pattern.

5. **Expanded SIMPLE_FILES:** An expanded class provides static-like convenience methods without needing to create instances. Useful for quick one-liner operations.

6. **Path Building:** The SIMPLE_PATH fluent builder pattern simplifies path construction without string concatenation.

7. **Platform Differences:** Path separators differ between Windows (\) and Unix (/). The PATH class handles this automatically.

---

## Competitive Analysis: File APIs in Other Languages

### Python (pathlib)

**Key Design Patterns:**
- Object-oriented paths as first-class objects (not strings)
- `Path` objects with chained methods: `Path("/home") / "user" / "file.txt"`
- Direct read/write methods on Path: `path.read_text()`, `path.write_text(content)`
- Cross-platform abstraction (PurePosixPath, PureWindowsPath)
- Iteration: `path.iterdir()` returns generator of Path objects

**Best Features to Adopt:**
- One-liner read/write: `content = Path("file.txt").read_text()`
- Path operator `/` for joining (Eiffel: use fluent `add()` method)
- `path.stem` (name without extension), `path.suffix` (extension)
- `path.resolve()` for canonical absolute path
- `path.with_suffix(".new")` to change extension

**Sources:** [Python pathlib docs](https://docs.python.org/3/library/pathlib.html), [Real Python pathlib guide](https://realpython.com/python-pathlib/)

### Node.js (fs/promises)

**Key Design Patterns:**
- Promise-based async API: `await fs.readFile(path)`
- Separate sync and async methods for flexibility
- FileHandle API for multiple operations on open file
- Buffer/string flexibility for binary vs text

**Best Features to Adopt:**
- `fs.readFile(path, 'utf8')` - encoding as parameter
- `fs.writeFile(path, data, { flag: 'a' })` - options object pattern
- `fs.mkdir(path, { recursive: true })` - ensure parents
- `fs.rm(path, { recursive: true, force: true })` - safe delete

**Sources:** [Node.js fs docs](https://nodejs.org/api/fs.html), [Modern Node.js patterns](https://kashw1n.com/blog/nodejs-2025/)

### Go (os package)

**Key Design Patterns:**
- Explicit error handling: `data, err := os.ReadFile(path)`
- `defer file.Close()` pattern for cleanup
- Buffered I/O via `bufio` for efficiency
- Platform-independent interface

**Best Features to Adopt:**
- Simple file permissions: `0644`, `0755` constants
- `os.MkdirAll` for recursive directory creation
- Clear separation of concerns (os vs io vs bufio)
- Explicit resource cleanup pattern

**Sources:** [Go os package](https://pkg.go.dev/os), [Go best practices](https://dave.cheney.net/practical-go/presentations/qcon-china.html)

### Ruby (File/IO)

**Key Design Patterns:**
- Block-based file handling: `File.open(path) { |f| f.read }`
- Automatic resource cleanup with blocks
- Rich enumeration: `File.readlines`, `File.foreach`
- Multiple approaches for same operation (choice)

**Best Features to Adopt:**
- `File.read(path)` / `File.write(path, content)` one-liners
- Block iteration for memory efficiency
- `File.join("path", "to", "file")` for cross-platform joining
- Rich File.* class methods for operations

**Sources:** [Ruby File docs](https://docs.ruby-lang.org/en/master/IO.html), [Better Stack Ruby guide](https://betterstack.com/community/guides/scaling-ruby/files-in-ruby/)

### Rust (std::fs)

**Key Design Patterns:**
- Result type for all operations (explicit error handling)
- Platform-specific extension traits
- Ownership model for file handles
- `std::path::Path` and `PathBuf` distinction

**Pain Points to Avoid:**
- `canonicalize()` requires file to exist - confusing
- Complex borrow checker interactions with paths
- Many small API decisions that don't compose well

**Sources:** [Rust std::fs](https://doc.rust-lang.org/std/fs/), [Tangram Vision blog](https://www.tangramvision.com/blog/building-robust-filesystem-interactions-in-rust)

---

## Common Developer Pain Points to Address

### 1. Encoding Hell
- Files opened without explicit encoding cause mojibake
- BOM detection inconsistent across platforms
- **Solution:** Default to UTF-8, auto-detect BOM, always use STRING_32

### 2. Resource Leaks
- Forgetting to close files after operations
- Exception paths that skip cleanup
- **Solution:** Auto-closing operations, block-style patterns in Eiffel (agents)

### 3. Path String Manipulation
- Manual string concatenation breaks cross-platform
- Trailing slash inconsistency
- **Solution:** Use PATH class internally, normalize on construction

### 4. Error Swallowing
- Silent failures when files don't exist
- Unclear why operation failed
- **Solution:** Consistent has_error/last_error, specific error types

### 5. Directory Creation Race Conditions
- `if not exists then create` can fail in concurrent code
- **Solution:** `ensure_directory` that handles already-exists gracefully

### 6. Large File Memory Issues
- Reading entire large file into memory
- **Solution:** Streaming API, line-by-line iteration, chunked reading

### 7. Permission Confusion
- Different permission models on Windows vs Unix
- **Solution:** Simple permission helpers: `make_readable`, `make_writable`

### 8. Path Canonicalization Gotchas
- Can't canonicalize path that doesn't exist yet
- Symlink resolution unexpected behavior
- **Solution:** Separate `normalize` (pure) from `resolve` (requires existence)

### 9. Temp File Cleanup
- Temp files left behind after crashes
- **Solution:** Auto-cleanup option, explicit `with_temp_file` pattern

### 10. File Locking Issues
- Cannot delete/move file that's open elsewhere
- **Solution:** Clear error messages, `try_delete` that doesn't throw

---

## Design Recommendations Based on Research

1. **Follow Python's pathlib pattern** for object-oriented paths with fluent API
2. **Provide both object and expanded class APIs** (like Ruby's dual approach)
3. **Default to UTF-8** with explicit encoding options when needed
4. **Auto-close pattern** for all one-liner operations
5. **Explicit error handling** via has_error/last_error (not exceptions)
6. **Streaming support** for large file handling
7. **Recursive options** for directory operations (create, delete, list)
8. **Cross-platform abstraction** hiding Windows/Unix differences
9. **Fluent path building** without operator overloading (use methods)
10. **Temp file support** with automatic cleanup options

