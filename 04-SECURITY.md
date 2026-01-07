# Security - Comprehensive Guide

## Table of Contents
1. [Hash vs Encrypt vs Encode](#hash-vs-encrypt-vs-encode)
2. [SSL/TLS and Certificates](#ssltls-and-certificates)
3. [Credential Storage](#credential-storage)
4. [DDoS Defense](#ddos-defense)

---

## Hash vs Encrypt vs Encode

### Hash

**Hash** is a one-way function that converts input into fixed-size output.

**Characteristics:**
- **One-way**: Cannot reverse hash to get original input
- **Deterministic**: Same input always produces same output
- **Fixed size**: Output size is constant (e.g., 256 bits)
- **Avalanche effect**: Small input change → Large output change

**Example:**
```
Input: "password123"
SHA-256: "ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94f"

Input: "password124" (one character changed)
SHA-256: "a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3"
(Completely different!)
```

**Use Cases:**
- Password storage
- Data integrity verification
- Digital signatures
- Hash tables

**Common Hash Functions:**
- **MD5**: 128 bits (deprecated, insecure)
- **SHA-1**: 160 bits (deprecated, insecure)
- **SHA-256**: 256 bits (secure)
- **SHA-512**: 512 bits (secure)

### Can Hash be Cracked?

**Direct Reversal**: No, mathematically impossible (one-way function)

**Attacks:**

**1. Brute Force:**
```
Try all possible inputs:
"a", "b", "c", ..., "password", "password1", ...
Compare hash until match found
```

**Time**: Exponential (2^256 for SHA-256)
**Feasible**: Only for weak passwords

**2. Dictionary Attack:**
```
Try common passwords:
"password", "123456", "qwerty", ...
Much faster than brute force
```

**3. Rainbow Tables:**
```
Precomputed hash tables:
"password" → hash1
"123456" → hash2
...
Lookup hash in table
```

**Defense**: **Salting** - Add random salt before hashing

**4. Collision Attack:**
```
Find two different inputs with same hash
MD5 and SHA-1 vulnerable to this
```

**Defense**: Use secure hash functions (SHA-256, SHA-512)

### Encrypt

**Encrypt** is a two-way function that converts plaintext to ciphertext using a key.

**Characteristics:**
- **Two-way**: Can decrypt to get original (with key)
- **Key-based**: Requires key to encrypt/decrypt
- **Reversible**: Decryption recovers original data

**Example:**
```
Plaintext: "Hello, World!"
Key: "mysecretkey"
Ciphertext: "U2FsdGVkX1+vupppZksvRf5pq5g5XkFyijB0V1D6B4Q="

Decrypt with key → "Hello, World!"
```

**Use Cases:**
- Secure data transmission
- Data at rest encryption
- Confidential information

### Symmetric vs Asymmetric Encryption

**Symmetric Encryption:**
- **Same key** for encryption and decryption
- Fast
- Examples: AES, DES, 3DES

**Example:**
```
Encrypt: ciphertext = encrypt(plaintext, key)
Decrypt: plaintext = decrypt(ciphertext, key)
Same key used for both
```

**Asymmetric Encryption:**
- **Different keys**: Public key (encrypt) and private key (decrypt)
- Slower
- Examples: RSA, ECC, ElGamal

**Example:**
```
Public Key: Everyone can have it
Private Key: Only owner has it

Encrypt: ciphertext = encrypt(plaintext, public_key)
Decrypt: plaintext = decrypt(ciphertext, private_key)
```

**Visual:**
```
Symmetric:
Key → [Encrypt] → Ciphertext → [Decrypt] → Plaintext
      (same key)                (same key)

Asymmetric:
Public Key → [Encrypt] → Ciphertext → [Decrypt] → Plaintext
                                                      ↑
                                                Private Key
```

### AES vs RSA

**AES (Advanced Encryption Standard):**
- **Type**: Symmetric
- **Key sizes**: 128, 192, 256 bits
- **Speed**: Very fast
- **Use**: Bulk data encryption

**RSA (Rivest-Shamir-Adleman):**
- **Type**: Asymmetric
- **Key sizes**: 2048, 4096 bits (much larger)
- **Speed**: Slow
- **Use**: Key exchange, digital signatures

**Hybrid Approach (Common in Practice):**
```
1. Use RSA to exchange AES key
2. Use AES to encrypt actual data
Best of both worlds: Security of RSA, speed of AES
```

### Fast Hash vs Slow Hash

**Fast Hash:**
- **Examples**: MD5, SHA-256
- **Speed**: Very fast
- **Use**: Data integrity, hash tables
- **Problem for passwords**: Easy to brute force

**Slow Hash (Password Hashing):**
- **Examples**: bcrypt, scrypt, Argon2, PBKDF2
- **Speed**: Intentionally slow (configurable)
- **Use**: Password storage
- **Benefit**: Resistant to brute force

**Why slow for passwords?**
```
Fast hash: 1 billion hashes/second
  → Crack password in seconds

Slow hash (bcrypt): 10,000 hashes/second
  → Crack password in years
```

**Example:**
```python
# Fast hash (bad for passwords)
hash = sha256(password)  # 1 billion/sec

# Slow hash (good for passwords)
hash = bcrypt.hash(password, rounds=12)  # 10,000/sec
```

### Encode

**Encode** is a reversible transformation (not for security).

**Characteristics:**
- **Two-way**: Can decode to get original
- **No key**: No secret needed
- **Not secure**: Anyone can decode

**Examples:**

**1. Base64:**
```
Input: "Hello"
Base64: "SGVsbG8="
Decode: "Hello"
```

**Use**: Transfer binary data as text (email attachments, URLs)

**2. URL Encoding:**
```
Input: "Hello World"
Encoded: "Hello%20World"
Decode: "Hello World"
```

**Use**: URLs, form data

**3. Hex Encoding:**
```
Input: "A"
Hex: "41"
Decode: "A"
```

**When to use Encode?**
- Data transmission (not security)
- Data representation
- Compatibility

**Never use for security!**

### Perfect Hash Function

**Perfect Hash Function**: Hash function with no collisions for a specific set of keys.

**Characteristics:**
- **No collisions**: Each input maps to unique output
- **Specific to dataset**: Designed for particular keys
- **Use**: Hash tables with known keys

**Example:**
```
Keys: ["apple", "banana", "cherry"]
Perfect hash:
  "apple" → 0
  "banana" → 1
  "cherry" → 2
No collisions!
```

**Limitation**: Only works for fixed, known set of keys.

### Load Factor of Hashing

**Load Factor**: Ratio of number of elements to number of buckets.

```
Load Factor = Number of Elements / Number of Buckets
```

**Example:**
```
Hash table with 10 buckets
5 elements stored
Load Factor = 5/10 = 0.5
```

**Impact:**
- **Low load factor** (< 0.5): Few collisions, fast lookups
- **High load factor** (> 0.7): Many collisions, slow lookups

**Rehashing:**
```
When load factor > threshold (e.g., 0.75):
  1. Create larger hash table
  2. Rehash all elements
  3. Redistribute to new buckets
```

**Visual:**
```
Before (load factor = 1.0):
Bucket: [0][1][2][3]
        [A][B][C][D]  (collisions)

After rehash (load factor = 0.5):
Bucket: [0][1][2][3][4][5][6][7]
        [A][B][C][D]  (no collisions)
```

---

## SSL/TLS and Certificates

### What is SSL/TLS?

**SSL (Secure Sockets Layer)** / **TLS (Transport Layer Security)** encrypts communication between client and server.

**Purpose:**
- **Encryption**: Data is encrypted in transit
- **Authentication**: Verify server identity
- **Integrity**: Detect tampering

### SSL/TLS Handshake

**Process:**

**1. Client Hello:**
```
Client → Server:
  - Supported TLS versions
  - Supported cipher suites
  - Random number
```

**2. Server Hello:**
```
Server → Client:
  - Selected TLS version
  - Selected cipher suite
  - Server certificate
  - Random number
```

**3. Certificate Verification:**
```
Client verifies server certificate:
  - Is certificate valid?
  - Is certificate from trusted CA?
  - Has certificate expired?
```

**4. Key Exchange:**
```
Client generates pre-master secret
Encrypts with server's public key
Sends to server
```

**5. Session Keys:**
```
Both derive session keys from:
  - Pre-master secret
  - Client random
  - Server random
```

**6. Encrypted Communication:**
```
Now using symmetric encryption (AES)
Fast and secure
```

**Visual Timeline:**
```
Client                    Server
  |                         |
  |---- Client Hello ------>|
  |                         |
  |<--- Server Hello -------|
  |     (Certificate)        |
  |                         |
  |-- Verify Certificate -->|
  |                         |
  |---- Key Exchange ------>|
  |                         |
  |<--- Key Exchange -------|
  |                         |
  |=== Encrypted Data ======|
```

### How to Verify a Certificate

**Certificate Structure:**
```
Certificate:
  - Subject: example.com
  - Issuer: Let's Encrypt
  - Public Key: Server's public key
  - Signature: Signed by CA
  - Validity: Expiration date
```

**Verification Steps:**

**1. Check Validity:**
```
Is current date within validity period?
Certificate valid: 2024-01-01 to 2025-01-01
Today: 2024-06-01 → Valid ✓
```

**2. Check Signature:**
```
1. Extract CA's public key
2. Decrypt certificate signature
3. Compare with certificate hash
4. If match → Signature valid ✓
```

**3. Check Certificate Chain:**
```
Server Certificate
    ↓ (signed by)
Intermediate CA Certificate
    ↓ (signed by)
Root CA Certificate
    ↓ (trusted by)
Operating System / Browser
```

**4. Check Revocation:**
```
Is certificate revoked?
Check OCSP (Online Certificate Status Protocol)
Or CRL (Certificate Revocation List)
```

### Types of Certificates

**1. Domain Validated (DV):**
- **Validation**: Prove domain ownership
- **Use**: Basic websites
- **Trust**: Medium
- **Example**: Let's Encrypt

**2. Organization Validated (OV):**
- **Validation**: Prove organization exists
- **Use**: Business websites
- **Trust**: Higher
- **Example**: DigiCert OV

**3. Extended Validation (EV):**
- **Validation**: Extensive verification
- **Use**: High-security sites
- **Trust**: Highest
- **Visual**: Green address bar (in old browsers)
- **Example**: DigiCert EV

**4. Wildcard Certificate:**
- **Covers**: *.example.com
- **Use**: Multiple subdomains
- **Example**: *.example.com covers a.example.com, b.example.com

**5. Multi-Domain (SAN) Certificate:**
- **Covers**: Multiple domains
- **Use**: Multiple domains on one certificate
- **Example**: example.com, example.org, example.net

### What is CA (Certificate Authority)?

**CA**: Trusted entity that issues digital certificates.

**Role:**
- **Verify**: Verify certificate requester's identity
- **Issue**: Create and sign certificates
- **Revoke**: Revoke compromised certificates

**Trust Chain:**
```
Root CA (self-signed, trusted by OS/browser)
    ↓
Intermediate CA (signed by root)
    ↓
Server Certificate (signed by intermediate)
```

**How to Verify CA Certificate:**

**1. Root CA in Trust Store:**
```
Operating system / browser has list of trusted root CAs
If CA is in list → Trusted ✓
```

**2. Certificate Chain:**
```
Follow chain from server certificate to root CA
If all signatures valid → Trusted ✓
```

**3. Certificate Pinning:**
```
Application stores expected certificate
Compare received certificate with stored one
If match → Trusted ✓
```

### Digital Signature

**Digital Signature**: Proves message authenticity and integrity.

**How it works:**

**1. Signing:**
```
Message: "Hello, World!"
Hash: hash("Hello, World!") = "abc123..."
Sign: encrypt(hash, private_key) = "signature..."
```

**2. Verification:**
```
Receive: Message + Signature
Hash message: hash("Hello, World!") = "abc123..."
Decrypt signature: decrypt("signature...", public_key) = "abc123..."
Compare: If match → Valid ✓
```

**Properties:**
- **Authenticity**: Proves sender identity
- **Integrity**: Detects tampering
- **Non-repudiation**: Sender can't deny sending

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

### HMAC (Hash-Based Message Authentication Code)

**HMAC**: Combines hash function with secret key.

**How it works:**
```
HMAC(message, key) = hash(key + hash(key + message))
```

**Properties:**
- **Authentication**: Proves message from someone with key
- **Integrity**: Detects tampering
- **Symmetric**: Both parties need same key

**Example:**
```
Message: "Hello, World!"
Key: "secretkey"
HMAC: hmac("Hello, World!", "secretkey") = "abc123..."

Send: Message + HMAC
Receiver: Compute HMAC, compare
```

**Use Cases:**
- API authentication
- JWT signatures
- Message verification

**HMAC vs Digital Signature:**
- **HMAC**: Symmetric (same key), faster
- **Digital Signature**: Asymmetric (different keys), stronger

---

## Credential Storage

### Password Storage

**Never store passwords in plain text!**

**Secure Storage:**

**1. Hash with Salt:**
```python
import bcrypt

# Generate salt
salt = bcrypt.gensalt()

# Hash password with salt
hashed = bcrypt.hashpw(password.encode(), salt)

# Store: hashed password
```

**Why salt?**
- Prevents rainbow table attacks
- Same password → Different hashes (different salts)

**Example:**
```
Password: "password123"
Salt 1: "abc"
Hash 1: hash("password123" + "abc") = "hash1"

Salt 2: "xyz"
Hash 2: hash("password123" + "xyz") = "hash2"
Different hashes for same password!
```

**2. Use Slow Hash Functions:**
- bcrypt
- scrypt
- Argon2
- PBKDF2

**3. Verification:**
```python
# User login
input_password = "password123"
stored_hash = "..."

# Verify
if bcrypt.checkpw(input_password.encode(), stored_hash):
    # Password correct
```

### Database Credentials

**Never hardcode in code!**

**Secure Storage:**

**1. Environment Variables:**
```bash
# .env file (not in git)
DB_HOST=localhost
DB_USER=myuser
DB_PASSWORD=secretpassword
```

**2. Secret Management Services:**
- AWS Secrets Manager
- HashiCorp Vault
- Azure Key Vault
- Google Secret Manager

**3. Configuration Files (Encrypted):**
```
Encrypted config file
Decrypt at runtime with master key
```

### API Keys and Tokens

**Storage:**

**1. Environment Variables:**
```bash
API_KEY=sk_live_1234567890
```

**2. Secret Management:**
- Use secret management service
- Rotate regularly

**3. Key Vaults:**
- Store in secure key vault
- Access via API with authentication

### User Information

**Sensitive Data:**
- Encrypt at rest
- Encrypt in transit (HTTPS)
- Access control (who can see what)

**PII (Personally Identifiable Information):**
- Encrypt sensitive fields
- Mask in logs
- GDPR compliance

### Secret Keys

**Application Secrets:**
- Encryption keys
- Signing keys
- API keys

**Best Practices:**
1. **Never commit to git**
2. **Use environment variables**
3. **Rotate regularly**
4. **Use secret management service**
5. **Limit access** (principle of least privilege)

**Key Rotation:**
```
Old Key: Still valid for existing connections
New Key: Used for new connections
Gradually migrate
Eventually revoke old key
```

---

## DDoS Defense

### What is DDoS?

**DDoS (Distributed Denial of Service)**: Overwhelm target with traffic from multiple sources.

**Types:**

**1. Volume-Based (Network Layer):**
- **Method**: Flood with traffic
- **Examples**: UDP flood, ICMP flood
- **Goal**: Exhaust bandwidth

**Visual:**
```
Attacker Botnet:
  Bot 1 ──┐
  Bot 2 ──┤
  Bot 3 ──┼──→ Target Server (overwhelmed)
  Bot 4 ──┤
  Bot 5 ──┘
```

**2. Protocol-Based (Transport Layer):**
- **Method**: Exploit protocol weaknesses
- **Examples**: SYN flood, ACK flood
- **Goal**: Exhaust server resources

**SYN Flood:**
```
Attacker sends many SYN packets
Server allocates resources for each
Never completes handshake
Server runs out of resources
```

**3. Application Layer (Layer 7):**
- **Method**: Target application logic
- **Examples**: HTTP flood, Slowloris
- **Goal**: Exhaust application resources

**Slowloris:**
```
Open many connections
Send headers slowly
Keep connections open
Server runs out of connections
```

**4. Memory-Based:**
- **Method**: Send requests that consume memory
- **Examples**: Large payloads, recursive queries
- **Goal**: Exhaust server memory

### Defense Strategies

**1. Rate Limiting:**
```
Limit requests per IP:
  - 100 requests/minute per IP
  - Block if exceeded
```

**Implementation:**
- Token bucket algorithm
- Sliding window
- Fixed window

**2. CAPTCHA:**
```
Challenge suspicious traffic:
  - Too many requests → Show CAPTCHA
  - Verify human user
```

**3. IP Filtering:**
```
Block known malicious IPs:
  - Blacklist
  - Geographic blocking
  - Known botnet IPs
```

**4. CDN and Load Balancing:**
```
Distribute traffic:
  - CDN absorbs attack
  - Load balancer distributes
  - Multiple servers handle load
```

**5. DDoS Protection Services:**
- **Cloudflare**: DDoS protection, CDN
- **AWS Shield**: AWS DDoS protection
- **Akamai**: DDoS mitigation
- **Cloudflare Magic Transit**: Network-level protection

**6. Scaling:**
```
Auto-scale resources:
  - More servers during attack
  - Absorb traffic
  - Scale down after attack
```

**7. Connection Limits:**
```
Limit connections per IP:
  - Max 10 connections per IP
  - Block if exceeded
```

**8. Timeout Settings:**
```
Short timeouts:
  - Close idle connections quickly
  - Prevent resource exhaustion
```

**9. Monitoring and Alerting:**
```
Monitor traffic patterns:
  - Detect anomalies
  - Alert on attack
  - Automatic mitigation
```

**10. WAF (Web Application Firewall):**
```
Filter malicious requests:
  - Block known attack patterns
  - Rate limiting
  - IP reputation
```

### Multi-Layer Defense

**Defense in Depth:**

**Layer 1: Network (ISP/CDN)**
- Filter at network edge
- Absorb volume attacks

**Layer 2: Infrastructure**
- Load balancers
- Auto-scaling
- Multiple data centers

**Layer 3: Application**
- Rate limiting
- Request validation
- Resource limits

**Layer 4: Monitoring**
- Real-time detection
- Automatic response
- Incident response

### Real-World Example

**Attack:**
```
10,000 bots send 100 requests/second each
= 1,000,000 requests/second to target
Target capacity: 10,000 requests/second
Result: Overwhelmed
```

**Defense:**
```
1. CDN absorbs 90% of traffic
2. Rate limiting: 10 requests/second per IP
3. CAPTCHA for suspicious traffic
4. Auto-scale to 100 servers
5. DDoS protection service filters malicious traffic

Result: Attack mitigated
```

---

## Summary

Security is critical for backend systems. Understanding encryption, hashing, certificates, credential storage, and DDoS defense is essential for building secure applications.

**Key Takeaways:**
- Hash for one-way operations (passwords)
- Encrypt for two-way operations (data)
- Encode for representation (not security)
- Use TLS/SSL for secure communication
- Never store credentials in plain text
- Implement multi-layer DDoS defense
- Security is an ongoing process, not a one-time setup

