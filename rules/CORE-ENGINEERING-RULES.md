# Core Engineering Rules

> Unified quality enforcement | 8 files consolidated | @author <AUTHOR_NAME>
> Last updated: July 2026

**This file consolidates**:

1. codingStandard.md (17k) - Quality enforcement
2. documentation.md (28k) - File organization
3. code-quality.md (7k) - Code standards
4. git-workflow.md (4.3k) - Version control
5. security-standards.md (6.5k) - Security rules
6. testing-conventions.md (6.2k) - Testing standards
7. verification.md (4.9k) - Evidence-based verification
8. immutability.md (3.7k) - Mutation prevention

---

# 1. CODING STANDARDS

## 1.1 Quality Gates (MANDATORY)

### 50-Line Function Limit

- Every function ≤50 lines (including braces)
- Extract helpers if exceeding
- Single Responsibility Principle
- Refactor immediately if violated

### @author <AUTHOR_NAME> Tag

- Required on ALL new functions/methods/classes
- Godoc (Go), Javadoc (Java), JSDoc (TypeScript)
- Tag new code only, not entire files
- Format: @author <AUTHOR_NAME>

### 80%+ Test Coverage

- Measured: JaCoCo (Java), go test -cover (Go), Jest (TS)
- Applies to NEW code
- 100% for critical paths (auth, payment, data)
- Zero untested code in production

### Zero-Linter-Failure

- ALL linters pass with zero warnings
- Go: gofmt, goimports, go vet, golangci-lint
- Java: checkstyle, spotbugs, pmd
- TypeScript: eslint, prettier
- Python: black, flake8, mypy

## 1.2 Defensive Programming

### Early Returns (Guard Clauses)

Validate inputs at function start, return errors immediately.

Example Go:

```
func CreateVolume(req *Request) (*Response, error) {
    if req == nil {
        return nil, fmt.Errorf("request is nil")
    }
    if req.Name == "" {
        return nil, status.Error(codes.InvalidArgument, "volume name required")
    }
    return processVolume(req), nil
}
```

### Error Wrapping

ALWAYS wrap errors with context: fmt.Errorf("context: %w", err)

### Credential Security

- ZERO hardcoded credentials
- Environment variables: os.Getenv("DATABASE_URL")
- Secret managers: K8s Secrets, AWS, Vault
- Fail-safe: App MUST NOT start if credentials missing

## 1.3 Code Clarity

### Naming

- volumeID not v or id
- userAuthToken not tok
- isValid, hasPermission, canDelete (booleans)

### Function Naming

- Actions: CreateVolume, DeleteUser (verbs)
- Getters: GetVolume, UserProfile (nouns)
- Booleans: IsValid, HasPermission (questions)

### Comments

- Public APIs: MUST have docs
- Complex logic: Explain WHY, not WHAT
- TODOs: Include ticket number
- No commented-out code

## 1.4 Surgical Edits (MANDATORY)

### Targeted Modifications

- Change ONLY required lines
- NO wholesale rewrites
- NO refactoring outside scope

### Minimal Churn

- Incremental updates over massive refactors
- Minimize diff size
- Use str-replace-editor with precise ranges

## 1.5 Pre-Commit Checklist

- [ ] Functions ≤50 lines
- [ ] @author tags present
- [ ] Tests written (80%+)
- [ ] Linters passing
- [ ] No hardcoded credentials
- [ ] Descriptive names
- [ ] Early returns
- [ ] Errors wrapped
- [ ] Code formatted
- [ ] Docs updated
- [ ] Surgical edits only
- [ ] Minimal churn

---

# 2. DOCUMENTATION SYSTEM

## 2.1 Core Principles

### Update-Only Principle

1. Search for existing docs
2. Update closest match
3. Only create if truly new

### 3-File Maximum Rule (MANDATORY)

- Max 3 files per directory
- Applies to ALL file types
- No directory may exceed 3 items
- Granular storage required

### Repository Cleanliness (MANDATORY)

NO documentation in source code directories

Prohibited:

- /storage-interface-driver/\*.md (except README)
- /pkg/**/\*.md, /cmd/**/_.md, /internal/\*\*/_.md

Required:

- /archive/analysis-reports/analysis-ES-{number}/
- /docs/architecture/
- /docs/deployment-{version}/

## 2.2 Auto-Discovery Engine

Before EVERY task:

1. Auto-scan .ai-config/\*\*
2. Classify files by type
3. Load context dynamically
4. Apply rules
5. Execute with awareness
6. Validate standards

## 2.3 Pre-Creation Checklist

- [ ] Check repository cleanliness
- [ ] Verify archive location
- [ ] Check existing docs
- [ ] Search for ES-{number} directory
- [ ] Verify <3 files
- [ ] Plan subdivision
- [ ] Descriptive path
- [ ] Attribution added

---

# 3. CODE QUALITY

## 3.1 Function Standards

- Max 30-50 lines
- Max 4 parameters
- Max 3 levels nesting
- Cyclomatic complexity <10

## 3.2 Completeness (MANDATORY)

Done Means Done:

- No TODOs in committed code
- All edge cases handled
- All tests passing
- All docs updated

Evidence Required:

- Fresh terminal output
- Coverage reports
- Lint output (zero warnings)
- Build success logs

## 3.3 DRY Principle

- Extract common logic (3+ occurrences)
- Use constants (no magic numbers)
- Don't over-DRY (sometimes duplication better)

## 3.4 SOLID Principles

- S: Single Responsibility
- O: Open/Closed
- L: Liskov Substitution
- I: Interface Segregation
- D: Dependency Inversion

---

# 4. GIT WORKFLOW

## 4.1 Commit Format

type(scope): message

Types: feat, fix, docs, style, refactor, test, chore, perf

Examples:

- feat(auth): add password reset
- fix(api): handle null response
- docs(readme): update install instructions

## 4.2 Branch Naming

- feature/short-description
- bugfix/issue-description
- hotfix/critical-issue

## 4.3 Rules (MANDATORY)

- Never work on main directly
- Keep branches short-lived
- Delete after merge
- Sync with main frequently

## 4.4 Dangerous Operations (ASK FIRST)

- Never force push to main/shared
- Never rebase shared branches
- Never commit to main directly
- Ask before: committing, pushing, PRs

## 4.5 What NOT to Commit

- Secrets, API keys, passwords
- .env files (except .env.example)
- Large binaries
- Generated files
- IDE files
- node_modules/, venv/

---

# 5. SECURITY STANDARDS

## 5.1 Secrets Management

Never Hardcode:

- Use environment variables
- Use secret managers (K8s, AWS, Vault)
- Fail if credentials missing

## 5.2 Input Validation

- Always validate user inputs
- Whitelist over blacklist
- Type checking
- Length limits
- Format validation

## 5.3 SQL Injection Prevention

Use parameterized queries:

- Python: cursor.execute(query, (param,))
- Never concatenate SQL strings

## 5.4 Authentication & Authorization

- Verify authentication
- Verify authorization
- Secure sessions (HTTPOnly, Secure, SameSite)
- Rate limiting
- MFA for sensitive ops
- Hash passwords: bcrypt, argon2 (NOT MD5)
- Invalidate sessions on logout
- Timeout inactive sessions

## 5.5 XSS Prevention

- Escape output (HTML-encode)
- Content Security Policy
- Sanitize HTML (DOMPurify)
- HTTPOnly cookies
- Avoid innerHTML

## 5.6 CSRF Prevention

- CSRF tokens in forms
- SameSite cookies
- Verify Origin/Referer
- Re-auth for sensitive ops

## 5.7 Logging

What to Log:

- Auth events
- Authorization violations
- Input validation failures
- Config changes
- Admin actions

What NOT to Log:

- Passwords
- Tokens
- API keys
- Credit cards
- SSNs
- PII/PHI

## 5.8 Security Headers

- Content-Security-Policy: default-src 'self'
- X-Frame-Options: DENY
- X-Content-Type-Options: nosniff
- Strict-Transport-Security: max-age=31536000

---

# 6. TESTING CONVENTIONS

## 6.1 Coverage Targets

- New code: 80%+
- Critical paths: 95%+
- Edge cases: All error conditions

## 6.2 Unit Tests

Arrange-Act-Assert pattern:

```
func TestCreateVolume_Success(t *testing.T) {
    // Arrange
    driver := NewTestDriver()
    req := &csi.CreateVolumeRequest{...}

    // Act
    resp, err := driver.CreateVolume(context.Background(), req)

    // Assert
    assert.NoError(t, err)
    assert.NotNil(t, resp)
}
```

## 6.3 Table-Driven Tests

```
tests := []struct {
    name    string
    input   Config
    wantErr bool
}{
    {"valid", validConfig, false},
    {"empty name", Config{Name: ""}, true},
}
```

## 6.4 Test Naming

- Describe behavior, not implementation
- should_return_error_when_user_not_found
- validates_user_creation_with_missing_email

## 6.5 Mocking

- Mock external dependencies
- Keep tests fast (<1s)
- Keep tests reliable (no network/DB)

---

# 7. VERIFICATION FRAMEWORK

## 7.1 Evidence-Based Completion

No Completion Claims Without Fresh Evidence

Before claiming done:

- [ ] Fresh terminal output (tests passing)
- [ ] Coverage report (80%+)
- [ ] Lint output (zero warnings)
- [ ] Build logs (success)
- [ ] Manual test results

## 7.2 Cross-Platform Verification

Test on ALL platforms:

- Linux (Ubuntu, RHEL)
- macOS
- Windows (if applicable)
- Target Kubernetes versions

## 7.3 Error Classification

- P0 Critical: Data loss, security breach
- P1 High: Feature broken
- P2 Medium: Minor bug
- P3 Low: Cosmetic

## 7.4 Confidence Scoring

- High (90%+): Tests pass, manual verification
- Medium (70-89%): Tests pass, limited manual
- Low (<70%): Tests pass, edge cases uncertain

---

# 8. IMMUTABILITY STANDARDS

## 8.1 Core Principle

Immutable by default, mutable by exception.

## 8.2 Banned Mutations

Arrays:

- BAD: array.push(item), array.pop(), array.shift()
- GOOD: [...array, item], array.slice(0, -1)

Objects:

- BAD: object.property = value
- GOOD: {...object, property: value}

Compound:

- BAD: count++, count += 5
- GOOD: count = count + 1, count = count + 5

Parameters:

- BAD: Mutating function parameters
- GOOD: Return new object

## 8.3 Framework Exceptions

These are ALLOWED (frameworks handle internally):

- Immer: produce() mutations
- Redux Toolkit: createSlice
- MobX: Observables
- Pinia: Store mutations
- Zustand: set() mutations

Mutations MUST stay within framework boundaries.

## 8.4 Benefits

- Predictability
- Time-travel debugging
- No test pollution
- No race conditions
- Framework optimizations work

---

# 9. LANGUAGE-SPECIFIC STANDARDS

## 9.1 Go

```
// @author <AUTHOR_NAME>
func (d *Driver) CreateVolume(ctx context.Context, req *csi.CreateVolumeRequest) (*csi.CreateVolumeResponse, error) {
    if err := d.validateRequest(req); err != nil {
        return nil, fmt.Errorf("invalid request: %w", err)
    }
    // Implementation <50 lines
}
```

Checks:

- gofmt -w .
- goimports -w .
- go vet ./...
- golangci-lint run
- go test -cover ./... (80%+)

## 9.2 Java

```
/**
 * @author <AUTHOR_NAME>
 */
@Service
public class UserService {
    public UserResponse createUser(CreateUserRequest request) {
        validateRequest(request);
        // <50 lines
    }
}
```

Checks:

- mvn clean verify
- mvn fmt:format
- mvn checkstyle:check

## 9.3 TypeScript/React

```
/**
 * @author <AUTHOR_NAME>
 */
export const UserProfile: React.FC<Props> = ({ userId }) => {
  const [user, setUser] = useState<User | null>(null);
  // <50 lines
};
```

Patterns:

- Functional components + hooks
- TypeScript strict mode
- Tailwind CSS
- React Query
- WCAG AAA accessibility

## 9.4 Python

```
def create_user(email: str, password: str) -> User:
    """
    @author <AUTHOR_NAME>
    """
    if not email or '@' not in email:
        raise ValueError("Invalid email")
    # <50 lines
```

Checks:

- black .
- flake8 .
- mypy .
- pytest --cov=.

---

# 10. SUMMARY

## Quality Gates Checklist

- [ ] Functions ≤50 lines
- [ ] @author tags present
- [ ] 80%+ test coverage
- [ ] Zero linter warnings
- [ ] No hardcoded credentials
- [ ] Descriptive names
- [ ] Early returns
- [ ] Error wrapping
- [ ] Immutable patterns
- [ ] Evidence-based verification
- [ ] Surgical edits
- [ ] Docs updated

---

**Author**: <AUTHOR_NAME>
**Status**: PRODUCTION-READY
**Version**: 1.0 (Consolidated July 2026)

_Quality is not an act, it is a habit._
