# simple_compression Research Report

**Date:** 2025-12-08
**Library:** simple_compression (Data Compression Wrapper for ISE zlib)

---

## Step 1: Specifications Research

### Compression Format Standards

| Format | RFC | Description |
|--------|-----|-------------|
| **DEFLATE** | RFC 1951 | Lossless compression using LZ77 + Huffman coding |
| **zlib** | RFC 1950 | DEFLATE with header and ADLER-32 checksum |
| **gzip** | RFC 1952 | DEFLATE with gzip file format headers |

### RFC 1951 - DEFLATE Compressed Data Format Specification (Version 1.3)

DEFLATE is a lossless data compression format designed by Phil Katz for PKZIP version 2. It uses a combination of the LZ77 algorithm and Huffman coding, with efficiency comparable to the best currently available general-purpose compression methods.

**Key Characteristics:**
- Worst case expansion: 5 bytes per 32K-byte block (0.015% increase)
- English text typically compresses 2.5x to 3x
- Executable files compress somewhat less
- Graphical data (raster images) may compress much more
- Designed to be implementable with bounded intermediate storage

### RFC 1950 - ZLIB Compressed Data Format Specification (Version 3.3)

The zlib format wraps DEFLATE compressed data with:
- **CMF byte**: Compression Method and flags (CM = 8 for DEFLATE with 32K window)
- **ADLER-32 checksum**: For data integrity verification
- Can handle arbitrarily long sequentially presented input data streams
- Used by gzip and PNG formats

### RFC 1952 - GZIP File Format

GZIP uses DEFLATE compression with additional metadata:
- File name, modification time, OS type
- CRC32 checksum (different from zlib's ADLER-32)
- Multi-member support (concatenated gzip streams)
- Compatible with GNU gzip utility

**Compression Performance:**
- Phil Katz designed the deflate format
- Jean-Loup Gailly and Mark Adler wrote the zlib implementation
- Starting with POWER9, IBM added hardware DEFLATE support to NX accelerator

**Sources:**
- [RFC 1951 - DEFLATE Specification](https://www.rfc-editor.org/rfc/rfc1951)
- [RFC 1950 - ZLIB Specification](https://datatracker.ietf.org/doc/html/rfc1950.html)
- [RFC 1951 at W3C](https://www.w3.org/Graphics/PNG/RFC-1951)
- [Deflate - Wikipedia](https://en.wikipedia.org/wiki/Deflate)

---

## Step 2: Tech-Stack Library Analysis

### Go: compress/gzip and compress/zlib (Standard Library)

**Architecture:**
- `compress/flate`: Core DEFLATE implementation (RFC 1951)
- `compress/zlib`: ZLIB format wrapper (RFC 1950)
- `compress/gzip`: GZIP format wrapper (RFC 1952)

**API Pattern - Simple Compression:**
```go
// Compress data
var buf bytes.Buffer
w := gzip.NewWriter(&buf)
w.Write([]byte("Hello, World!"))
w.Close()
compressed := buf.Bytes()

// Decompress data
r, _ := gzip.NewReader(bytes.NewReader(compressed))
decompressed, _ := io.ReadAll(r)
```

**API Pattern - File Operations:**
```go
// Compress file
inFile, _ := os.Open("input.txt")
outFile, _ := os.Create("output.gz")
gzWriter := gzip.NewWriter(outFile)
io.Copy(gzWriter, inFile)
gzWriter.Close()

// Decompress file
gzFile, _ := os.Open("file.gz")
gzReader, _ := gzip.NewReader(gzFile)
io.Copy(os.Stdout, gzReader)
```

**Compression Levels:**
- `NoCompression` (0): No compression, just framing
- `BestSpeed` (1): Fastest compression
- `DefaultCompression` (-1): Level 6
- `BestCompression` (9): Highest compression ratio
- `HuffmanOnly` (-2): Huffman coding only, no LZ77

**Key Features:**
- Reader/Writer interface integration
- Streaming support via io.Reader/io.Writer
- Automatic flush control
- Dictionary support for zlib
- Reset() for writer reuse without reallocation

**Alternative: klauspost/compress**
- Drop-in replacement with optimized performance
- 2-3x faster than standard library
- Same API, better throughput

**Sources:**
- [compress/gzip Package](https://pkg.go.dev/compress/gzip)
- [compress/zlib Package](https://pkg.go.dev/compress/zlib)
- [compress/flate Package](https://pkg.go.dev/compress/flate)
- [klauspost/compress GitHub](https://github.com/klauspost/compress)

### Rust: flate2 (DEFLATE, gzip, zlib bindings)

**Backend Options:**
- **miniz_oxide** (default): Pure Rust, safe, portable
- **zlib-rs**: Highest performance, outperforms C implementations
- **zlib-ng**: C library, high performance
- **zlib**: Standard C library

**Module Architecture:**
- `flate2::read`: Types working on `io::Read`
- `flate2::write`: Types working on `io::Write`
- `flate2::bufread`: Types working on `io::BufRead`

**API Pattern - Zlib Compression:**
```rust
use flate2::Compression;
use flate2::write::ZlibEncoder;
use std::io::prelude::*;

let mut encoder = ZlibEncoder::new(Vec::new(), Compression::default());
encoder.write_all(b"Hello, World!")?;
let compressed = encoder.finish()?;
```

**API Pattern - Gzip Decompression:**
```rust
use flate2::read::GzDecoder;
use std::io::prelude::*;

let mut decoder = GzDecoder::new(&compressed_data[..]);
let mut decompressed = String::new();
decoder.read_to_string(&mut decompressed)?;
```

**Compression Levels:**
- `Compression::none()`: No compression (level 0)
- `Compression::fast()`: Fastest (level 1)
- `Compression::default()`: Balanced (level 6)
- `Compression::best()`: Highest ratio (level 9)
- `Compression::new(n)`: Custom level 0-9

**Key Features:**
- Three parallel APIs (read/write/bufread) for flexibility
- Pure Rust default backend (no C dependencies)
- Optional high-performance backends
- Supports zlib, gzip, and raw DEFLATE streams
- Automatic format detection in some modes

**Sources:**
- [flate2 crate documentation](https://docs.rs/flate2)
- [flate2 GitHub](https://github.com/rust-lang/flate2-rs)
- [flate2 on crates.io](https://crates.io/crates/flate2)

### Python: zlib and gzip Modules

**zlib Module (Low-Level):**
```python
import zlib

# Simple compress/decompress
compressed = zlib.compress(b"Hello, World!", level=6)
decompressed = zlib.decompress(compressed)

# Streaming compression
compressor = zlib.compressobj(level=9)
chunk1 = compressor.compress(b"first chunk")
chunk2 = compressor.compress(b"second chunk")
final = compressor.flush()
```

**gzip Module (File-Oriented):**
```python
import gzip

# Write compressed file
with gzip.open('file.gz', 'wt') as f:
    f.write("Hello, World!")

# Read compressed file
with gzip.open('file.gz', 'rt') as f:
    content = f.read()

# Compress bytes
compressed = gzip.compress(b"data", compresslevel=6)
decompressed = gzip.decompress(compressed)
```

**Compression Levels:**
- Level -1: Default (level 6)
- Level 0: No compression
- Level 1: Fastest (BestSpeed)
- Level 9: Best compression (BestCompression)

**Key Features:**
- Simple one-function API for basic use: `compress()`, `decompress()`
- Streaming API for large data: `compressobj()`, `decompressobj()`
- File-like interface with context managers
- Multi-member gzip support (concatenated streams)
- CRC32 and ADLER32 checksum functions
- Recent optimization: Python 3.11+ uses one-shot compression for speed

**Performance Alternative: python-isal**
- Intel ISA-L bindings for faster compression
- Drop-in replacement for zlib/gzip
- 2-5x faster than standard library

**Sources:**
- [Python zlib module](https://docs.python.org/3/library/zlib.html)
- [Python gzip module](https://docs.python.org/3/library/gzip.html)
- [StackAbuse zlib tutorial](https://stackabuse.com/python-zlib-library-tutorial/)
- [python-isal GitHub](https://github.com/pycompression/python-isal)

### Node.js: zlib Module

**Stream-Based Architecture:**
```javascript
const zlib = require('zlib');
const fs = require('fs');

// File compression (streaming)
const gzip = zlib.createGzip();
const input = fs.createReadStream('input.txt');
const output = fs.createWriteStream('input.txt.gz');
input.pipe(gzip).pipe(output);

// Buffer compression
const input = Buffer.from('Hello, World!');
zlib.gzip(input, (err, compressed) => {
    zlib.gunzip(compressed, (err, decompressed) => {
        console.log(decompressed.toString());
    });
});

// Promise-based (modern)
const { promisify } = require('util');
const gzip = promisify(zlib.gzip);
const compressed = await gzip(buffer);
```

**Compression Methods:**
- `createGzip()` / `gzip()`: Gzip compression
- `createGunzip()` / `gunzip()`: Gzip decompression
- `createDeflate()` / `deflate()`: Zlib compression
- `createInflate()` / `inflate()`: Zlib decompression
- `createDeflateRaw()` / `deflateRaw()`: Raw DEFLATE
- `createBrotliCompress()`: Brotli compression (newer)

**Options:**
- `level`: 0-9 (default: `zlib.constants.Z_DEFAULT_COMPRESSION`)
- `windowBits`: Memory usage control
- `memLevel`: Memory/speed trade-off
- `strategy`: `Z_DEFAULT_STRATEGY`, `Z_FILTERED`, `Z_HUFFMAN_ONLY`, `Z_RLE`, `Z_FIXED`

**Key Features:**
- Built-in core module (no external dependencies)
- Stream API integration with Node.js streams
- HTTP compression support via Accept-Encoding headers
- Caching recommended due to CPU cost
- Newer Brotli support for better compression ratios

**Sources:**
- [Node.js zlib documentation](https://nodejs.org/api/zlib.html)
- [GeeksforGeeks Node.js zlib](https://www.geeksforgeeks.org/node-js/nodejs-zlib-module/)
- [BlackSlate Data Compression in Node.js](https://www.blackslate.io/articles/data-compression-in-nodejs-using-zlib)

---

## Step 3: Developer Pain Points

### Common Compression Frustrations

**1. Algorithm Selection Paralysis**
- **BZIP2**: Good compression ratio but slow (5-10x slower than gzip)
- **LZO**: Fast but poor compression on random data, limited use cases
- **Zlib/gzip**: Balanced but not optimized for specific workloads
- **LZ4**: Extremely fast (500+ MB/s) but lower compression ratio
- **Zstandard**: Best modern choice but not universally available

Problem: Developers struggle to choose the right algorithm without benchmarking their specific data.

**2. Buffer Sizing Nightmares**
- "How big should my output buffer be?"
- Worst case: 0.015% expansion for DEFLATE (5 bytes per 32KB)
- Common mistake: Allocating input_size buffer, which fails on incompressible data
- Best practice: Use `compressBound()` or similar to get safe upper limit
- Stream-based APIs avoid this but have performance overhead

**3. Memory Management Complexity**
- Frequent allocation/deallocation kills performance
- Stream-based implementations are slow due to resizing
- Pre-allocation is faster but wastes memory
- Finding the balance: Reuse buffers, shrink if needed
- Memory leak risks: Forgetting `inflateEnd()` or similar cleanup

**4. Error Handling Confusion**
- `Z_BUF_ERROR`: Not fatal, but what to do? (call again with more space)
- `Z_DATA_ERROR`: Corrupted data or wrong format?
- `Z_MEM_ERROR`: Out of memory, but when? (deferred allocation in inflate)
- `Z_NEED_DICT`: Dictionary required, but which one?
- Many developers don't handle errors beyond "it failed"

**5. Compression Level Selection**
- Level 1: Fast but poor compression
- Level 9: Great compression but 10-100x slower
- Default (level 6): Often not optimal for specific use case
- Real-time apps need level 1-3
- Archive apps need level 9
- Web APIs often use level 6 without benchmarking

**6. Stream vs Buffer Mode Confusion**
- When to use stream API vs one-shot buffer API?
- Stream: Large data, real-time, network
- Buffer: Small data, all available at once
- Performance: Buffer mode is faster for small data
- Flexibility: Stream mode handles any size

**7. Security Vulnerabilities**
- **Compression bombs**: Tiny compressed input expands to gigabytes
- Solution: Validate compressed size, set memory limits
- **CRC/checksum validation**: Often skipped, leading to silent corruption
- **Untrusted input**: Malformed data can crash or DoS
- Best practice: Always validate checksums, limit decompression size

**Sources:**
- [Meta OpenZL compression framework](https://engineering.fb.com/2025/10/06/developer-tools/openzl-open-source-format-aware-compression-framework/)
- [NCBI C++ Toolkit compression](https://ncbi.github.io/cxx-toolkit/pages/ch_compress)
- [Jellyfish Developer Pain Points](https://jellyfish.co/library/developer-productivity/pain-points/)

---

## Step 4: Innovation Opportunities

### What Would Make Developers' Lives Easier?

**1. Simple String/Bytes Compression (80% Use Case)**
```eiffel
-- Just compress a string, don't make me think
compressed := compression.compress_string ("Hello, World!")
decompressed := compression.decompress_string (compressed)

-- Or bytes
compressed_bytes := compression.compress (data_bytes)
```

**2. Safe Buffer Allocation**
```eiffel
-- Automatic safe buffer sizing
create compressor.make
compressed := compressor.compress (input_data)
-- No need to calculate buffer size, library handles it
```

**3. Sensible Defaults with Easy Overrides**
```eiffel
-- Default: balanced compression (level 6)
compressed := compression.compress (data)

-- Fast compression for real-time
compressed := compression.compress_fast (data)  -- level 1

-- Maximum compression for archives
compressed := compression.compress_best (data)  -- level 9

-- Custom level
create compressor.make_with_level (5)
compressed := compressor.compress (data)
```

**4. Gzip File Operations Made Simple**
```eiffel
-- Write gzip file
compression.compress_file ("input.txt", "output.gz")

-- Read gzip file
content := compression.decompress_file ("data.gz")

-- Stream large files
create reader.make ("large_file.gz")
across reader.lines as line loop
    process (line.item)
end
```

**5. Stream Compression for Large Data**
```eiffel
-- Stream API for large data
create stream.make_compress (output_file, level)
stream.write (chunk1)
stream.write (chunk2)
stream.write (chunk3)
stream.close

-- Decompress stream
create reader.make_decompress (input_file)
from reader.start until reader.after loop
    chunk := reader.read_chunk (8192)
    process (chunk)
    reader.forth
end
```

**6. Error Handling with Eiffel Contracts**
```eiffel
compress (data: ARRAY [NATURAL_8]): ARRAY [NATURAL_8]
    require
        data_not_empty: data.count > 0
    ensure
        compressed_valid: Result.count > 0
        -- In worst case, compressed <= original + 0.015%
        compressed_bounded: Result.count <= data.count + (data.count // 6553) + 5
    rescue
        -- Handle Z_MEM_ERROR, Z_BUF_ERROR
    end
```

**7. Format Detection and Validation**
```eiffel
-- Auto-detect format
if compression.is_gzip (data) then
    decompressed := compression.decompress_gzip (data)
elseif compression.is_zlib (data) then
    decompressed := compression.decompress_zlib (data)
end

-- Validate before decompressing
if compression.validate_checksum (compressed_data) then
    decompressed := compression.decompress (compressed_data)
else
    -- Handle corrupted data
end
```

**8. Memory-Conscious API**
```eiffel
-- Reusable compressor (avoid reallocation)
create compressor.make
compressed1 := compressor.compress (data1)
compressor.reset  -- Reuse without reallocating
compressed2 := compressor.compress (data2)
```

**9. Compression Statistics**
```eiffel
create compressor.make
compressed := compressor.compress (data)
ratio := compressor.compression_ratio
io.put_string ("Compressed " + data.count.out + " bytes to " +
               compressed.count.out + " bytes (" +
               ratio.out + "x compression)")
```

---

## Step 5: Design Strategy Synthesis

### Core Design Principles

**1. Simple by Default, Powerful When Needed**
- One-line compression for simple cases
- Advanced features available but not required
- Progressive disclosure: Don't expose complexity upfront

**2. Safety First**
- Automatic buffer sizing (use compressBound internally)
- Checksum validation by default
- Protection against compression bombs
- Clear error messages

**3. Eiffel Philosophy**
- Use contracts for validation
- Type-safe interfaces
- No void safety violations
- Resource management via creation/disposal

**4. Work Within ISE Constraints**
- Wrap ISE zlib library, don't reinvent
- Handle external DLL requirement gracefully
- Provide clear error if zlibwapi.dll missing
- Document DLL deployment

### What simple_compression Should Be

A lightweight, safe, and simple wrapper around ISE's zlib library that makes compression easy for 80% of use cases while providing access to advanced features for the remaining 20%.

**Target Use Cases:**
1. Compress/decompress strings and byte arrays
2. Read/write gzip files
3. HTTP response compression
4. Data storage compression
5. Simple archive creation

### What simple_compression Should NOT Be

- A full archive manager (like ZIP with multiple entries) - that's simple_archive
- A cryptographic library - that's simple_encryption
- A network protocol implementation
- A replacement for ISE zlib - it's a wrapper

### API Surface Design

**Minimal API (Core Features):**
```eiffel
class SIMPLE_COMPRESSION

feature -- Basic Compression

    compress_string (input: STRING): STRING
        -- Compress string, return compressed data as base64 or hex

    decompress_string (input: STRING): STRING
        -- Decompress string from base64/hex

    compress (input: ARRAY [NATURAL_8]): ARRAY [NATURAL_8]
        -- Compress byte array

    decompress (input: ARRAY [NATURAL_8]): ARRAY [NATURAL_8]
        -- Decompress byte array

feature -- File Operations

    compress_file (source_path, dest_path: STRING)
        -- Compress file to gzip format

    decompress_file (source_path, dest_path: STRING)
        -- Decompress gzip file

    read_compressed_file (path: STRING): STRING
        -- Read and decompress gzip file to string

feature -- Level Selection

    set_level (level: INTEGER)
        -- Set compression level (1=fast, 6=default, 9=best)
        require
            valid_level: level >= 0 and level <= 9

feature -- Status

    last_error: detachable STRING
        -- Last error message if operation failed

    compression_ratio: REAL
        -- Ratio of last compression operation
```

**Extended API (Advanced Features):**
```eiffel
feature -- Stream Compression

    create_compressor (output_path: STRING; level: INTEGER)
        -- Create streaming compressor for large files

    write_chunk (data: ARRAY [NATURAL_8])
        -- Write chunk to compressor

    close_compressor
        -- Flush and close compressor

feature -- Format Detection

    is_gzip_format (data: ARRAY [NATURAL_8]): BOOLEAN
        -- Check if data is gzip format (magic bytes 1F 8B)

    is_zlib_format (data: ARRAY [NATURAL_8]): BOOLEAN
        -- Check if data is zlib format

feature -- Validation

    validate_gzip (data: ARRAY [NATURAL_8]): BOOLEAN
        -- Validate gzip CRC32 checksum

    calculate_crc32 (data: ARRAY [NATURAL_8]): NATURAL_32
        -- Calculate CRC32 checksum

feature -- Advanced Options

    compress_with_options (
        input: ARRAY [NATURAL_8];
        level: INTEGER;
        window_bits: INTEGER): ARRAY [NATURAL_8]
        -- Compress with custom window size
```

---

## Step 6: Gap Analysis - ISE Zlib Library

### Understanding the ISE Foundation

**Location:** `$ISE_LIBRARY/unstable/library/compression/zlib/`

**External Dependency:**
- Requires zlibwapi.dll on Windows
- Requires libz.so on Linux
- Must be deployed with application

**ISE Classes Available:**

1. **ZLIB_COMPRESS** - Compression operations
   - `compress()`: Compress byte array
   - `compress_bound()`: Calculate maximum compressed size
   - Support for compression levels

2. **ZLIB_UNCOMPRESS** - Decompression operations
   - `uncompress()`: Decompress byte array
   - Error handling for corrupted data

3. **GZIP_FILE_READER** - Read gzip files
   - `gzopen()`: Open gzip file
   - `gzread()`: Read compressed data
   - `gzclose()`: Close file

4. **GZIP_FILE_WRITER** - Write gzip files
   - `gzopen()`: Create gzip file
   - `gzwrite()`: Write data
   - `gzclose()`: Close and finalize

5. **DEFLATE_STREAM** / **INFLATE_STREAM** - Low-level streaming
   - `deflateInit()`, `deflate()`, `deflateEnd()`
   - `inflateInit()`, `inflate()`, `inflateEnd()`
   - Fine-grained control over compression

### What ISE Provides

- Low-level wrappers around zlib C functions
- Basic compress/uncompress functions
- Gzip file I/O
- Stream API for advanced use
- Compression level control
- Error codes from zlib

### What ISE Does NOT Provide

- High-level string compression
- Automatic buffer sizing
- Format detection (gzip vs zlib vs raw)
- CRC32 validation helpers
- Easy file compression (source -> dest)
- Compression statistics
- Memory-safe abstractions
- Error messages (just error codes)

### Technical Challenges

**1. DLL Management**
- Application must distribute zlibwapi.dll
- DLL must be in PATH or application directory
- Version compatibility issues
- Clear error message needed if DLL missing

**2. Buffer Management**
- Need to call `compress_bound()` before compression
- Output buffer must be pre-allocated
- Risk of buffer overflow if sized wrong
- Need safe wrapper that handles this

**3. Error Handling**
- ISE returns integer error codes
- Need to translate to meaningful messages:
  - Z_OK (0): Success
  - Z_MEM_ERROR (-4): Out of memory
  - Z_BUF_ERROR (-5): Buffer too small
  - Z_DATA_ERROR (-3): Corrupted data
  - Z_STREAM_ERROR (-2): Invalid parameters

**4. Memory Leaks**
- Stream API requires explicit `End()` calls
- Need ensure cleanup via Eiffel disposal

**5. String Encoding**
- ISE works with NATURAL_8 arrays (bytes)
- Need conversion to/from STRING
- UTF-8 encoding considerations

---

## Step 7: Implementation Recommendations

### Phase 1: Essential Features (MVP)

**Priority 1: Basic Compression**
1. `compress(ARRAY[NATURAL_8]): ARRAY[NATURAL_8]`
   - Wrap ISE `ZLIB_COMPRESS`
   - Automatic buffer sizing using `compress_bound()`
   - Default level 6
   - Clear error messages

2. `decompress(ARRAY[NATURAL_8]): ARRAY[NATURAL_8]`
   - Wrap ISE `ZLIB_UNCOMPRESS`
   - Handle Z_BUF_ERROR by increasing buffer size
   - Limit maximum decompression size (prevent bombs)

3. `compress_string(STRING): STRING` and `decompress_string(STRING): STRING`
   - Convert STRING to UTF-8 byte array
   - Compress/decompress
   - Encode result as base64 for safe string storage

**Priority 2: Level Control**
4. `set_level(INTEGER)` with constants
   - `compression_level_fast = 1`
   - `compression_level_default = 6`
   - `compression_level_best = 9`

5. `compress_fast()` and `compress_best()` convenience methods

**Priority 3: File Operations**
6. `compress_file(source, dest: STRING)`
   - Read source file
   - Compress
   - Write to gzip file using ISE `GZIP_FILE_WRITER`

7. `decompress_file(source, dest: STRING)`
   - Read gzip file using ISE `GZIP_FILE_READER`
   - Decompress
   - Write to destination

### Phase 2: Enhanced Features

**Priority 4: Format Detection**
8. `is_gzip_format(data): BOOLEAN`
   - Check for magic bytes: 1F 8B

9. `is_zlib_format(data): BOOLEAN`
   - Check for zlib header: 78 9C (default) or 78 DA (best)

**Priority 5: Validation**
10. `validate_gzip(data): BOOLEAN`
    - Parse gzip footer
    - Extract CRC32
    - Compare with calculated CRC32

11. `calculate_crc32(data): NATURAL_32`
    - Use ISE zlib CRC32 function

**Priority 6: Statistics**
12. `last_input_size`, `last_output_size`, `compression_ratio`
    - Track compression effectiveness
    - Useful for debugging and optimization

### Phase 3: Advanced Features (Future)

**Priority 7: Streaming**
13. Stream compression for large files
    - `GZIP_STREAM_WRITER` wrapper
    - Chunk-based processing
    - Memory-efficient for GB-size files

14. Stream decompression
    - `GZIP_STREAM_READER` wrapper
    - Iterator-style interface

**Priority 8: Advanced Options**
15. Window bits control (memory vs compression trade-off)
16. Strategy selection (filtered, huffman-only, RLE)
17. Dictionary support for repetitive data

### Error Handling Strategy

```eiffel
class COMPRESSION_ERROR
    inherit
        EXCEPTION

feature
    error_code: INTEGER
        -- Zlib error code

    error_message: STRING
        -- Human-readable message

feature {NONE}

    make_from_zlib_code (code: INTEGER)
        do
            error_code := code
            error_message := error_name (code)
        end

    error_name (code: INTEGER): STRING
        do
            inspect code
            when z_ok then Result := "Success"
            when z_mem_error then Result := "Out of memory"
            when z_buf_error then Result := "Buffer too small"
            when z_data_error then Result := "Corrupted or invalid compressed data"
            when z_stream_error then Result := "Invalid parameters or stream state"
            else Result := "Unknown error: " + code.out
            end
        end
end
```

### DLL Deployment Documentation

**README.md section:**
```markdown
## Requirements

simple_compression requires the zlib library:

**Windows:**
- zlibwapi.dll must be in your application directory or system PATH
- Download from: https://www.zlib.net/
- Or use the version from $ISE_LIBRARY/studio/spec/win64/bin/

**Linux:**
- libz.so (usually pre-installed)
- If missing: `sudo apt-get install zlib1g` (Debian/Ubuntu)
- Or: `sudo yum install zlib` (RedHat/CentOS)

**Error Message:**
If you see "Cannot load zlib library", ensure the DLL/SO is available.
```

### Memory Safety Approach

```eiffel
compress (input: ARRAY [NATURAL_8]): ARRAY [NATURAL_8]
    local
        bound: INTEGER
        output: ARRAY [NATURAL_8]
        compressor: ZLIB_COMPRESS
    do
        create compressor

        -- Safe buffer allocation
        bound := compressor.compress_bound (input.count)
        create output.make_filled (0, 1, bound)

        -- Perform compression
        compressor.compress (output, input, compression_level_default)

        if compressor.last_operation_successful then
            -- Trim to actual size
            Result := output.subarray (1, compressor.compressed_size)
        else
            -- Raise exception with clear message
            create {COMPRESSION_ERROR}.make_from_zlib_code (compressor.last_error_code)
            raise
        end
    ensure
        compressed: Result /= Void
        not_empty: Result.count > 0
    rescue
        if attached {COMPRESSION_ERROR} exception_manager.last_exception as e then
            last_error := e.error_message
        end
    end
```

---

## Conclusion

The simple_compression library should provide a clean, safe wrapper around ISE's zlib library that addresses common developer pain points:

**Must Have:**
- Simple string and byte array compression/decompression
- Automatic safe buffer allocation
- Clear error messages (not just codes)
- Sensible defaults with easy level selection
- Gzip file operations

**Should Have:**
- Format detection and validation
- CRC32 checksum helpers
- Compression statistics
- Protection against compression bombs

**Nice to Have:**
- Streaming API for large files
- Advanced zlib options (window bits, strategy)
- Dictionary compression

The key to success is hiding the complexity of the underlying ISE zlib wrapper while preserving safety and performance. The library should make the common case (compress a string, write a gzip file) trivial while still allowing access to advanced features when needed.

**Critical Success Factors:**
1. Handle DLL dependency gracefully with clear error messages
2. Never crash on corrupted input (catch all zlib errors)
3. Prevent memory leaks (ensure cleanup of streams)
4. Protect against compression bombs (limit decompression size)
5. Provide excellent documentation with examples

By following modern API design patterns from Go, Rust, Python, and Node.js, simple_compression can provide an Eiffel-idiomatic interface that feels natural to developers familiar with compression in other languages.

---

## Sources Summary

**Specifications:**
- [RFC 1951 - DEFLATE Specification](https://www.rfc-editor.org/rfc/rfc1951)
- [RFC 1950 - ZLIB Specification](https://datatracker.ietf.org/doc/html/rfc1950.html)
- [Deflate - Wikipedia](https://en.wikipedia.org/wiki/Deflate)

**Go Libraries:**
- [compress/gzip Package](https://pkg.go.dev/compress/gzip)
- [compress/zlib Package](https://pkg.go.dev/compress/zlib)
- [klauspost/compress GitHub](https://github.com/klauspost/compress)

**Rust Libraries:**
- [flate2 Documentation](https://docs.rs/flate2)
- [flate2 GitHub](https://github.com/rust-lang/flate2-rs)

**Python Libraries:**
- [Python zlib module](https://docs.python.org/3/library/zlib.html)
- [Python gzip module](https://docs.python.org/3/library/gzip.html)
- [python-isal GitHub](https://github.com/pycompression/python-isal)

**Node.js:**
- [Node.js zlib documentation](https://nodejs.org/api/zlib.html)

**Developer Experience:**
- [Meta OpenZL](https://engineering.fb.com/2025/10/06/developer-tools/openzl-open-source-format-aware-compression-framework/)
- [API Design Best Practices](https://datanizant.com/api-design-best-practices/)
- [Compression Best Practices](https://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)

**Compression Algorithms:**
- [Zstandard at Meta](https://engineering.fb.com/2016/08/31/core-infra/smaller-and-faster-data-compression-with-zstandard/)
- [Compression Level Trade-offs](https://www.linkedin.com/advice/0/what-trade-offs-between-speed-compression-ratio)

**Technical Implementation:**
- [zlib Manual](https://www.zlib.net/manual.html)
- [zlib Usage Example](https://zlib.net/zlib_how.html)
- [Windows Compression API](https://learn.microsoft.com/en-us/windows/win32/cmpapi/using-the-compression-api-in-buffer-mode)
- [Eiffel-Loop compression](http://www.eiffel-loop.com/library/compression.html)
