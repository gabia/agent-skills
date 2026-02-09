---
name: java-xmlrpc-guideline
description: Use when designing Java XML-RPC APIs, interfaces, or DTOs. Applies when working with XmlRpcException, void-returning methods, or serialization strategy decisions.
---

# Java XML-RPC API Design Guideline

## Overview

Design XML-RPC APIs with clear exception handling, proper return types, and interoperable serialization.

**Core principle:** Exceptions signal failure, return values signal success. Use JAXB for cross-language compatibility.

## When to Use

- Designing new XML-RPC API interfaces
- Implementing XML-RPC handlers or controllers
- Creating DTOs for XML-RPC request/response
- Reviewing XML-RPC API code
- Migrating from Serializable to JAXB

## Quick Reference

| Scenario | Pattern |
|----------|---------|
| API interface method | `ReturnType method(Param p) throws XmlRpcException` |
| Void-like operation | Return `int`, always `0`, throw exception on failure |
| Success result | Return value (DTO, primitive, etc.) |
| Failure result | Throw `XmlRpcException` |
| DTO serialization | Use JAXB annotations (`@XmlRootElement`, `@XmlAttribute`) |

## Exception Handling

### All API Methods Must Declare XmlRpcException

Every interface method must include `throws XmlRpcException`:

```java
public interface SomeApi {
    SomeDto someAction(SomeParameterDto request) throws XmlRpcException;
}
```

**Why:** XML-RPC library can transmit one exception type to client. Most projects use extension features enabling this. The exception serializes as:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<methodResponse xmlns:ex="http://ws.apache.org/xmlrpc/namespaces/extensions">
    <fault>
        <value>
            <struct>
                <member>
                    <name>faultCode</name>
                    <value><i4>1</i4></value>
                </member>
                <member>
                    <name>faultString</name>
                    <value>failed to execute api</value>
                </member>
            </struct>
        </value>
    </fault>
</methodResponse>
```

**Avoid:** `enabledForException` feature. It serializes Java exceptions including chained exceptions, but only works with Java clients.

## Return Type Guidelines

### Void Methods Must Return int

XML-RPC library doesn't support `void` return type. Use `int` instead:

```java
public interface ServiceControlApi {
    int start(String name) throws XmlRpcException;
}
```

**Rules:**
- Always return `0` on success
- Throw `XmlRpcException` on failure
- Never return magic numbers like `-1`

**Why:** The purpose of void methods is action execution. Success/failure is binary. Use exceptions for failure, not return codes.

### Success vs Failure: Clear Separation

| Outcome | How to Signal |
|---------|---------------|
| Operation succeeded | Return value |
| Query found nothing | Return empty/false (this is success) |
| Operation failed | Throw `XmlRpcException` |

**Example - Query API:**

```java
public interface PublicIpApi {
    boolean isExists(InetAddress address) throws XmlRpcException;
}
```

- IP exists → return `true`
- IP doesn't exist → return `false` (success case - query worked)
- System error during query → throw `XmlRpcException`

**Example - Action API:**

```java
public interface ServiceControlApi {
    int start(String name) throws XmlRpcException;
}
```

- Service started → return `0`
- Service already running → return `0` (if idempotent) or throw (if strict)
- Service failed to start → throw `XmlRpcException`

**Why this matters:**
- Returning error codes in response (like `-1` or error field in DTO) makes XML-RPC request appear successful
- Server logs show success, no stack trace
- Client must inspect response to detect failure
- Debugging becomes difficult

## DTO Serialization

### Use JAXB, Not Serializable

DTOs must use XML structures via JAXB for cross-language compatibility:

```java
// GOOD: JAXB DTO
@XmlRootElement
public class ComplexJaxbDto implements Element {
    @XmlAttribute
    private String type;
    
    @XmlAttribute
    private String number;
    
    private List<NestedDto> nestedDtos;
    
    private ComplexJaxbDto() { }
    
    public static final class NestedDto {
        private String value;
        private NestedDto() { }
    }
}
```

```java
// BAD: Serializable DTO (Java-only)
public class SerializableDto implements Serializable {
    private String value;
    public SerializableDto(String value) {
        this.value = value;
    }
}
```

### Serialization Comparison

**Serializable output (unreadable, Java-only):**
```xml
<ex:serializable>rO0ABXNyACx0aWwueG1scnBj...</ex:serializable>
```

**JAXB output (readable, cross-language):**
```xml
<ex:jaxb>
    <complexJaxbDto number="number" type="type">
        <nestedDtos>
            <value>test</value>
        </nestedDtos>
        <nestedDtos>
            <value>test2</value>
        </nestedDtos>
    </complexJaxbDto>
</ex:jaxb>
```

**Why JAXB:**
- XML-RPC is designed for interoperability
- Serializable limits clients to Java
- JAXB produces human-readable XML
- Long-term maintainability

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Missing `throws XmlRpcException` | Client can't receive errors | Add to all API methods |
| Using `void` return type | XML-RPC library error | Use `int`, return `0` |
| Returning `-1` for errors | Request appears successful | Throw `XmlRpcException` |
| Error codes in DTO fields | Request appears successful | Throw `XmlRpcException` |
| Using `Serializable` | Java-only, unreadable | Use JAXB annotations |
| Using `enabledForException` | Java-only | Avoid, use standard faults |

## Idempotency Decisions

For action APIs, decide if operation should be idempotent:

**Idempotent approach:**
- `start()` on running service → return `0`
- Easier for clients, more forgiving

**Strict approach:**
- `start()` on running service → throw `XmlRpcException`
- Explicit about state transitions

Document your choice in the API contract. Either is valid - consistency matters.

## Checklist

Before completing XML-RPC API design:

- [ ] All interface methods declare `throws XmlRpcException`
- [ ] No `void` return types (use `int` returning `0`)
- [ ] Success cases return values
- [ ] Failure cases throw exceptions (no error codes)
- [ ] No magic return values (`-1`, error fields)
- [ ] Idempotency behavior documented
