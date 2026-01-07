# Credential Storage Deep Dive - Complete Understanding

## Table of Contents
1. [The Credential Storage Problem](#the-credential-storage-problem)
2. [Password Storage - Hashing and Salting](#password-storage---hashing-and-salting)
3. [Database Credentials - Protecting Access](#database-credentials---protecting-access)
4. [API Keys and Tokens - Secure Management](#api-keys-and-tokens---secure-management)
5. [Secret Management Services](#secret-management-services)
6. [Key Rotation - Keeping Secrets Fresh](#key-rotation---keeping-secrets-fresh)
7. [Environment Variables - Best Practices](#environment-variables---best-practices)
8. [Hardware Security Modules (HSM)](#hardware-security-modules-hsm)
9. [Common Mistakes and How to Avoid Them](#common-mistakes-and-how-to-avoid-them)

---

## The Credential Storage Problem

### Why Credentials Are Sensitive

**Credentials Provide Access:**
```
Password → User account access
API key → Service access
Database credentials → Data access
```

**If Compromised:**
- **Unauthorized access**: Attacker gains access
- **Data breach**: Sensitive data exposed
- **System compromise**: Full system access
- **Reputation damage**: Loss of trust

### Never Store in Plain Text

**Common Mistakes:**
```python
# BAD - Never do this!
password = "mypassword123"
db_password = "secret123"
api_key = "sk_live_1234567890"
```

**Problems:**
- **Code repository**: Committed to git
- **Logs**: May appear in logs
- **Memory dumps**: Visible in memory
- **Database breaches**: Stored in database

---

## Password Storage - Hashing and Salting

### Why Hash Passwords?

**Problem:**
```
Store password in plain text
Database breached
All passwords exposed
```

**Solution: Hash**
```
Store hash, not password
Even if database breached
Passwords not exposed
```

### Hashing Process

**Registration:**
```
1. User enters password
2. Hash password: hash = hash_function(password)
3. Store hash in database
4. Discard original password
```

**Login:**
```
1. User enters password
2. Hash entered password
3. Compare with stored hash
4. If match → Correct password
```

### The Salt Problem

**Problem Without Salt:**
```
Same password → Same hash
"password123" → Always same hash
Rainbow tables: Precomputed hashes
```

**Solution: Salt**
```
Password: "mypassword123"
Salt: "randomsalt456"
Hash: hash("mypassword123" + "randomsalt456")
Store: hash + salt

Same password, different salt → Different hash!
```

### Password Hashing Algorithms

**1. bcrypt:**
```python
import bcrypt

# Hash password
salt = bcrypt.gensalt()
hashed = bcrypt.hashpw(password.encode(), salt)

# Verify
if bcrypt.checkpw(password.encode(), hashed):
    # Correct password
```

**Characteristics:**
- **Slow by design**: Resistant to brute force
- **Automatic salt**: Generates salt automatically
- **Configurable cost**: Adjustable difficulty

**2. Argon2:**
```
Winner of Password Hashing Competition
Memory-hard: Resistant to hardware attacks
Recommended for new systems
```

**3. scrypt:**
```
Memory-hard function
Good for passwords
Configurable parameters
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

## Database Credentials - Protecting Access

### The Problem

**Database Credentials:**
```
Host: localhost
User: dbuser
Password: secretpassword
```

**If Exposed:**
- **Full database access**: Attacker can read/write all data
- **Data breach**: All data compromised
- **System compromise**: Can modify system

### Secure Storage Methods

**1. Environment Variables:**
```bash
# .env file (not in git!)
DB_HOST=localhost
DB_USER=dbuser
DB_PASSWORD=secretpassword
```

**Application reads:**
```python
import os
db_password = os.getenv('DB_PASSWORD')
```

**2. Secret Management Services:**
```
AWS Secrets Manager
HashiCorp Vault
Azure Key Vault
Google Secret Manager
```

**3. Configuration Files (Encrypted):**
```
Encrypt config file
Decrypt at runtime with master key
Store master key securely
```

### Best Practices

**1. Never in Code:**
```
Don't hardcode credentials
Don't commit to git
```

**2. Use Secret Management:**
```
Centralized secret storage
Access control
Audit logging
```

**3. Principle of Least Privilege:**
```
Minimum permissions needed
Separate credentials for different purposes
```

---

## API Keys and Tokens - Secure Management

### API Key Types

**1. Public Keys:**
```
Can be exposed (client-side)
Limited functionality
Example: Stripe publishable key
```

**2. Secret Keys:**
```
Must be kept secret
Full access
Example: Stripe secret key
```

### Secure Storage

**1. Environment Variables:**
```bash
API_KEY=sk_live_1234567890
```

**2. Secret Management:**
```
Store in secret management service
Retrieve at runtime
Rotate regularly
```

**3. Key Vaults:**
```
Encrypted storage
Access via API
Audit access
```

### Token Management

**JWT Tokens:**
```
Store signing key securely
Rotate keys periodically
Use short expiration
```

**OAuth Tokens:**
```
Store refresh tokens securely
Encrypt if stored in database
Rotate regularly
```

---

## Secret Management Services

### AWS Secrets Manager

**Features:**
```
Centralized storage
Automatic rotation
Encryption at rest
Access control (IAM)
Audit logging
```

**Usage:**
```python
import boto3

client = boto3.client('secretsmanager')
secret = client.get_secret_value(SecretId='db-credentials')
credentials = json.loads(secret['SecretString'])
```

### HashiCorp Vault

**Features:**
```
Open source
Dynamic secrets
Encryption as a service
Access policies
```

**Usage:**
```python
import hvac

client = hvac.Client(url='https://vault.example.com')
secret = client.secrets.kv.v2.read_secret_version(path='db-credentials')
password = secret['data']['data']['password']
```

### Comparison

| Feature | AWS Secrets Manager | HashiCorp Vault |
|--------|---------------------|-----------------|
| **Hosting** | Managed | Self-hosted or managed |
| **Rotation** | Automatic | Manual or scripts |
| **Cost** | Pay per secret | Open source (free) |
| **Integration** | AWS services | Multi-cloud |

---

## Key Rotation - Keeping Secrets Fresh

### Why Rotate?

**Reasons:**
- **Compromised keys**: If key leaked, rotate
- **Time-based**: Rotate periodically
- **Compliance**: Regulations require rotation
- **Best practice**: Limit exposure window

### Rotation Process

**1. Generate New Key:**
```
Create new key
Keep old key active
```

**2. Update Systems:**
```
Gradually update to new key
Both keys work during transition
```

**3. Monitor:**
```
Verify new key works
Check for errors
```

**4. Revoke Old Key:**
```
After all systems updated
Revoke old key
```

### Automated Rotation

**AWS Secrets Manager:**
```
Automatic rotation
Lambda function rotates
No manual intervention
```

**Custom Scripts:**
```
Script rotates keys
Updates systems
Validates rotation
```

---

## Environment Variables - Best Practices

### Secure Usage

**1. Never Commit:**
```
Add .env to .gitignore
Never commit secrets
```

**2. Use Different Files:**
```
.env.development
.env.production
.env.local (gitignored)
```

**3. Validate:**
```
Check all required variables present
Fail fast if missing
```

**4. Don't Log:**
```
Never log environment variables
Mask in logs if needed
```

### .env File Structure

```
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=dbuser
DB_PASSWORD=secretpassword

# API Keys
STRIPE_SECRET_KEY=sk_live_...
AWS_ACCESS_KEY_ID=AKIA...
AWS_SECRET_ACCESS_KEY=...

# Application
SECRET_KEY=application-secret-key
```

---

## Hardware Security Modules (HSM)

### What is HSM?

**HSM**: Physical device for secure key storage and operations.

**Characteristics:**
- **Physical security**: Tamper-resistant
- **Keys never leave**: Operations done in HSM
- **High security**: Highest level of protection
- **Expensive**: Hardware cost

### Use Cases

**1. Certificate Authority:**
```
Root CA private keys
Highest security needed
```

**2. Payment Processing:**
```
Encryption keys
PCI compliance
```

**3. High-Security Applications:**
```
Government systems
Financial institutions
```

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Hardcoding Secrets

**Bad:**
```python
api_key = "sk_live_1234567890"  # In code!
```

**Good:**
```python
api_key = os.getenv('API_KEY')  # From environment
```

### Mistake 2: Committing to Git

**Bad:**
```
Commit .env file
Secrets in repository
```

**Good:**
```
Add .env to .gitignore
Use .env.example (without secrets)
```

### Mistake 3: Logging Secrets

**Bad:**
```python
logger.info(f"API key: {api_key}")  # Logs secret!
```

**Good:**
```python
logger.info("API key: ***")  # Masked
```

### Mistake 4: Weak Password Hashing

**Bad:**
```python
hash = hashlib.sha256(password.encode()).hexdigest()  # Fast hash!
```

**Good:**
```python
hash = bcrypt.hashpw(password.encode(), bcrypt.gensalt())  # Slow hash
```

### Mistake 5: No Key Rotation

**Bad:**
```
Same key for years
Never rotated
```

**Good:**
```
Rotate keys periodically
Automated rotation
Monitor rotation
```

---

## Summary

Credential storage is critical for security. Understanding password hashing, secret management, and best practices is essential for backend engineers.

**Key Takeaways:**
- Never store credentials in plain text
- Hash passwords with slow hashes (bcrypt, Argon2)
- Use salts to prevent rainbow table attacks
- Store secrets in environment variables or secret management
- Rotate keys regularly
- Use principle of least privilege
- Never commit secrets to git
- Never log secrets
- Use secret management services for production
- Consider HSM for high-security applications

**Next Steps:**
- Audit your credential storage
- Implement proper password hashing
- Set up secret management
- Implement key rotation
- Train team on best practices
- Monitor for exposed secrets
- Regularly review and update practices

