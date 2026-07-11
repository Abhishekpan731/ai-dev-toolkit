# Generate Test Suite Skill

## Table-Driven Pattern (Go)

```go
func TestCreateUser_EdgeCases(t *testing.T) {
    tests := []struct {
        name    string
        email   string
        pass    string
        wantErr bool
    }{
        {"valid", "user@example.com", "password123", false},
        {"empty email", "", "password123", true},
        {"invalid email", "notanemail", "password123", true},
        {"short password", "user@example.com", "short", true},
        {"empty password", "user@example.com", "", true},
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            user, err := CreateUser(tt.email, tt.pass)
            if (err != nil) != tt.wantErr {
                t.Errorf("CreateUser() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if !tt.wantErr && user == nil {
                t.Error("CreateUser() returned nil user")
            }
        })
    }
}
```

## Jest Pattern (TypeScript)

```typescript
describe('createUser', () => {
    it('should create user with valid data', async () => {
        const user = await createUser('test@example.com', 'password123');
        expect(user).toBeDefined();
        expect(user.email).toBe('test@example.com');
    });
    
    it('should reject invalid email', async () => {
        await expect(createUser('invalid', 'password123'))
            .rejects.toThrow('Invalid email');
    });
    
    it('should reject short password', async () => {
        await expect(createUser('test@example.com', 'short'))
            .rejects.toThrow('Password too short');
    });
});
```

## Pytest Pattern (Python)

```python
import pytest

def test_create_user_valid():
    user = create_user('test@example.com', 'password123')
    assert user is not None
    assert user.email == 'test@example.com'

def test_create_user_invalid_email():
    with pytest.raises(ValueError, match='Invalid email'):
        create_user('notanemail', 'password123')

def test_create_user_short_password():
    with pytest.raises(ValueError, match='Password too short'):
        create_user('test@example.com', 'short')
```

**Coverage Target**: 80%+ mandatory
**Edge Cases**: Always test empty, nil, invalid inputs
