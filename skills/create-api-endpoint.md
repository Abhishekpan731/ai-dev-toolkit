# Create API Endpoint Skill

## Pattern
1. Create handler (<50 lines, @author <AUTHOR_NAME>)
2. Add route
3. Define types
4. Add validation
5. Write tests (80%+)
6. Update docs

## Go Template
```go
// @author <AUTHOR_NAME>
func HandleCreateUser(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodPost {
        http.Error(w, "Method not allowed", 405)
        return
    }
    
    var req CreateUserRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "Invalid request", 400)
        return
    }
    
    if err := validateRequest(req); err != nil {
        http.Error(w, err.Error(), 400)
        return
    }
    
    user, err := createUser(req)
    if err != nil {
        http.Error(w, err.Error(), 500)
        return
    }
    
    json.NewEncoder(w).Encode(user)
}
```

## TypeScript Template
```typescript
/**
 * @author <AUTHOR_NAME>
 */
export const createUser: RequestHandler = async (req, res) => {
    const { email, password } = req.body;
    
    if (!email || !password) {
        return res.status(400).json({ error: 'Missing required fields' });
    }
    
    try {
        const user = await User.create({ email, password });
        res.status(201).json(user);
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
};
```

## Python Template
```python
def create_user(email: str, password: str) -> User:
    """
    @author <AUTHOR_NAME>
    """
    if not email or '@' not in email:
        raise ValueError("Invalid email")
    if len(password) < 8:
        raise ValueError("Password too short")
    
    user = User(email=email)
    user.set_password(password)
    user.save()
    return user
```
