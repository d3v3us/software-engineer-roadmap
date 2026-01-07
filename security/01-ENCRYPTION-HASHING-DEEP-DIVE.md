# Encryption and Hashing Deep Dive - Complete Understanding

## Table of Contents
1. [Understanding Encryption, Hashing, and Encoding](#understanding-encryption-hashing-and-encoding)
2. [Symmetric Encryption - Shared Secrets](#symmetric-encryption---shared-secrets)
3. [Asymmetric Encryption - Public and Private Keys](#asymmetric-encryption---public-and-private-keys)
4. [Hash Functions - One-Way Transformations](#hash-functions---one-way-transformations)
5. [Password Hashing - Secure Storage](#password-hashing---secure-storage)
6. [Digital Signatures - Authentication and Integrity](#digital-signatures---authentication-and-integrity)
7. [Key Management - Protecting Keys](#key-management---protecting-keys)
8. [Common Cryptographic Attacks and Defenses](#common-cryptographic-attacks-and-defenses)

---

## Understanding Encryption, Hashing, and Encoding

### The Three Concepts

**1. Encryption:**
- **Two-way**: Can encrypt and decrypt
- **Requires key**: Need key to decrypt
- **Purpose**: Confidentiality (keep data secret)

**2. Hashing:**
- **One-way**: Cannot reverse
- **No key**: Deterministic function
- **Purpose**: Integrity, authentication, indexing

**3. Encoding:**
- **Two-way**: Can encode and decode
- **No key**: No secret needed
- **Purpose**: Representation (not security)

### Quick Comparison

| Aspect | Encryption | Hashing | Encoding |
|--------|------------|---------|----------|
| **Reversible** | Yes (with key) | No | Yes |
| **Key Required** | Yes | No | No |
| **Purpose** | Confidentiality | Integrity/Auth | Representation |
| **Security** | Secure | Secure (one-way) | Not secure |

### Real-World Analogy

**Encryption = Safe:**
- Put document in safe (encrypt)
- Lock with key
- Only with key can open (decrypt)
- **Purpose**: Keep secret

**Hashing = Fingerprint:**
- Document → Fingerprint (hash)
- Can't get document from fingerprint
- Same document → Same fingerprint
- **Purpose**: Verify identity/integrity

**Encoding = Translation:**
- English → Spanish (encode)
- Spanish → English (decode)
- No secret, just different representation
- **Purpose**: Compatibility

---

## Symmetric Encryption - Shared Secrets

### What is Symmetric Encryption?

**Symmetric Encryption**: Same key used for encryption and decryption.

**Key Characteristics:**
- **One key**: Same key encrypts and decrypts
- **Fast**: Efficient algorithms
- **Secure**: With good key management

**Visual:**
```
Plaintext → [Encrypt with Key] → Ciphertext
Ciphertext → [Decrypt with Key] → Plaintext
Same key for both!
```

### Common Symmetric Algorithms

**1. AES (Advanced Encryption Standard):**
- **Key sizes**: 128, 192, 256 bits
- **Block size**: 128 bits
- **Modes**: ECB, CBC, GCM, etc.
- **Status**: Current standard, very secure

**2. DES (Data Encryption Standard):**
- **Key size**: 56 bits (too small!)
- **Status**: Deprecated, insecure

**3. 3DES (Triple DES):**
- **Key size**: 168 bits (effectively)
- **Status**: Legacy, being phased out

**4. ChaCha20:**
- **Stream cipher**: Encrypts data stream
- **Fast**: Good performance
- **Status**: Modern alternative to AES

### AES Deep Dive

**AES-256 Example:**
```
Key: 256 bits (32 bytes)
Plaintext: "Hello, World!"
Ciphertext: "U2FsdGVkX1+vupppZksvRf5pq5g5XkFyijB0V1D6B4Q="
```

**Modes of Operation:**

**1. ECB (Electronic Codebook):**
```
Each block encrypted independently
Problem: Same plaintext → Same ciphertext (patterns visible)
Not recommended!
```

**2. CBC (Cipher Block Chaining):**
```
Each block XORed with previous ciphertext
Needs IV (Initialization Vector)
More secure than ECB
```

**3. GCM (Galois/Counter Mode):**
```
Combines encryption and authentication
Provides confidentiality AND integrity
Recommended for modern applications
```

### Key Exchange Problem

**Problem:**
```
Alice and Bob want to communicate securely
They need to share the same key
How do they exchange the key securely?
```

**Solutions:**
- **Physical exchange**: Meet in person
- **Asymmetric encryption**: Use public key to exchange symmetric key
- **Key derivation**: Derive from password (with salt)

---

## Asymmetric Encryption - Public and Private Keys

### What is Asymmetric Encryption?

**Asymmetric Encryption**: Different keys for encryption and decryption.

**Key Pair:**
- **Public Key**: Can be shared publicly (encrypt)
- **Private Key**: Must be kept secret (decrypt)

**Visual:**
```
Plaintext → [Encrypt with Public Key] → Ciphertext
Ciphertext → [Decrypt with Private Key] → Plaintext
Different keys!
```

### How It Works

**Encryption:**
```
1. Bob generates key pair (public, private)
2. Bob shares public key with Alice
3. Alice encrypts message with Bob's public key
4. Alice sends ciphertext to Bob
5. Bob decrypts with his private key
```

**Key Insight:**
- **Anyone** can encrypt with public key
- **Only Bob** can decrypt with private key
- **Public key** can be shared freely

### Common Asymmetric Algorithms

**1. RSA (Rivest-Shamir-Adleman):**
- **Key sizes**: 2048, 4096 bits
- **Based on**: Factoring large numbers
- **Use**: Key exchange, digital signatures
- **Status**: Widely used, secure with large keys

**2. ECC (Elliptic Curve Cryptography):**
- **Key sizes**: 256, 384 bits (smaller than RSA)
- **Based on**: Elliptic curve discrete logarithm
- **Use**: Key exchange, digital signatures
- **Status**: Modern, efficient

**3. Diffie-Hellman:**
- **Purpose**: Key exchange only
- **Use**: Establish shared secret
- **Status**: Foundation of many protocols

### RSA Example

**Key Generation:**
```
1. Choose two large primes: p, q
2. Compute n = p * q
3. Compute φ(n) = (p-1)(q-1)
4. Choose e (public exponent)
5. Compute d (private exponent)
6. Public key: (n, e)
7. Private key: (n, d)
```

**Encryption:**
```
ciphertext = plaintext^e mod n
```

**Decryption:**
```
plaintext = ciphertext^d mod n
```

### Why Both Symmetric and Asymmetric?

**Hybrid Approach:**
```
1. Use asymmetric (RSA) to exchange symmetric key
   - Slow but secure for key exchange
2. Use symmetric (AES) to encrypt actual data
   - Fast for bulk data encryption
```

**Example (HTTPS):**
```
1. Client and server use RSA to exchange AES key
2. All data encrypted with AES (fast)
3. Best of both worlds!
```

---

## Hash Functions - One-Way Transformations

### What is a Hash Function?

**Hash Function**: Takes input of any size, produces fixed-size output.

**Properties:**
- **Deterministic**: Same input → Same output
- **One-way**: Cannot reverse (get input from output)
- **Avalanche effect**: Small input change → Large output change
- **Fixed size**: Output always same size

**Visual:**
```
Input: "Hello"
Hash: "8b1a9953c4611296a827abf8c47804d7"

Input: "Hello!" (one character different)
Hash: "9522862b5e3c5b7877b5e3c5b7877b5e" (completely different!)
```

### Common Hash Functions

**1. MD5:**
- **Output**: 128 bits
- **Status**: **Deprecated, insecure**
- **Use**: **Never for security!**

**2. SHA-1:**
- **Output**: 160 bits
- **Status**: **Deprecated, insecure**
- **Use**: **Never for security!**

**3. SHA-256:**
- **Output**: 256 bits
- **Status**: **Secure, recommended**
- **Use**: Data integrity, digital signatures

**4. SHA-512:**
- **Output**: 512 bits
- **Status**: **Secure, recommended**
- **Use**: When need larger hash

### Hash Use Cases

**1. Data Integrity:**
```
Send file + hash
Receiver computes hash
Compare: If match → File not corrupted
```

**2. Digital Signatures:**
```
Hash message → Sign hash (faster than signing entire message)
```

**3. Password Storage:**
```
Hash password (with salt)
Store hash, not password
```

**4. Hash Tables:**
```
Hash key → Index in array
Fast lookup
```

### Can Hashes be Cracked?

**Direct Reversal:**
- **No**: Mathematically impossible (one-way function)

**Attacks:**

**1. Brute Force:**
```
Try all possible inputs
Compare hash
Very slow for good hashes
```

**2. Dictionary Attack:**
```
Try common passwords
Much faster than brute force
```

**3. Rainbow Tables:**
```
Precomputed hash tables
Lookup hash → Get input
Defense: Salting
```

**4. Collision Attack:**
```
Find two inputs with same hash
MD5 and SHA-1 vulnerable
Use SHA-256 or SHA-512
```

---

## Password Hashing - Secure Storage

### The Problem

**Never store passwords in plain text!**

**Why:**
- **Database breach**: All passwords exposed
- **Reuse**: Users reuse passwords
- **Identity theft**: Can impersonate users

### The Solution: Hashing

**Store hash, not password:**
```
Password: "mypassword123"
Hash: "5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8"
Store: Hash (not password)
```

**Verification:**
```
User enters: "mypassword123"
Compute hash
Compare with stored hash
If match → Correct password
```

### The Salt Problem

**Problem:**
```
Same password → Same hash
"password123" → Always same hash
Rainbow tables: Precomputed hashes
```

**Solution: Salt:**
```
Password: "mypassword123"
Salt: "randomsalt123"
Hash: hash("mypassword123" + "randomsalt123")
Store: Hash + Salt

Same password, different salt → Different hash!
```

### Password Hashing Algorithms

**1. bcrypt:**
```
Designed for passwords
Slow (configurable)
Salt included automatically
Recommended
```

**2. scrypt:**
```
Memory-hard (resistant to hardware attacks)
Slow (configurable)
Good for passwords
```

**3. Argon2:**
```
Winner of Password Hashing Competition
Modern, secure
Recommended for new systems
```

**4. PBKDF2:**
```
Older but still used
Configurable iterations
Widely supported
```

### Why Slow Hashes?

**Fast Hash (SHA-256):**
```
1 billion hashes/second
Crack password in seconds
```

**Slow Hash (bcrypt):**
```
10,000 hashes/second
Crack password in years
```

**Trade-off:**
- **Slow for attackers**: Hard to brute force
- **Acceptable for users**: Login still fast (one hash)

---

## Digital Signatures - Authentication and Integrity

### What is a Digital Signature?

**Digital Signature**: Proves message authenticity and integrity.

**Properties:**
- **Authenticity**: Proves sender identity
- **Integrity**: Detects tampering
- **Non-repudiation**: Sender can't deny sending

### How Digital Signatures Work

**Signing:**
```
1. Hash message: hash(message) = H
2. Encrypt hash with private key: sign(H, private_key) = signature
3. Send: message + signature
```

**Verification:**
```
1. Receive: message + signature
2. Hash message: hash(message) = H
3. Decrypt signature with public key: decrypt(signature, public_key) = H'
4. Compare: If H == H' → Valid signature
```

**Visual:**
```
Sender:
Message → Hash → Sign with Private Key → Signature
                                    ↓
                            Send: Message + Signature

Receiver:
Message + Signature → Hash Message → Compare with Decrypted Signature
                                         ↑
                              Decrypt with Public Key
```

### Why Hash Then Sign?

**Efficiency:**
```
Sign entire message: Slow (asymmetric encryption is slow)
Hash message: Fast
Sign hash: Fast (hash is small)
→ Much faster!
```

**Security:**
```
If message changes → Hash changes → Signature invalid
→ Detects tampering
```

### HMAC (Hash-Based Message Authentication Code)

**What:**
- **Symmetric** version of digital signature
- **Uses shared secret** (not public/private keys)

**How:**
```
HMAC(message, key) = hash(key + hash(key + message))
```

**Use Cases:**
- **API authentication**: Verify request authenticity
- **JWT signatures**: Sign tokens
- **Message verification**: Verify message integrity

**HMAC vs Digital Signature:**
- **HMAC**: Symmetric (same key), faster
- **Digital Signature**: Asymmetric (different keys), stronger

---

## Key Management - Protecting Keys

### The Key Management Problem

**Problem:**
- **Encryption is only as strong as key security**
- **If key is compromised**: All encrypted data is compromised
- **Key management is critical**

### Key Storage

**Never store keys in code!**

**Options:**

**1. Environment Variables:**
```
export ENCRYPTION_KEY="..."
Application reads from environment
```

**2. Key Management Services:**
```
AWS KMS, Azure Key Vault, Google Cloud KMS
Managed service
Secure storage
```

**3. Hardware Security Modules (HSM):**
```
Physical devices
Keys never leave HSM
Highest security
```

**4. Secret Management:**
```
HashiCorp Vault, AWS Secrets Manager
Centralized secret storage
Access control
```

### Key Rotation

**Why:**
- **Compromised keys**: If key leaked, rotate
- **Time-based**: Rotate periodically
- **Compliance**: Some regulations require rotation

**Process:**
```
1. Generate new key
2. Re-encrypt data with new key (gradually)
3. Update applications to use new key
4. Keep old key (for decrypting old data)
5. Eventually retire old key
```

### Key Derivation

**From Password:**
```
Password → Key Derivation Function → Encryption Key
```

**PBKDF2 Example:**
```
key = PBKDF2(password, salt, iterations)
```

**Benefits:**
- **Don't store key**: Derive from password
- **Slow**: Resistant to brute force
- **Salt**: Prevents rainbow tables

---

## Common Cryptographic Attacks and Defenses

### 1. Brute Force Attack

**Attack:**
```
Try all possible keys
Eventually find correct key
```

**Defense:**
- **Large key size**: More possible keys
- **Rate limiting**: Limit attempts
- **Account lockout**: Lock after failed attempts

### 2. Man-in-the-Middle (MITM)

**Attack:**
```
Attacker intercepts communication
Modifies messages
```

**Defense:**
- **TLS/SSL**: Encrypt communication
- **Certificate pinning**: Verify server identity
- **Public key infrastructure**: Trusted certificates

### 3. Replay Attack

**Attack:**
```
Capture encrypted message
Replay later
```

**Defense:**
- **Nonces**: One-time values
- **Timestamps**: Expire old messages
- **Sequence numbers**: Detect replays

### 4. Side-Channel Attacks

**Attack:**
```
Analyze timing, power consumption
Infer key information
```

**Defense:**
- **Constant-time algorithms**: Same time regardless of input
- **Hardware protection**: HSM

---

## Summary

Understanding encryption, hashing, and their applications is essential for secure systems. Encryption provides confidentiality, hashing provides integrity, and proper key management ensures security.

**Key Takeaways:**
- Encryption: Two-way, requires key, for confidentiality
- Hashing: One-way, no key, for integrity/authentication
- Encoding: Two-way, no key, for representation (not security)
- Symmetric: Fast, same key, for bulk encryption
- Asymmetric: Slow, different keys, for key exchange/signatures
- Password hashing: Use slow hashes (bcrypt, Argon2) with salt
- Digital signatures: Prove authenticity and integrity
- Key management: Critical for security
- Protect keys: Never in code, use key management services

**Next Steps:**
- Understand your application's security needs
- Choose appropriate algorithms
- Implement proper key management
- Use established libraries (don't implement yourself)
- Keep algorithms and keys secure
- Stay updated on cryptographic best practices

