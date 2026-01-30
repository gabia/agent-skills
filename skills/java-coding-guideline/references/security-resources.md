# Java Security & Resource Management Deep Dive

**When to use this guide**: Security-sensitive code (authentication, encryption, PII)

---

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
            Arrays.fill(password, '\0');  // 메모리에서 즉시 삭제
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
        // MD5, SHA1, SHA256 단독 사용 금지 - 레인보우 테이블 공격에 취약
        return DigestUtils.sha256Hex(password);
    }

    public boolean verify(@NonNull String password, @NonNull String stored) {
        // 타이밍 공격에 취약한 직접 비교 금지
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
    // SQL 인젝션 취약점!
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
        // 예측 가능한 난수 - 보안 용도 사용 금지
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
    // 민감 정보 마스킹
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
    // PII 로깅 금지!
    logger.info("Processing payment: {}", request);  // 카드번호 노출 가능
    logger.debug("Card number: {}", request.cardNumber());  // 절대 금지
}
```

---

## Additional Security Best Practices

### Input Validation

Always validate and sanitize user input:

```java
public User createUser(@NonNull String email, @NonNull String name) {
    // 이메일 형식 검증
    if (!email.matches("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+$")) {
        throw new IllegalArgumentException("Invalid email format");
    }
    
    // 길이 제한
    if (name.length() > 100) {
        throw new IllegalArgumentException("Name too long");
    }
    
    // HTML 이스케이프 (XSS 방지)
    String safeName = HtmlUtils.htmlEscape(name);
    
    return new User(email, safeName);
}
```

---

### File Upload Security

```java
public void uploadFile(@NonNull MultipartFile file) throws IOException {
    // 파일 크기 제한
    if (file.getSize() > 10_000_000) {  // 10MB
        throw new IllegalArgumentException("File too large");
    }
    
    // 확장자 화이트리스트
    String ext = FilenameUtils.getExtension(file.getOriginalFilename());
    if (!List.of("jpg", "png", "pdf").contains(ext.toLowerCase())) {
        throw new IllegalArgumentException("Invalid file type");
    }
    
    // 안전한 파일명 생성 (디렉토리 트래버설 방지)
    String safeFilename = UUID.randomUUID().toString() + "." + ext;
    Path uploadPath = Paths.get("/uploads").resolve(safeFilename);
    
    // 경로 정규화 후 검증
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
// 안전한 대안: JSON (Jackson, Gson)
ObjectMapper mapper = new ObjectMapper();
User user = mapper.readValue(json, User.class);

// Java 직렬화는 피하되, 불가피한 경우:
ObjectInputStream ois = new ObjectInputStream(inputStream) {
    @Override
    protected Class<?> resolveClass(ObjectStreamClass desc) throws IOException, ClassNotFoundException {
        // 화이트리스트 검증
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
