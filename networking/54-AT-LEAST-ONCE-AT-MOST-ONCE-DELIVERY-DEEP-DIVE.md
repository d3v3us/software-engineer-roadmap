# At-Least-Once and At-Most-Once Delivery Deep Dive

## Table of Contents
1. [Delivery Semantics Overview](#delivery-semantics-overview)
2. [At-Least-Once Delivery](#at-least-once-delivery)
3. [At-Most-Once Delivery](#at-most-once-delivery)
4. [Comparison](#comparison)
5. [Implementation](#implementation)
6. [Best Practices](#best-practices)

---

## Delivery Semantics Overview

**Three Delivery Semantics:**
- **At-Most-Once**: Message delivered at most once (may be lost, no duplicates)
- **At-Least-Once**: Message delivered at least once (may be duplicated, no losses)
- **Exactly-Once**: Message delivered exactly once (no duplicates, no losses)

---

## At-Least-Once Delivery

### Definition

**At-Least-Once**: Guarantee that message is delivered at least once (may be duplicated).

**Characteristics:**
- **No losses**: No message losses
- **May duplicate**: Messages may be duplicated
- **Idempotency**: Requires idempotent processing

### Implementation

**Retry on Failure:**
```
Send message
  ↓
If no acknowledgment: Retry
  ↓
Continue until acknowledged
```

**Use Cases:**
- **Critical messages**: Critical messages that must not be lost
- **Idempotent processing**: When processing is idempotent
- **Reliability**: When reliability is priority

---

## At-Most-Once Delivery

### Definition

**At-Most-Once**: Guarantee that message is delivered at most once (may be lost).

**Characteristics:**
- **No duplicates**: No duplicate messages
- **May lose**: Messages may be lost
- **Simple**: Simple implementation

### Implementation

**No Retries:**
```
Send message
  ↓
If no acknowledgment: Don't retry
  ↓
Accept loss
```

**Use Cases:**
- **Non-critical**: Non-critical messages
- **High volume**: High volume messages
- **Performance**: When performance is priority

---

## Comparison

| Aspect | At-Most-Once | At-Least-Once | Exactly-Once |
|--------|--------------|---------------|--------------|
| Duplicates | No | Yes | No |
| Losses | Yes | No | No |
| Complexity | Low | Medium | High |
| Use Case | Non-critical | Critical | Critical + No duplicates |

---

## Implementation

### At-Least-Once Example

```java
// Producer with retries
Properties props = new Properties();
props.put("retries", Integer.MAX_VALUE);
props.put("acks", "all");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("topic", "key", "value"));
```

### At-Most-Once Example

```java
// Producer without retries
Properties props = new Properties();
props.put("retries", 0);
props.put("acks", "0");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.send(new ProducerRecord<>("topic", "key", "value"));
```

---

## Best Practices

1. **Choose based on requirements**: At-least-once for critical, at-most-once for non-critical
2. **Implement idempotency**: For at-least-once, ensure idempotent processing
3. **Monitor delivery**: Monitor message delivery rates

---

## Summary

Understanding at-least-once and at-most-once delivery semantics is crucial for message systems. Choose based on requirements and implement accordingly.

