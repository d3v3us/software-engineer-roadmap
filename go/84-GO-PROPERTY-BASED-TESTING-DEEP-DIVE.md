# Go Property-Based Testing Deep Dive - Complete Understanding

## Table of Contents
1. [What is Property-Based Testing?](#what-is-property-based-testing)
2. [Why Property-Based Testing Matters](#why-property-based-testing-matters)
3. [Property-Based Testing Libraries](#property-based-testing-libraries)
4. [Writing Property Tests](#writing-property-tests)
5. [QuickCheck Patterns](#quickcheck-patterns)
6. [Test Case Generation](#test-case-generation)
7. [Best Practices](#best-practices)

---

## What is Property-Based Testing?

### Definition

**Property-Based Testing**: Testing approach that verifies properties hold for automatically generated test cases.

**Key Characteristics:**
- **Property verification**: Verifies properties
- **Automatic generation**: Generates test cases
- **Comprehensive**: More comprehensive
- **Edge cases**: Finds edge cases

### Real-World Analogy

**Property-Based Testing = Stress Testing:**
- **Unit tests**: Specific scenarios
- **Property tests**: All scenarios
- **Automatic**: Automatic generation
- **Comprehensive**: Comprehensive coverage

**Programming:**
- **Unit tests**: Specific inputs
- **Property tests**: Generated inputs
- **Properties**: Invariants to verify
- **Coverage**: Better coverage

---

## Why Property-Based Testing Matters?

### Benefits

**1. Comprehensive Coverage:**
```
Automatic generation
  ↓
Property-based testing
  ↓
Better coverage
```

**2. Edge Case Discovery:**
```
Generated inputs
  ↓
Property-based testing
  ↓
Find edge cases
```

**3. Property Verification:**
```
Properties
  ↓
Property-based testing
  ↓
Verify invariants
```

---

## Property-Based Testing Libraries

### gopter

**Installation:**
```bash
go get github.com/leanovate/gopter
```

**Features:**
- **Property testing**: Property-based testing
- **Generators**: Built-in generators
- **Shrinking**: Test case shrinking

### rapid

**Installation:**
```bash
go get pgregory.net/rapid
```

**Features:**
- **Fast**: Fast generation
- **Comprehensive**: Comprehensive testing
- **Go 1.18+**: Uses generics

---

## Writing Property Tests

### Basic Property Test

**Example with gopter:**
```go
import "github.com/leanovate/gopter"
import "github.com/leanovate/gopter/gen"
import "github.com/leanovate/gopter/prop"

func TestReverseProperty(t *testing.T) {
    properties := gopter.NewProperties(nil)
    
    properties.Property("reverse twice is identity", prop.ForAll(
        func(s string) bool {
            return reverse(reverse(s)) == s
        },
        gen.AlphaString(),
    ))
    
    properties.TestingRun(t)
}
```

### Property Test with rapid

**Example:**
```go
import "pgregory.net/rapid"

func TestReverseProperty(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        s := rapid.String().Draw(t, "s")
        reversed := reverse(s)
        reversedTwice := reverse(reversed)
        if reversedTwice != s {
            t.Fatalf("reverse(reverse(%q)) = %q, want %q", s, reversedTwice, s)
        }
    })
}
```

---

## QuickCheck Patterns

### Pattern 1: Round-Trip Property

**Round-trip:**
```go
properties.Property("encode/decode round-trip", prop.ForAll(
    func(data []byte) bool {
        encoded := encode(data)
        decoded := decode(encoded)
        return bytes.Equal(data, decoded)
    },
    gen.SliceOf(gen.UInt8()),
))
```

### Pattern 2: Invariant Property

**Invariant:**
```go
properties.Property("sum is always positive", prop.ForAll(
    func(numbers []int) bool {
        sum := 0
        for _, n := range numbers {
            sum += n
        }
        return sum >= 0 || len(numbers) == 0
    },
    gen.SliceOf(gen.IntRange(0, 100)),
))
```

### Pattern 3: Commutative Property

**Commutative:**
```go
properties.Property("addition is commutative", prop.ForAll(
    func(a, b int) bool {
        return add(a, b) == add(b, a)
    },
    gen.Int(),
    gen.Int(),
))
```

---

## Test Case Generation

### Custom Generators

**Custom generator:**
```go
func genPoint() gopter.Gen {
    return gen.Struct(reflect.TypeOf(Point{}), map[string]gopter.Gen{
        "X": gen.IntRange(0, 100),
        "Y": gen.IntRange(0, 100),
    })
}

properties.Property("point distance", prop.ForAll(
    func(p Point) bool {
        d := p.Distance()
        return d >= 0
    },
    genPoint(),
))
```

### Constrained Generators

**Constrained:**
```go
gen.IntRange(0, 100)  // Range
gen.AlphaString()     // Pattern
gen.SliceOf(gen.Int()) // Slice
```

---

## Best Practices

### 1. Define Clear Properties

**Why:**
- **Clarity**: Clear intent
- **Correctness**: Correct properties
- **Maintenance**: Easier maintenance

**Guidelines:**
- **Properties**: Define clear properties
- **Invariants**: Test invariants
- **Document**: Document properties

### 2. Use Appropriate Generators

**Why:**
- **Coverage**: Better coverage
- **Efficiency**: More efficient
- **Relevance**: More relevant

**Guidelines:**
- **Generators**: Use appropriate generators
- **Constraints**: Apply constraints
- **Custom**: Create custom generators

### 3. Shrink Test Cases

**Why:**
- **Debugging**: Easier debugging
- **Minimal**: Minimal test cases
- **Understanding**: Better understanding

**Guidelines:**
- **Shrinking**: Enable shrinking
- **Minimal**: Find minimal cases
- **Debug**: Use for debugging

### 4. Combine with Unit Tests

**Why:**
- **Complementary**: Complementary testing
- **Coverage**: Better coverage
- **Quality**: Better quality

**Guidelines:**
- **Unit tests**: Write unit tests
- **Property tests**: Add property tests
- **Combine**: Combine both

---

## Summary

Property-based testing enables comprehensive testing with automatically generated test cases. Understanding property-based testing libraries, writing property tests, QuickCheck patterns, test case generation, and best practices is crucial for comprehensive testing.

**Key Takeaways:**
- **Property-based testing**: Testing approach verifying properties (property verification, automatic generation, comprehensive, edge cases)
- **Property-based testing libraries**: gopter (property testing, generators, shrinking), rapid (fast, comprehensive, Go 1.18+)
- **Writing property tests**: Basic property test (gopter example, rapid example)
- **QuickCheck patterns**: Round-trip property (encode/decode), invariant property (sum positive), commutative property (addition commutative)
- **Test case generation**: Custom generators (genPoint, custom types), constrained generators (IntRange, AlphaString, SliceOf)
- **Best practices**: Define clear properties, use appropriate generators, shrink test cases, combine with unit tests

**Property-Based Testing Benefits:**
- **Comprehensive coverage**: Better coverage
- **Edge case discovery**: Find edge cases
- **Property verification**: Verify invariants

**Best Practices:**
- Define clear properties
- Use appropriate generators
- Shrink test cases
- Combine with unit tests

**Next Steps:**
- Learn property-based testing
- Practice writing property tests
- Use QuickCheck patterns
- Apply best practices

