
## General

- **Allow Lombok**, but **restrict it to a small, approved set** of annotations, and **ban the risky ones**.

## Allowed

### Logging

- `@Slf4j` (preferred) Use it instead of manually declaring a logger.

### Boilerplate reduction (safe)

- `@Getter` (prefer on fields or class for read-only models)    
- `@RequiredArgsConstructor` (great for DI + `final` fields)
- `@NoArgsConstructor` / `@AllArgsConstructor` **only when required by frameworks**, and document why
- `@Builder` (good for complex constructors / test fixtures)
- `@StandardException` (good for exception)

### Use with caution

- `@ToString`
	- Use cautiously. It can accidentally log secrets or huge object graphs.
	- If object has secrets or has expected huge object graphs, then
		- `@ToString(onlyExplicitlyIncluded = true)`
		- exclude sensitive fields explicitly
- `@EqualsAndHashCode`
	- OK for Value Object
    - `cacheStrategy=lazy` is recommended
	- but when using this for Aggregate Root or Domain objects then ID matches requires
		- `@EqualsAndHashCode(onlyExplicitlyIncluded = true)`

## Ban (Strongly recommended)

The following annotations are explicitly banned:

| Annotation | Reason |
|------------|--------|
| `@Setter` | 불변성 위반, 캡슐화 약화 |
| `@Data` | `@Setter` 포함, equals/hashCode 자동 생성 위험 |
| `@Value` | Record와 혼동, 암묵적 동작 과다 |
| `@SneakyThrows` | Checked exception 우회, 예외 처리 의도 불명확 |
| `@Cleanup` | try-with-resources 사용 권장 |
| `@val` / `@var` | Java 10+ `var` 키워드 사용 |
| `@Synchronized` | 명시적 동기화 코드 작성 권장 |
| `@UtilityClass` | `private` 생성자와 `final class` 명시 권장 |
| `@With` | 불변 객체 복사는 명시적 빌더 사용 |
| `@Accessors` | 비표준 getter/setter 패턴 |

**Rule**: Ban any annotation not explicitly listed in the Allowed section.
