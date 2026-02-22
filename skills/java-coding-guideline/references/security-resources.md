# Java Security & Resource Management Deep Dive

## Password Handling

Never store passwords in plain text. Use BCrypt or Argon2 for hashing.

### ✅ Do

```java
public final class PasswordService {

    private static final BCryptPasswordEncoder ENCODER = new BCryptPasswordEncoder();

    @NonNull
    public String hash(@NonNull char[] password) {
        try {
            return ENCODER.encode(new String(password));
        } finally {
            Arrays.fill(password, '\0');  // Clear from memory immediately
        }
    }

    public boolean verify(@NonNull char[] password, @NonNull String hashedPassword) {
        try {
            return ENCODER.matches(new String(password), hashedPassword);
        } finally {
            Arrays.fill(password, '\0');
        }
    }
}
```

**Key Points**:
- Use `char[]` instead of `String` for passwords (can be zeroed out)
- Use BCrypt with work factor 10-12
- Clear password from memory immediately after use
- Use constant-time comparison (BCrypt handles this)

### ❌ Don't

```java
public final class PasswordService {

    @NonNull
    public String hash(@NonNull String password) {
        // Do not use MD5/SHA1/SHA256 alone - vulnerable to rainbow table attacks
        return DigestUtils.sha256Hex(password);
    }

    public boolean verify(@NonNull String password, @NonNull String stored) {
        // Avoid direct comparison vulnerable to timing attacks
        return hash(password).equals(stored);
    }
}
```

---

## SQL Injection Prevention

Always use `PreparedStatement`. Never concatenate user input into SQL.

### ✅ Do

```java
@Nullable
public User findByEmail(@NonNull String email) throws SQLException {
    String sql = "SELECT * FROM users WHERE email = ?";
    try (var conn = dataSource.getConnection();
         var stmt = conn.prepareStatement(sql)) {
        stmt.setString(1, email);
        try (var rs = stmt.executeQuery()) {
            return rs.next() ? mapUser(rs) : null;
        }
    }
}
```

**Additional Protection**:
- Use ORM (JPA/Hibernate) with named parameters
- Validate input length and format
- Use stored procedures for complex queries
- Never disable SQL injection protection in ORM

### ❌ Don't

```java
@Nullable
public User findByEmail(@NonNull String email) throws SQLException {
    // SQL injection vulnerability!
    String sql = "SELECT * FROM users WHERE email = '" + email + "'";
    try (var conn = dataSource.getConnection();
         var stmt = conn.createStatement();
         var rs = stmt.executeQuery(sql)) {
        return rs.next() ? mapUser(rs) : null;
    }
}
```

---

## Secure Random

Use `SecureRandom` for security-sensitive random values.

### ✅ Do

```java
public final class TokenGenerator {

    private static final SecureRandom RANDOM = new SecureRandom();

    @NonNull
    public String generateToken(int length) {
        byte[] bytes = new byte[length];
        RANDOM.nextBytes(bytes);
        return Base64.getUrlEncoder().withoutPadding().encodeToString(bytes);
    }
}
```

**Use Cases for SecureRandom**:
- Session tokens
- CSRF tokens
- API keys
- Password reset tokens
- Encryption initialization vectors (IVs)
- Salt for password hashing

### ❌ Don't

```java
public final class TokenGenerator {

    @NonNull
    public String generateToken() {
        // Predictable random value - do not use for security
        return String.valueOf(new Random().nextLong());
    }
}
```

---

## PII (Personally Identifiable Information) Logging

Never log sensitive personal information.

### ✅ Do

```java
public void processPayment(@NonNull PaymentRequest request) {
    // Mask sensitive data
    logger.info("Processing payment. userId={}, amount={}, cardLast4={}",
        request.userId(),
        request.amount(),
        maskCard(request.cardNumber()));
}

@NonNull
private String maskCard(@NonNull String cardNumber) {
    return "****" + cardNumber.substring(cardNumber.length() - 4);
}
```

**PII Categories to Mask**:
- Credit card numbers (show last 4 digits only)
- Email addresses (mask middle: `u***@example.com`)
- Phone numbers (mask middle digits)
- Social security numbers (never log)
- Passwords/tokens (never log)
- Medical records (never log)
- Biometric data (never log)

### ❌ Don't

```java
public void processPayment(@NonNull PaymentRequest request) {
    // Do not log PII!
    logger.info("Processing payment: {}", request);  // Card number may be exposed
    logger.debug("Card number: {}", request.cardNumber());  // Never do this
}
```

---

## Additional Security Best Practices

### Input Validation

Always validate and sanitize user input:

```java
public User createUser(@NonNull String email, @NonNull String name) {
    // Validate email format
    if (!email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$")) {
        throw new IllegalArgumentException("Invalid email format");
    }
    
    // Length limit
    if (name.length() > 100) {
        throw new IllegalArgumentException("Name too long");
    }
    
    // HTML escape (prevent XSS)
    String safeName = HtmlUtils.htmlEscape(name);
    
    return new User(email, safeName);
}
```

---

### File Upload Security

```java
public void uploadFile(@NonNull MultipartFile file) throws IOException {
    // File size limit
    if (file.getSize() > 10_000_000) {  // 10MB
        throw new IllegalArgumentException("File too large");
    }
    
    // Extension whitelist
    String ext = FilenameUtils.getExtension(file.getOriginalFilename());
    if (!List.of("jpg", "png", "pdf").contains(ext.toLowerCase())) {
        throw new IllegalArgumentException("Invalid file type");
    }
    
    // Generate a safe filename (prevent directory traversal)
    String safeFilename = UUID.randomUUID().toString() + "." + ext;
    Path uploadPath = Paths.get("/uploads").resolve(safeFilename);
    
    // Validate after path normalization
    if (!uploadPath.normalize().startsWith("/uploads")) {
        throw new SecurityException("Invalid upload path");
    }
    
    file.transferTo(uploadPath);
}
```

---

### Deserialization Safety

Avoid Java serialization. If unavoidable, use whitelist filtering:

```java
// Safer alternative: JSON (Jackson, Gson)
ObjectMapper mapper = new ObjectMapper();
User user = mapper.readValue(json, User.class);

// Avoid Java serialization; if unavoidable:
ObjectInputStream ois = new ObjectInputStream(inputStream) {
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
        // Whitelist validation
        if (!ALLOWED_CLASSES.contains(desc.getName())) {
            throw new InvalidClassException("Unauthorized deserialization attempt", desc.getName());
        }
        return super.resolveClass(desc);
    }
};
```

---

## Summary Checklist

### Security
- ✅ Use BCrypt/Argon2 for password hashing
- ✅ Use SecureRandom for security-sensitive random values
- ✅ Mask PII in logs
- ✅ Validate and sanitize all user input
- ✅ Prefer JSON over Java serialization
