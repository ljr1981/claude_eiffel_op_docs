# simple_encryption Research Report

**Date:** 2025-12-08
**Library:** simple_encryption (Cryptographic Wrapper for ISE EEL)

---

## Step 1: Specifications Research

### Cryptographic Standards Overview

| Standard | Specification | Key Characteristics |
|----------|--------------|---------------------|
| **AES** | FIPS 197 | 128/192/256-bit keys, 128-bit blocks, symmetric encryption |
| **SHA-2** | FIPS 180-4 | SHA-224/256/384/512, cryptographic hashing |
| **RSA** | PKCS#1 v2.2 (RFC 8017) | Asymmetric encryption, OAEP padding recommended |
| **ECDSA** | ANSI X9.62, NIST SP 800-186 | Elliptic curve signatures, smaller keys than RSA |

### AES - FIPS 197

The Advanced Encryption Standard (AES) specifies a FIPS-approved cryptographic algorithm that can encrypt and decrypt digital information. Three variants exist:
- **AES-128**: 128-bit key, 10 rounds
- **AES-192**: 192-bit key, 12 rounds
- **AES-256**: 256-bit key, 14 rounds (quantum-resistant)

Each transforms data in blocks of 128 bits using four transformations: byte substitution (S-box), row shifting, column mixing, and round key addition.

**Quantum Considerations:** AES-256 is considered quantum-resistant with 128 bits of security against quantum attacks. AES-128 drops to 64 bits of security against quantum computers, making it insecure for long-term protection.

### SHA-2 - FIPS 180-4

SHA-2 family consists of six hash functions with digests of 224, 256, 384, or 512 bits. When a message less than 2^64 bits (for SHA-256) or 2^128 bits (for SHA-512) is input, the result is a fixed-length message digest.

**Current Status:** SHA-1 was deprecated by NIST in 2011 and disallowed for digital signatures in 2013. SHA-256 and SHA-512 remain secure for 2025.

### RSA - PKCS#1 v2.2

RSA defines asymmetric encryption and signature schemes:
- **RSAES-OAEP**: Recommended encryption scheme using Optimal Asymmetric Encryption Padding
- **RSAES-PKCS1-v1_5**: Legacy scheme, vulnerable to Bleichenbacher's attack (1998)
- **RSASSA-PSS**: Recommended signature scheme using Probabilistic Signature Scheme
- **RSASSA-PKCS1-v1_5**: Legacy signature scheme

**Security Note:** PKCS#1 v1.5 padding is vulnerable to padding oracle attacks. Modern applications should use OAEP for encryption and PSS for signatures.

### ECDSA - Elliptic Curve Digital Signatures

ECDSA offers equivalent security to RSA with much smaller key sizes:
- 256-bit ECDSA ≈ 3072-bit RSA security
- Standardized by ANSI (1999), IEEE and NIST (2000)
- Used in Bitcoin, Ethereum, TLS, SSH

**Common Curves:** secp256k1 (Bitcoin), secp256r1/P-256 (NIST), Curve25519 (modern recommendation)

**Critical Vulnerability:** Nonce reuse in ECDSA allows private key recovery (PlayStation 3 breach, 2010).

**Sources:**
- [FIPS 197 Advanced Encryption Standard](https://csrc.nist.gov/pubs/fips/197/final)
- [FIPS 180-4 Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final)
- [RFC 8017 PKCS #1: RSA Cryptography v2.2](https://tools.ietf.org/html/rfc8017)
- [ECDSA Wikipedia](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)

---

## Step 2: Tech-Stack Library Analysis

### Go: crypto Standard Library

**Key Features:**
- **AES**: `crypto/aes` with all standard modes (GCM recommended)
- **Hashing**: `crypto/sha256`, `crypto/sha512`, `crypto/md5` (legacy)
- **RSA**: `crypto/rsa` with OAEP and PSS support
- **Random**: `crypto/rand` for CSPRNG
- **TLS**: Full implementation in `crypto/tls`

**Extended Library (golang.org/x/crypto):**
- **Argon2**: Winner of Password Hashing Competition, recommended KDF
  - `argon2id`: Hybrid mode (recommended): time=1, memory=max available
  - `argon2i`: Side-channel resistant: time=3, memory=max available
- **bcrypt**: Adaptive hashing, cost factor 10-12 recommended
- **PBKDF2**: Legacy KDF, 600,000+ iterations with SHA-256
- **ChaCha20-Poly1305**: Modern AEAD cipher

**API Pattern:**
```go
// Simple encryption with sensible defaults
import "crypto/aes"
import "crypto/cipher"

key := []byte("32-byte-key-for-aes-256-cipher!!")
plaintext := []byte("Hello, World!")

block, _ := aes.NewCipher(key)
aead, _ := cipher.NewGCM(block)

nonce := make([]byte, aead.NonceSize())
rand.Read(nonce)

ciphertext := aead.Seal(nil, nonce, plaintext, nil)
```

### Rust: ring and RustCrypto

**ring Library:**
- Focus: "Safe, fast, small crypto"
- Based on BoringSSL primitives
- Hard-to-misuse API design
- Written in Rust, C, and assembly

**RustCrypto Ecosystem:**
- Pure Rust implementations
- Trait-based design for algorithm abstraction
- 38+ repositories covering all crypto primitives

**Key Finding:** Research shows only half of crypto libraries explicitly focus on usability and misuse resistance. Ring is designed to be "hard to misuse" but developers were paradoxically more successful with rust-crypto despite its simpler API.

**API Pattern:**
```rust
use ring::aead::{Aad, LessSafeKey, Nonce, UnboundKey, AES_256_GCM};
use ring::rand::{SecureRandom, SystemRandom};

let rng = SystemRandom::new();
let mut key_bytes = [0u8; 32];
rng.fill(&mut key_bytes)?;

let unbound_key = UnboundKey::new(&AES_256_GCM, &key_bytes)?;
let key = LessSafeKey::new(unbound_key);

let mut nonce_bytes = [0u8; 12];
rng.fill(&mut nonce_bytes)?;
let nonce = Nonce::assume_unique_for_key(nonce_bytes);

let mut in_out = plaintext.to_vec();
key.seal_in_place_append_tag(nonce, Aad::empty(), &mut in_out)?;
```

### Python: cryptography and PyCryptodome

**cryptography Library (Recommended):**
- High-level "recipes" for common tasks
- Low-level "hazmat" primitives for experts
- Focus on secure defaults

**PyCryptodome:**
- Drop-in replacement for deprecated PyCrypto
- Native support for AES, RSA, ECC, ChaCha20-Poly1305
- Key Derivation Functions: PBKDF2, scrypt, bcrypt

**PBKDF2 API:**
```python
from Crypto.Protocol.KDF import PBKDF2
from Crypto.Hash import SHA512
from Crypto.Random import get_random_bytes

password = b'my super secret'
salt = get_random_bytes(16)
key = PBKDF2(password, salt, 64, count=1000000, hmac_hash_module=SHA512)
```

**Important Note:** PBKDF2 is insufficiently protected against GPU attacks. New applications should use scrypt, bcrypt, or Argon2.

### Node.js: crypto Module and Libraries

**Built-in crypto Module:**
- Native support for common algorithms
- `crypto.randomBytes()` for CSPRNG
- `crypto.pbkdf2()` for key derivation
- `crypto.scrypt()` available since Node 10.5.0

**bcrypt vs Native crypto:**
- **bcrypt Wins** for password hashing: Simpler API, combined salt+hash storage
- Native crypto requires separate salt storage
- bcrypt work factor (rounds): 10 = ~10 hashes/sec, 12 = 2-3 hashes/sec, 13 = ~1 sec/hash

**Recommended Pattern:**
```javascript
const bcrypt = require('bcrypt');

// Hash password
const saltRounds = 12;
const hash = await bcrypt.hash(password, saltRounds);

// Verify password
const match = await bcrypt.compare(password, hash);
```

**OWASP Guidelines:**
- Use adaptive hash functions (bcrypt, Argon2)
- Never use MD5, SHA1 for passwords
- Salt and hash passwords before storage
- Use constant-time comparison for verification

**Sources:**
- [Go crypto/pbkdf2](https://pkg.go.dev/golang.org/x/crypto/pbkdf2)
- [Go crypto/argon2](https://pkg.go.dev/golang.org/x/crypto/argon2)
- [Rust ring documentation](https://docs.rs/ring)
- [PyCryptodome KDF documentation](https://pycryptodome.readthedocs.io/en/latest/src/protocol/kdf.html)
- [Node.js bcrypt package](https://www.npmjs.com/package/bcrypt)

---

## Step 3: Eiffel Ecosystem Analysis

### ISE Eiffel Encryption Library (EEL)

**Location:** `$ISE_LIBRARY/library/eel/`

**Available Capabilities:**

**Symmetric Encryption:**
- **AES**: Supports CBC, CFB, CTR, ECB, OFB modes
- Block cipher operations with padding

**Hash Functions:**
- MD5 (legacy, cryptographically broken)
- SHA-1 (deprecated since 2011)
- SHA-256 (recommended)

**Message Authentication:**
- HMAC with MD5, SHA-1, SHA-256

**Asymmetric Cryptography:**
- RSA key pair generation
- Elliptic Curve cryptography
- X.509 certificate support
- DER encoding/decoding

**Architecture Patterns:**
EEL follows object-oriented design with:
- Base classes for cipher operations
- Mode-specific descendants (CBC, CTR, GCM, etc.)
- Separate classes for encoding (Base64, hex)

**Key Gaps in EEL:**
- No Argon2, scrypt, or bcrypt for password hashing
- No PBKDF2 for key derivation
- No ChaCha20-Poly1305 (modern AEAD)
- No built-in secure random number generation wrapper
- Low-level API requires cryptographic expertise
- Easy to misuse (wrong mode, no IV, weak padding)

### Related simple_* Libraries

**simple_hash:**
- Pure Eiffel SHA-256, HMAC-SHA256, MD5
- Hex conversion utilities
- Simple one-method API: `hasher.sha256("data")`

**simple_base64:**
- Standard Base64 and URL-safe Base64URL
- Validation and format conversion
- Used for encoding encrypted data

**Design Philosophy of simple_* Ecosystem:**
1. Single-method simplicity for common cases
2. Pure Eiffel when possible (no C dependencies)
3. Design by Contract with preconditions/postconditions
4. README-driven documentation with clear examples

---

## Step 4: Developer Pain Points

### Common Cryptography Mistakes

**1. Initialization Vector (IV) Misuse:**
- ❌ Reusing IV with same key (catastrophic in AES-GCM)
- ❌ Using constant/predictable IV
- ❌ Not using IV at all
- ✅ Generate random IV using CSPRNG for each encryption
- ✅ Store IV with ciphertext (it's not secret)

**2. Nonce Reuse (AES-GCM, ChaCha20-Poly1305):**
- **Single reuse** exposes authentication key → universal forgery
- **ECDSA nonce reuse** allows private key recovery (PlayStation 3 hack)
- Must use CSPRNG for nonces (minimum 112 bits for AES-GCM)

**3. Padding Scheme Mismatches:**
- PKCS#5/PKCS#7 padding required for CBC mode
- GCM mode does **not** require padding
- Padding oracle attacks exploit incorrect padding validation
- Always use authenticated encryption (AEAD) to prevent this

**4. Wrong Mode Selection:**
- **ECB mode**: Never use (reveals patterns in plaintext)
- **CBC mode**: Vulnerable to padding oracles, requires HMAC for authentication
- **GCM mode**: Recommended (authenticated encryption built-in)
- **CTR mode**: Requires external MAC (or use GCM)

**5. Encryption Without Authentication:**
- OWASP 2025: "Always use authenticated encryption instead of just encryption"
- Prefer AEAD ciphers: AES-GCM, ChaCha20-Poly1305
- If using CBC, apply encrypt-then-MAC pattern

### API Usability Research Findings

**Study of 500 Stack Overflow Posts:**
- 112 posts: Encryption/decryption problems
- 111 posts: Installation/compilation issues
- 63 posts: Interoperability between crypto libraries
- Only 2 posts: Actual attacks against cryptography

**Key Insight:** Developers struggle with **using** crypto correctly, not understanding theory.

**What Makes Crypto Libraries Hard:**
- Too many algorithm choices (decision fatigue)
- Low-level primitives require assembly
- Missing features (password hashing, key storage)
- Poor documentation and examples
- Defaults are often insecure

**NaCl/libsodium Solution:**
- "Cryptography on Rails" - convention over configuration
- High-level APIs with secure defaults
- Hard to misuse by design
- Expert-assembled cryptographic operations
- Example: `crypto_box()` does key agreement (X25519) + encryption (Salsa20-Poly1305) in one call

**Sources:**
- [OWASP Cryptographic Failures 2025](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
- [10 Cryptography Mistakes](https://www.appsecengineer.com/blog/10-cryptography-mistakes-youre-probably-making)
- [Comparing Usability of Cryptographic APIs](https://www.cl.cam.ac.uk/events/shb/2017/fahl.pdf)

---

## Step 5: Innovation Opportunities

### What Would Make Encryption Simple for Eiffel Developers?

### 1. Password Hashing Made Easy

**Problem:** Developers use SHA-256 directly on passwords (insecure).

**Solution:** High-level password hashing API
```eiffel
create crypto.make

-- Hash password (uses bcrypt-equivalent with salt)
hashed := crypto.hash_password ("user_password")
-- Result: "$2b$12$KIXxGVE8KvmYP0fH.WLTiOjPQv7qF..."

-- Verify password
if crypto.verify_password ("user_password", hashed) then
    -- Login successful
end
```

**Implementation:** Could wrap EEL's crypto with PBKDF2, or use simple_hash with high iteration count until bcrypt/Argon2 available.

### 2. Simple Encrypt/Decrypt with Sensible Defaults

**Problem:** Choosing algorithm, mode, padding, IV is error-prone.

**Solution:** Single-method encryption with AES-256-GCM default
```eiffel
create crypto.make

-- Encrypt (handles key derivation, IV generation, authentication)
encrypted := crypto.encrypt_with_password ("secret data", "password123")
-- Returns: nonce + ciphertext + authentication_tag (all Base64 encoded)

-- Decrypt (automatically extracts nonce and verifies authentication)
if attached crypto.decrypt_with_password (encrypted, "password123") as data then
    -- Success
else
    -- Wrong password or tampered data
end
```

**Under the Hood:**
- Password → PBKDF2 → AES-256 key
- Random nonce generation
- AES-256-GCM encryption
- Automatic authentication tag verification
- Base64 encoding for storage

### 3. Secure Random Number Generation

**Problem:** EEL doesn't provide CSPRNG wrapper.

**Solution:** Simple random utilities
```eiffel
create crypto.make

-- Generate random bytes
random_bytes := crypto.random_bytes (32)  -- 256 bits

-- Generate random string (for tokens, session IDs)
token := crypto.random_token (32)  -- 32-character hex string

-- Generate random integer (for verification codes)
code := crypto.random_integer (100000, 999999)  -- 6-digit code
```

**Implementation:** Wrap platform CSPRNG (`/dev/urandom` on Unix, `BCryptGenRandom` on Windows).

### 4. Key Derivation Functions (PBKDF2/scrypt/Argon2)

**Problem:** Deriving encryption keys from passwords is non-trivial.

**Solution:** Simple KDF API
```eiffel
-- Derive key from password with salt
salt := crypto.random_bytes (16)
key := crypto.derive_key_pbkdf2 ("password", salt, 600000, 32)

-- Or use higher-level method that handles salt internally
key_material := crypto.derive_key ("password", 32)
```

**Parameters (2025 Recommendations):**
- **PBKDF2-SHA256**: 600,000+ iterations
- **Argon2id**: 64MB memory, 3 iterations, 4 threads
- **scrypt**: N=2^17, r=8, p=1

### 5. Encoding Helpers (Integration with simple_base64)

**Solution:** Built-in encoding for ciphertext storage
```eiffel
-- Encrypt and encode in one step
encoded_ciphertext := crypto.encrypt_and_encode ("data", key)
-- Returns Base64 string suitable for database storage

-- Decode and decrypt
if attached crypto.decode_and_decrypt (encoded_ciphertext, key) as data then
    -- Success
end
```

### 6. File Encryption Utilities

**Solution:** Stream encryption for large files
```eiffel
create crypto.make

-- Encrypt file with password
crypto.encrypt_file ("document.pdf", "encrypted.pdf.enc", "password")

-- Decrypt file
crypto.decrypt_file ("encrypted.pdf.enc", "document.pdf", "password")
```

**Implementation:** Chunked encryption to avoid loading entire file into memory.

### 7. Digital Signatures (High-Level)

**Solution:** Simple signing API
```eiffel
-- Generate key pair
key_pair := crypto.generate_keypair

-- Sign data
signature := crypto.sign ("message", key_pair.private_key)

-- Verify signature
if crypto.verify ("message", signature, key_pair.public_key) then
    -- Authentic
end
```

---

## Step 6: Design Strategy Synthesis

### Core Design Principles

1. **Secure by Default**
   - AES-256-GCM (not CBC or ECB)
   - Automatic IV/nonce generation
   - Authenticated encryption always
   - Safe parameter choices (high iteration counts)

2. **Simple API for Common Cases**
   - One-method encryption: `encrypt_with_password(data, password)`
   - One-method password hashing: `hash_password(password)`
   - Hide complexity (IV, salt, encoding handled internally)

3. **Progressive Disclosure**
   - Simple API for 80% use cases
   - Advanced API for custom algorithms/modes
   - Expert API to access raw EEL if needed

4. **Hard to Misuse**
   - No separate IV parameter (auto-generated and prepended)
   - No mode selection for basic operations (GCM default)
   - Authentication automatic (AEAD ciphers only)
   - Contract-driven validation

### What simple_encryption Should Be

**Primary Goals:**
- Wrapper around ISE EEL with safe defaults
- Password-based encryption for application data
- Password hashing for user credentials
- Random number generation utilities
- Base64 encoding integration (via simple_base64)

**Target Use Cases:**
- Encrypting configuration files
- Storing user passwords securely
- Generating API tokens and session IDs
- Encrypting database fields
- Protecting files at rest

### What simple_encryption Should NOT Be

**Out of Scope:**
- TLS/SSL implementation (too complex)
- Full PKI and certificate management
- Hardware security module (HSM) integration
- Blockchain or cryptocurrency operations
- Zero-knowledge proof systems
- Homomorphic encryption

**Delegate to EEL Directly:**
- Custom cipher modes (CTR, OFB, CFB)
- Low-level RSA operations
- X.509 certificate parsing
- DER encoding/decoding
- Elliptic curve operations

### API Surface Design

**Minimal API (Version 1.0):**

```eiffel
class SIMPLE_ENCRYPTION

feature -- Password Hashing

    hash_password (password: STRING): STRING
        -- Hash password with secure algorithm (PBKDF2-SHA256)
        -- Returns combined salt+hash string

    verify_password (password: STRING; hash: STRING): BOOLEAN
        -- Verify password against stored hash

feature -- Symmetric Encryption

    encrypt_with_password (data: STRING; password: STRING): STRING
        -- Encrypt data with password (AES-256-GCM, auto IV)
        -- Returns Base64: nonce + ciphertext + tag

    decrypt_with_password (encrypted: STRING; password: STRING): detachable STRING
        -- Decrypt and verify. Returns Void if wrong password/tampered

feature -- Random Generation

    random_bytes (count: INTEGER): ARRAY [NATURAL_8]
        -- Generate cryptographically secure random bytes

    random_token (length: INTEGER): STRING
        -- Generate random hex string for tokens/IDs

feature -- Key Derivation

    derive_key (password: STRING; key_size: INTEGER): ARRAY [NATURAL_8]
        -- Derive encryption key from password using PBKDF2
        -- Automatically generates and includes salt

end
```

**Extended API (Future Versions):**

```eiffel
feature -- Advanced Encryption

    encrypt_with_key (data: STRING; key: ARRAY [NATURAL_8]): STRING
        -- Direct encryption with existing key

    encrypt_file (source_path: STRING; dest_path: STRING; password: STRING)
        -- Encrypt large file with streaming

feature -- Hashing

    sha256 (data: STRING): STRING
        -- Quick access to SHA-256 (delegates to simple_hash)

    hmac_sha256 (key: STRING; message: STRING): STRING
        -- HMAC-SHA256 (delegates to simple_hash)

feature -- Advanced Key Derivation

    derive_key_with_salt (password: STRING; salt: ARRAY [NATURAL_8];
                          iterations: INTEGER; key_size: INTEGER): ARRAY [NATURAL_8]
        -- PBKDF2 with explicit parameters

    derive_key_scrypt (password: STRING; salt: ARRAY [NATURAL_8];
                       n, r, p: INTEGER; key_size: INTEGER): ARRAY [NATURAL_8]
        -- scrypt KDF (if available in EEL or via C extension)
```

### Security Recommendations for Implementation

**Must Have:**
1. Never store encryption keys in code
2. Use CSPRNG for all random generation (IV, salt, nonce)
3. Verify authentication tags before returning plaintext
4. Wipe sensitive data from memory after use
5. Validate all inputs (key size, iteration count)

**Should Have:**
1. Timing-safe password comparison (prevent timing attacks)
2. Key stretching with high iteration counts (600k+ for PBKDF2)
3. Clear error messages without leaking information
4. Comprehensive preconditions and postconditions

**Could Have:**
1. Memory locking for key material (prevent swap to disk)
2. Automatic key rotation utilities
3. Audit logging of cryptographic operations

---

## Step 7: Gap Analysis

### Comparison: Modern Crypto Libraries vs. simple_encryption Goals

| Feature | Go crypto | Python cryptography | Node bcrypt | simple_encryption Goal |
|---------|-----------|---------------------|-------------|----------------------|
| AES-GCM encryption | ✅ | ✅ | ❌ | ✅ (via EEL) |
| Password hashing (bcrypt/Argon2) | ✅ (x/crypto) | ✅ | ✅ | ⚠️ (PBKDF2 only) |
| PBKDF2 | ✅ | ✅ | ❌ | ✅ (via EEL) |
| CSPRNG | ✅ | ✅ | ✅ | ✅ (OS-level) |
| One-method encrypt | ❌ | ⚠️ | N/A | ✅ Target |
| Auto IV generation | Manual | Manual | N/A | ✅ Automatic |
| Base64 encoding | Manual | Manual | Built-in | ✅ (via simple_base64) |
| File encryption | Manual | Manual | N/A | ✅ Planned |
| Key derivation | ✅ | ✅ | ❌ | ✅ (via EEL) |

### What ISE EEL Provides (Available to Wrap)

✅ **Strong Foundation:**
- AES with multiple modes (CBC, CTR, GCM, OFB, CFB)
- SHA-256 hashing
- HMAC-SHA256
- RSA operations
- Elliptic curve support
- X.509 certificates

### What simple_encryption Must Add

❌ **Missing Layer:**
1. **High-level API** - One-method encryption/decryption
2. **Automatic IV/nonce handling** - Generate, prepend, extract
3. **Password-based encryption** - Key derivation wrapper
4. **Password hashing** - PBKDF2 with safe parameters
5. **CSPRNG wrapper** - Platform random number generation
6. **Base64 integration** - Auto-encode encrypted output
7. **File encryption** - Streaming for large files
8. **Safe defaults** - No mode selection for basic use
9. **Authentication verification** - Auto-verify before decrypt
10. **Clear error handling** - Void for auth failure, not exception

### Critical Security Gaps to Address

**1. No Built-in bcrypt/Argon2:**
- **Workaround:** Implement PBKDF2 with 600,000+ iterations as interim solution
- **Future:** Add C extension for Argon2id or bcrypt
- **Documentation:** Clearly state PBKDF2 limitations vs GPU attacks

**2. No Secure Random Wrapper:**
- **Solution:** Create `SIMPLE_RANDOM` class wrapping:
  - Unix: `/dev/urandom` via external process
  - Windows: `BCryptGenRandom` via WEL or inline C

**3. AES-GCM Nonce Management:**
- **Risk:** EEL requires manual nonce generation
- **Solution:** Auto-generate 96-bit nonce, prepend to ciphertext
- **Contract:** Never allow nonce reuse (track in implementation)

**4. Memory Security:**
- **Risk:** Sensitive data (keys, passwords) in managed memory
- **Mitigation:** Minimize lifetime, no string copies when possible
- **Future:** Memory locking if EEL supports

### Implementation Roadmap

**Phase 1: Core Encryption (Weeks 1-2)**
1. Wrap EEL AES-256-GCM with auto IV
2. Implement `encrypt_with_password` / `decrypt_with_password`
3. PBKDF2 key derivation (600k iterations)
4. Base64 encoding integration (simple_base64)
5. Basic error handling

**Phase 2: Password Hashing (Week 3)**
1. PBKDF2-based password hashing
2. Salt generation and storage
3. Timing-safe password verification
4. Hash format: `$pbkdf2$iterations$salt$hash`

**Phase 3: Random & Utilities (Week 4)**
1. CSPRNG wrapper (platform-specific)
2. Random token generation
3. Random bytes utility
4. Hex/Base64 encoding helpers

**Phase 4: File Encryption (Week 5)**
1. Streaming file encryption
2. Chunked processing (avoid memory issues)
3. Progress callback support
4. Integrity verification

**Phase 5: Advanced Features (Future)**
1. Key rotation utilities
2. Multiple recipient encryption
3. Digital signatures wrapper
4. Argon2 integration (C extension)

---

## Recommendations

### Immediate Priorities for simple_encryption v1.0

**Must Implement:**
1. ✅ Password-based AES-256-GCM encryption
2. ✅ Automatic IV/nonce generation and handling
3. ✅ PBKDF2 password hashing (600k iterations)
4. ✅ CSPRNG random number generation
5. ✅ Base64 encoding for ciphertext storage

**Should Implement:**
1. ⚠️ File encryption with streaming
2. ⚠️ Random token generation utilities
3. ⚠️ Integration tests with test vectors
4. ⚠️ Comprehensive documentation with security warnings

**Could Implement (v2.0):**
1. ⏸️ Argon2id password hashing (requires C extension)
2. ⏸️ ChaCha20-Poly1305 support (if added to EEL)
3. ⏸️ Key rotation utilities
4. ⏸️ Multiple encryption recipient support

### Security Warnings to Document

**Critical Warnings:**
- ⚠️ **Password Hashing:** PBKDF2 is vulnerable to GPU attacks. For high-security applications, consider external Argon2 or bcrypt.
- ⚠️ **Key Storage:** Never hardcode encryption keys. Use environment variables or secure key management systems.
- ⚠️ **Nonce Reuse:** Catastrophic in AES-GCM. This library handles it automatically—don't bypass.
- ⚠️ **Quantum Resistance:** AES-256 is quantum-resistant; AES-128 is not. We default to AES-256.

**Best Practices:**
- ✅ Use `encrypt_with_password` for application data encryption
- ✅ Use `hash_password` for user credential storage
- ✅ Use `random_token` for session IDs and API tokens
- ✅ Store encrypted data as Base64 in databases
- ✅ Never decrypt and re-encrypt with same nonce

---

## Conclusion

Modern crypto libraries (Go, Rust, Python, Node.js) have converged on several best practices:

**Universal Patterns:**
1. **High-level APIs** hide complexity (NaCl/libsodium model)
2. **Secure defaults** prevent common mistakes (AES-GCM not CBC, auto IV)
3. **AEAD ciphers** provide authentication automatically
4. **Password hashing** uses adaptive algorithms (Argon2 > bcrypt > scrypt > PBKDF2)
5. **Key derivation** with high iteration counts (600k+ for PBKDF2)
6. **CSPRNG** for all random generation

**simple_encryption Value Proposition:**

For Eiffel developers, `simple_encryption` should be the **"NaCl for Eiffel"**—a library that makes cryptography simple without sacrificing security. By wrapping ISE EEL with:
- One-method encryption/decryption
- Automatic IV/nonce/salt handling
- Safe parameter defaults
- Clear error handling
- Design by Contract validation

We can prevent the common mistakes (wrong mode, IV reuse, weak parameters, no authentication) that plague cryptographic implementations.

**Target Developer Experience:**
```eiffel
-- Instead of 20 lines of EEL configuration...
create crypto.make
encrypted := crypto.encrypt_with_password ("secret data", "password")
```

**The Goal:** Make secure encryption as easy as:
- `simple_hash.sha256("data")` for hashing
- `simple_base64.encode("data")` for encoding
- `simple_encryption.encrypt_with_password("data", "pass")` for encryption

This research demonstrates that **simplicity and security are not mutually exclusive**—they are achieved through expert assembly of cryptographic primitives with safe defaults and comprehensive validation.

---

**Research Sources:**
- [NIST Cryptographic Standards](https://csrc.nist.gov/)
- [OWASP Cryptographic Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [Password Storage OWASP](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
- [NaCl: Networking and Cryptography library](https://nacl.cr.yp.to/)
- [libsodium Documentation](https://doc.libsodium.org/)
- [Comparing Usability of Cryptographic APIs (Research Paper)](https://www.cl.cam.ac.uk/events/shb/2017/fahl.pdf)
