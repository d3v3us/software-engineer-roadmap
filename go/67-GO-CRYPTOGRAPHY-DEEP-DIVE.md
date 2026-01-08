# Go Cryptography Deep Dive - Complete Understanding

## Table of Contents
1. [What is Cryptography in Go?](#what-is-cryptography-in-go)
2. [Why Cryptography Matters](#why-cryptography-matters)
3. [Crypto Package](#crypto-package)
4. [Hashing](#hashing)
5. [Encryption and Decryption](#encryption-and-decryption)
6. [Digital Signatures](#digital-signatures)
7. [TLS Implementation](#tls-implementation)
8. [Best Practices](#best-practices)

---

## What is Cryptography in Go?

### Definition

**Cryptography**: Practice of secure communication and data protection using mathematical algorithms.

**Key Characteristics:**
- **Security**: Data security
- **Encryption**: Data encryption
- **Authentication**: Authentication
- **Integrity**: Data integrity

### Real-World Analogy

**Cryptography = Safe:**
- **Data**: Valuables
- **Safe**: Cryptography
- **Lock**: Encryption
- **Key**: Decryption key

**Programming:**
- **Data**: Sensitive data
- **Cryptography**: Protection
- **Algorithms**: Cryptographic algorithms
- **Security**: Security

---

## Why Cryptography Matters?

### Benefits

**1. Security:**
```
Sensitive data
  ↓
Cryptography
  ↓
Protected data
```

**2. Privacy:**
```
Private data
  ↓
Cryptography
  ↓
Encrypted data
```

**3. Integrity:**
```
Data integrity
  ↓
Cryptography
  ↓
Verified data
```

---

## Crypto Package

### Import

```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
    "crypto/sha256"
    "crypto/hmac"
    "crypto/rsa"
    "crypto/ecdsa"
)
```

### Available Algorithms

**Hashing:**
- `crypto/sha256`: SHA-256
- `crypto/sha512`: SHA-512
- `crypto/md5`: MD5 (deprecated)

**Encryption:**
- `crypto/aes`: AES encryption
- `crypto/des`: DES (deprecated)

**Signatures:**
- `crypto/rsa`: RSA signatures
- `crypto/ecdsa`: ECDSA signatures

---

## Hashing

### SHA-256

**Example:**
```go
import (
    "crypto/sha256"
    "fmt"
)

func hash(data []byte) []byte {
    h := sha256.New()
    h.Write(data)
    return h.Sum(nil)
}

func main() {
    data := []byte("hello")
    hash := hash(data)
    fmt.Printf("%x\n", hash)
}
```

### HMAC

**Example:**
```go
import (
    "crypto/hmac"
    "crypto/sha256"
)

func hmacHash(key, data []byte) []byte {
    h := hmac.New(sha256.New, key)
    h.Write(data)
    return h.Sum(nil)
}
```

---

## Encryption and Decryption

### AES Encryption

**Example:**
```go
import (
    "crypto/aes"
    "crypto/cipher"
    "crypto/rand"
)

func encrypt(key, plaintext []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonce := make([]byte, gcm.NonceSize())
    if _, err := rand.Read(nonce); err != nil {
        return nil, err
    }
    
    ciphertext := gcm.Seal(nonce, nonce, plaintext, nil)
    return ciphertext, nil
}

func decrypt(key, ciphertext []byte) ([]byte, error) {
    block, err := aes.NewCipher(key)
    if err != nil {
        return nil, err
    }
    
    gcm, err := cipher.NewGCM(block)
    if err != nil {
        return nil, err
    }
    
    nonceSize := gcm.NonceSize()
    nonce, ciphertext := ciphertext[:nonceSize], ciphertext[nonceSize:]
    
    plaintext, err := gcm.Open(nil, nonce, ciphertext, nil)
    return plaintext, err
}
```

---

## Digital Signatures

### RSA Signatures

**Example:**
```go
import (
    "crypto/rand"
    "crypto/rsa"
    "crypto/sha256"
)

func sign(privateKey *rsa.PrivateKey, message []byte) ([]byte, error) {
    hashed := sha256.Sum256(message)
    return rsa.SignPKCS1v15(rand.Reader, privateKey, crypto.SHA256, hashed[:])
}

func verify(publicKey *rsa.PublicKey, message, signature []byte) error {
    hashed := sha256.Sum256(message)
    return rsa.VerifyPKCS1v15(publicKey, crypto.SHA256, hashed[:], signature)
}
```

### ECDSA Signatures

**Example:**
```go
import (
    "crypto/ecdsa"
    "crypto/rand"
    "crypto/sha256"
)

func signECDSA(privateKey *ecdsa.PrivateKey, message []byte) ([]byte, error) {
    hashed := sha256.Sum256(message)
    r, s, err := ecdsa.Sign(rand.Reader, privateKey, hashed[:])
    if err != nil {
        return nil, err
    }
    // Encode r and s
    return encodeSignature(r, s), nil
}
```

---

## TLS Implementation

### TLS Server

**Example:**
```go
import (
    "crypto/tls"
    "net/http"
)

func main() {
    cert, err := tls.LoadX509KeyPair("cert.pem", "key.pem")
    if err != nil {
        log.Fatal(err)
    }
    
    config := &tls.Config{
        Certificates: []tls.Certificate{cert},
    }
    
    server := &http.Server{
        TLSConfig: config,
    }
    
    server.ListenAndServeTLS("", "")
}
```

### TLS Client

**Example:**
```go
import (
    "crypto/tls"
    "net/http"
)

func main() {
    tr := &http.Transport{
        TLSClientConfig: &tls.Config{
            InsecureSkipVerify: false,
        },
    }
    
    client := &http.Client{Transport: tr}
    resp, err := client.Get("https://example.com")
}
```

---

## Best Practices

### 1. Use Strong Algorithms

**Why:**
- **Security**: Better security
- **Protection**: Better protection
- **Standards**: Follow standards

**Guidelines:**
- **AES-256**: Use AES-256 for encryption
- **SHA-256/512**: Use SHA-256 or SHA-512
- **RSA 2048+**: Use RSA 2048+ or ECDSA

### 2. Use Random Keys

**Why:**
- **Security**: Better security
- **Unpredictability**: Unpredictable
- **Protection**: Better protection

**Guidelines:**
- **crypto/rand**: Use crypto/rand
- **Never**: Never use predictable keys
- **Length**: Use appropriate key length

### 3. Protect Keys

**Why:**
- **Security**: Key security
- **Protection**: Key protection
- **Access**: Control access

**Guidelines:**
- **Storage**: Secure storage
- **Access**: Limit access
- **Rotation**: Rotate keys

### 4. Use Authenticated Encryption

**Why:**
- **Security**: Better security
- **Integrity**: Data integrity
- **Protection**: Better protection

**Guidelines:**
- **GCM**: Use GCM mode
- **HMAC**: Use HMAC
- **Authenticated**: Always authenticate

---

## Summary

Cryptography is essential for secure Go applications. Understanding crypto package, hashing, encryption/decryption, digital signatures, TLS implementation, and best practices is crucial for security.

**Key Takeaways:**
- **Cryptography**: Practice of secure communication (security, encryption, authentication, integrity)
- **Crypto package**: Available algorithms (hashing: SHA-256/512, encryption: AES, signatures: RSA/ECDSA)
- **Hashing**: SHA-256 (hash function), HMAC (keyed hash)
- **Encryption and decryption**: AES encryption (GCM mode, nonce, authenticated encryption)
- **Digital signatures**: RSA signatures (SignPKCS1v15, VerifyPKCS1v15), ECDSA signatures (Sign, Verify)
- **TLS implementation**: TLS server (LoadX509KeyPair, TLSConfig), TLS client (TLSClientConfig)
- **Best practices**: Use strong algorithms, use random keys, protect keys, use authenticated encryption

**Cryptography Benefits:**
- **Security**: Data security
- **Privacy**: Data privacy
- **Integrity**: Data integrity

**Best Practices:**
- Use strong algorithms
- Use random keys
- Protect keys
- Use authenticated encryption

**Next Steps:**
- Learn crypto package
- Practice hashing
- Practice encryption
- Apply best practices

