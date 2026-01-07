# Webhooks Deep Dive - Complete Understanding

## Table of Contents
1. [What are Webhooks?](#what-are-webhooks)
2. [Webhooks vs Polling](#webhooks-vs-polling)
3. [How Webhooks Work](#how-webhooks-work)
4. [Webhook Security](#webhook-security)
5. [Webhook Delivery](#webhook-delivery)
6. [Webhook Retries](#webhook-retries)
7. [Webhook Idempotency](#webhook-idempotency)
8. [Webhook Signatures](#webhook-signatures)
9. [Webhook Best Practices](#webhook-best-practices)
10. [Common Webhook Patterns](#common-webhook-patterns)
11. [Webhook Testing](#webhook-testing)
12. [Troubleshooting Webhooks](#troubleshooting-webhooks)

---

## What are Webhooks?

### Definition

**Webhook**: HTTP callback mechanism where one service notifies another service about events by making an HTTP POST request to a URL provided by the receiving service.

**Key Concept:**
- **Event-driven**: Triggered by events
- **HTTP POST**: Uses HTTP POST requests
- **Push-based**: Pushes data to receiver
- **Real-time**: Near real-time notifications

### Real-World Analogy

**Webhook = Doorbell:**
- **Event**: Someone at door (event occurs)
- **Notification**: Doorbell rings (webhook fires)
- **Action**: You answer door (receiver processes)

**Without Webhook (Polling):**
```
You: "Is someone at the door?" (check every minute)
You: "Is someone at the door?" (check every minute)
You: "Is someone at the door?" (check every minute)
...
```

**With Webhook:**
```
Someone at door → Doorbell rings → You answer immediately
```

---

## Webhooks vs Polling

### Polling Approach

**How Polling Works:**
```
Client → API: "Any updates?"
API → Client: "No"
  ↓
Wait 5 minutes
  ↓
Client → API: "Any updates?"
API → Client: "No"
  ↓
Wait 5 minutes
  ↓
Client → API: "Any updates?"
API → Client: "Yes, here's update"
```

**Problems:**
- **Inefficient**: Many unnecessary requests
- **Delayed**: Updates delayed (up to polling interval)
- **Wasteful**: Wastes resources

### Webhook Approach

**How Webhooks Work:**
```
Event occurs
  ↓
API → Client: POST /webhook (immediately)
  ↓
Client processes update
```

**Benefits:**
- **Efficient**: Only sends when needed
- **Real-time**: Near real-time delivery
- **Resource-efficient**: Less resource usage

### Comparison

| Aspect | Polling | Webhooks |
|--------|---------|----------|
| **Efficiency** | Low (many requests) | High (only when needed) |
| **Latency** | High (up to interval) | Low (immediate) |
| **Resource Usage** | High | Low |
| **Complexity** | Simple (client-side) | More complex (both sides) |

---

## How Webhooks Work

### Basic Flow

**1. Registration:**
```
Client → API: "Notify me at https://client.com/webhook when event X occurs"
API: Stores webhook URL
```

**2. Event Occurs:**
```
Event: User created account
  ↓
API detects event
```

**3. Webhook Delivery:**
```
API → Client: POST https://client.com/webhook
Body: {
  "event": "user.created",
  "data": { "user_id": 123, "email": "user@example.com" }
}
```

**4. Client Processing:**
```
Client receives webhook
  ↓
Validates signature
  ↓
Processes event
  ↓
Returns 200 OK
```

### Example: Payment Webhook

**Registration:**
```python
# Client registers webhook
POST /api/webhooks
{
    "url": "https://myapp.com/payment-webhook",
    "events": ["payment.completed", "payment.failed"]
}
```

**Event Occurs:**
```python
# Payment completed
payment = process_payment()
if payment.success:
    trigger_webhook("payment.completed", payment.data)
```

**Webhook Delivery:**
```python
# API sends webhook
POST https://myapp.com/payment-webhook
Headers:
    X-Webhook-Event: payment.completed
    X-Webhook-Signature: sha256=...
Body:
{
    "event": "payment.completed",
    "data": {
        "payment_id": "pay_123",
        "amount": 100.00,
        "status": "completed"
    },
    "timestamp": "2024-01-15T10:30:00Z"
}
```

---

## Webhook Security

### Security Concerns

**1. Unauthorized Requests:**
```
Attacker → Client: POST /webhook (fake webhook)
  ↓
Client processes fake event
  ↓
Security breach
```

**2. Replay Attacks:**
```
Attacker captures webhook
  ↓
Replays webhook later
  ↓
Duplicate processing
```

**3. Data Tampering:**
```
Attacker modifies webhook payload
  ↓
Client processes tampered data
  ↓
Data corruption
```

### Security Solutions

**1. Webhook Signatures:**
```
API signs webhook with secret
  ↓
Client verifies signature
  ↓
Only authentic webhooks accepted
```

**2. HTTPS:**
```
Use HTTPS for webhook delivery
  ↓
Encrypts data in transit
  ↓
Prevents interception
```

**3. IP Whitelisting:**
```
Client whitelists API IPs
  ↓
Only requests from whitelisted IPs accepted
  ↓
Prevents unauthorized requests
```

---

## Webhook Delivery

### Delivery Process

**1. Queue Webhook:**
```
Event occurs
  ↓
Queue webhook for delivery
  ↓
Return immediately (don't wait)
```

**2. Deliver Webhook:**
```
Background worker
  ↓
HTTP POST to webhook URL
  ↓
Wait for response
```

**3. Handle Response:**
```
200 OK → Success, mark delivered
4xx/5xx → Retry later
Timeout → Retry later
```

### Implementation

**Basic Delivery:**
```python
import requests
from queue import Queue

webhook_queue = Queue()

def queue_webhook(webhook_url, payload):
    webhook_queue.put((webhook_url, payload))

def deliver_webhook(webhook_url, payload):
    try:
        response = requests.post(
            webhook_url,
            json=payload,
            timeout=10,
            headers={
                "X-Webhook-Event": payload["event"],
                "X-Webhook-Signature": sign_webhook(payload)
            }
        )
        response.raise_for_status()
        return True
    except Exception as e:
        logger.error(f"Webhook delivery failed: {e}")
        return False

# Background worker
def webhook_worker():
    while True:
        webhook_url, payload = webhook_queue.get()
        success = deliver_webhook(webhook_url, payload)
        if not success:
            # Retry logic
            retry_webhook(webhook_url, payload)
```

---

## Webhook Retries

### Why Retries?

**Problem:**
```
Webhook delivery fails
  ↓
Client never receives event
  ↓
Data inconsistency
```

**Solution: Retries**

### Retry Strategy

**Exponential Backoff:**
```
Attempt 1: Immediate
Attempt 2: Wait 1 minute
Attempt 3: Wait 2 minutes
Attempt 4: Wait 4 minutes
Attempt 5: Wait 8 minutes
...
Max attempts: 5
```

**Implementation:**
```python
import time

def retry_webhook(webhook_url, payload, max_attempts=5):
    for attempt in range(max_attempts):
        success = deliver_webhook(webhook_url, payload)
        if success:
            return True
        
        if attempt < max_attempts - 1:
            # Exponential backoff
            delay = 2 ** attempt * 60  # seconds
            time.sleep(delay)
    
    # All attempts failed
    log_failed_webhook(webhook_url, payload)
    return False
```

### Retry Considerations

**1. Idempotency:**
- **Idempotent processing**: Client must handle duplicates
- **Idempotency keys**: Use idempotency keys
- **Safe retries**: Safe to retry

**2. Max Attempts:**
- **Reasonable limit**: Don't retry forever
- **Dead letter queue**: Store failed webhooks
- **Manual review**: Review failed webhooks

**3. Backoff:**
- **Exponential**: Exponential backoff
- **Jitter**: Add jitter to prevent thundering herd
- **Max delay**: Cap maximum delay

---

## Webhook Idempotency

### Why Idempotency?

**Problem:**
```
Webhook delivered
  ↓
Client processes
  ↓
Webhook retried (duplicate)
  ↓
Client processes again
  ↓
Duplicate processing
```

**Solution: Idempotency**

### Idempotency Implementation

**1. Idempotency Key:**
```python
# Include idempotency key in webhook
{
    "event": "payment.completed",
    "idempotency_key": "webhook_pay_123_20240115",
    "data": {...}
}
```

**2. Client-Side Deduplication:**
```python
# Client stores processed webhooks
processed_webhooks = set()

def handle_webhook(payload):
    idempotency_key = payload["idempotency_key"]
    
    if idempotency_key in processed_webhooks:
        # Already processed, ignore
        return {"status": "already_processed"}
    
    # Process webhook
    process_event(payload)
    
    # Mark as processed
    processed_webhooks.add(idempotency_key)
    
    return {"status": "processed"}
```

**3. Database Deduplication:**
```python
# Store idempotency keys in database
def handle_webhook(payload):
    idempotency_key = payload["idempotency_key"]
    
    # Check if already processed
    if webhook_processed(idempotency_key):
        return {"status": "already_processed"}
    
    # Process in transaction
    with db.transaction():
        process_event(payload)
        mark_webhook_processed(idempotency_key)
    
    return {"status": "processed"}
```

---

## Webhook Signatures

### Why Signatures?

**Problem:**
```
Attacker → Client: POST /webhook (fake)
  ↓
Client processes fake webhook
  ↓
Security breach
```

**Solution: Signatures**

### Signature Implementation

**1. Generate Signature:**
```python
import hmac
import hashlib

def sign_webhook(payload, secret):
    # Create signature
    signature = hmac.new(
        secret.encode(),
        json.dumps(payload).encode(),
        hashlib.sha256
    ).hexdigest()
    return signature
```

**2. Include in Webhook:**
```python
payload = {
    "event": "payment.completed",
    "data": {...}
}

signature = sign_webhook(payload, webhook_secret)

# Send webhook with signature
requests.post(
    webhook_url,
    json=payload,
    headers={
        "X-Webhook-Signature": f"sha256={signature}"
    }
)
```

**3. Verify Signature:**
```python
def verify_webhook_signature(payload, signature_header, secret):
    # Extract signature
    expected_signature = signature_header.replace("sha256=", "")
    
    # Compute signature
    computed_signature = hmac.new(
        secret.encode(),
        json.dumps(payload).encode(),
        hashlib.sha256
    ).hexdigest()
    
    # Compare (constant-time comparison)
    return hmac.compare_digest(computed_signature, expected_signature)

# Client verifies
if not verify_webhook_signature(payload, signature, secret):
    return {"error": "Invalid signature"}, 401
```

---

## Webhook Best Practices

### 1. Always Use HTTPS

**Why:**
- **Encryption**: Encrypts data in transit
- **Security**: Prevents interception
- **Trust**: Builds trust

**Implementation:**
```python
# Only accept HTTPS webhooks
if not webhook_url.startswith("https://"):
    raise ValueError("Webhook URL must use HTTPS")
```

### 2. Verify Signatures

**Why:**
- **Authentication**: Verifies sender
- **Security**: Prevents fake webhooks
- **Trust**: Ensures authenticity

**Implementation:**
```python
# Always verify signature
if not verify_signature(payload, signature, secret):
    return {"error": "Invalid signature"}, 401
```

### 3. Make Processing Idempotent

**Why:**
- **Safe retries**: Safe to retry
- **No duplicates**: No duplicate processing
- **Reliability**: More reliable

**Implementation:**
```python
# Use idempotency keys
if idempotency_key in processed:
    return  # Already processed
```

### 4. Return Quickly

**Why:**
- **Timeout prevention**: Prevents timeouts
- **Better UX**: Better user experience
- **Reliability**: More reliable

**Implementation:**
```python
# Queue for async processing
queue_webhook(webhook_url, payload)
return 200  # Return immediately
```

### 5. Log Everything

**Why:**
- **Debugging**: Easier debugging
- **Audit trail**: Audit trail
- **Monitoring**: Better monitoring

**Implementation:**
```python
logger.info(f"Webhook sent: {event} to {webhook_url}")
logger.info(f"Webhook response: {response.status_code}")
```

---

## Common Webhook Patterns

### Pattern 1: Event Sourcing

**Webhook as Event:**
```
Event occurs
  ↓
Webhook sent
  ↓
Client stores as event
  ↓
Replay events to rebuild state
```

### Pattern 2: Pub/Sub

**Webhook as Message:**
```
Event occurs
  ↓
Webhook sent to multiple subscribers
  ↓
Each subscriber processes independently
```

### Pattern 3: Request-Reply

**Webhook with Callback:**
```
Client → API: Request with callback URL
  ↓
API processes
  ↓
API → Client: Webhook with result
```

---

## Webhook Testing

### Testing Strategies

**1. Local Testing:**
```python
# Use ngrok or similar
ngrok http 8080
# Provides public URL: https://abc123.ngrok.io
# Register as webhook URL
```

**2. Webhook Testing Tools:**
- **Webhook.site**: Test webhook endpoints
- **RequestBin**: Capture webhook requests
- **Postman**: Test webhook delivery

**3. Mock Webhooks:**
```python
# Mock webhook delivery
def mock_webhook_delivery(webhook_url, payload):
    # Don't actually send, just log
    logger.info(f"Would send webhook to {webhook_url}: {payload}")
```

---

## Troubleshooting Webhooks

### Common Issues

**1. Webhook Not Received:**
- **Check URL**: Verify webhook URL
- **Check firewall**: Firewall blocking?
- **Check logs**: Check server logs

**2. Webhook Timeout:**
- **Process quickly**: Process quickly
- **Return 200**: Return 200 immediately
- **Async processing**: Process async

**3. Invalid Signature:**
- **Check secret**: Verify secret matches
- **Check payload**: Verify payload format
- **Check algorithm**: Verify algorithm

**4. Duplicate Processing:**
- **Idempotency**: Implement idempotency
- **Deduplication**: Deduplicate webhooks
- **Idempotency keys**: Use idempotency keys

---

## Summary

Webhooks enable real-time, event-driven communication between services. Understanding security, delivery, retries, and best practices is essential for building reliable webhook systems.

**Key Takeaways:**
- **Webhooks**: HTTP callbacks for event notifications
- **vs Polling**: More efficient than polling
- **Security**: Use HTTPS, signatures, IP whitelisting
- **Delivery**: Queue and deliver asynchronously
- **Retries**: Exponential backoff with max attempts
- **Idempotency**: Make processing idempotent
- **Signatures**: Always verify signatures
- **Best practices**: HTTPS, verify, idempotent, fast, log

**Webhook Flow:**
1. Event occurs
2. Queue webhook
3. Deliver webhook
4. Client processes
5. Retry if failed

**Best Practices:**
- Always use HTTPS
- Verify signatures
- Make processing idempotent
- Return quickly
- Log everything

**Common Issues:**
- Webhook not received
- Timeout
- Invalid signature
- Duplicate processing

**Next Steps:**
- Implement webhook delivery
- Add signature verification
- Implement retries
- Make idempotent
- Test thoroughly

