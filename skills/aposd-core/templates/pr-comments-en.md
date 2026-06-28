# APoSD PR Comments Template — English

This file provides English comment templates for PR reviews using APoSD red flags.

Use the appropriate template based on the severity and type of issue detected.

---

## Shallow Module

**When to use**: Interface is complex relative to the implementation; callers need to know too much about internals.

### Blocking Version (fix in this PR)

```
## Shallow Module — `file.py:line_range`

This repository class exposes SQL directly, making the interface complex relative to what it provides.

**Situation**:
- Multiple methods, each returning a single SQL query result
- Callers are directly aware of SQL and table structure

**Impact**:
- Changes to the User table schema ripple to every query method
- Tests are tightly coupled to the database

**Suggestion**:
Deepen this module by hiding SQL and adding business logic.

Example:
```python
# Before
def get_active_users(self):
    return db.query("SELECT * FROM users WHERE status = 'active'")
def get_by_email(self, email):
    return db.query("SELECT * FROM users WHERE email = ?", email)

# After
def find_active(self):
    # SQL, caching, validation hidden inside
    return self._query_users(status='active')
def find_by_email(self, email):
    # Interface expresses intent, not SQL
    return self._query_users(email=email)
```

This also improves testability (easier to mock).
```

### Non-Blocking Version (address later)

```
## Shallow Module — Consider for follow-up `file.py:line_range`

This service layer just forwards calls to the repository.

**Pattern**:
```python
def get_user(self, id):
    return self.repository.get_user(id)  # Just forwarding
```

**To consider in a follow-up**:
- Does this service layer add real value, or could callers use the repository directly?
- If it should exist, give it responsibilities (caching, logging, validation)

Not required for this PR, but worth tracking for a future refactor.
```

---

## Information Leakage

**When to use**: Callers know too much about internal structure (types, database schema, storage format).

### Blocking Version

```
## Information Leakage — `file.py:line_range`

Callers are directly aware of internal User status enum.

**Situation**:
```python
from models import User, UserStatus  # Internal types exposed
user = repo.get(id)
if user.status == UserStatus.PENDING:  # Depends on internal representation
    ...
```

**Impact**:
- Changes to User status representation require updates in calling code
- Database schema changes ripple through multiple files

**Suggestion**:
Add a helper method to the repository to hide the enum.

```python
# After
user = repo.find_verified(id)
if user:  # Repository encapsulates the status check
    # Caller never sees the status enum
```

This way, internal representation changes don't affect calling code.
```

### Non-Blocking Version

```
## Information Leakage — Consider for follow-up `file.py:line_range`

API response mirrors the database schema exactly.

**Pattern**:
```python
{
  "user_id": 123,
  "user_name": "Alice",
  "created_at": "2024-01-01T00:00:00Z"
}
```

This response format is tightly coupled to the database table structure.

**To consider in a follow-up**:
- Decouple API contract from database schema
- Return only necessary fields; design the API independently

Not required for this PR, but worth considering in future API design work.
```

---

## Change Amplification

**When to use**: A single logical change requires modifications in many places.

### Blocking Version

```
## Change Amplification — `file_a.py`, `file_b.py`, `file_c.py`

Adding a single user email field cascades through 5 files.

**Situation**:
- `models.py`: Add email field to User
- `repository.py`: Update SQL query
- `service.py`: Add validation logic
- `api.py`: Include email in response
- `test.py`: Update multiple tests

**Impact**:
Adding one field to User requires changes across all layers. This indicates weak module boundaries.

**Suggestion**:
Redesign so the repository completely hides the database schema, and the API layer is independent of the internal User type.

```python
# After
# Repository returns internal User type, but API is independent
class UserResponse(BaseModel):
    id: int
    email: str
    # Schema changes only here

# Controller layer
user = repository.find(id)
return UserResponse.from_user(user)  # Mapping happens here
```

With proper layer separation, database changes stay in the repository.
```

### Non-Blocking Version

```
## Change Amplification — Consider for follow-up `multiple files`

User status checks are duplicated in middleware and controller.

**Pattern**:
Same logic in multiple places means changes must happen in multiple places.

**To consider in a follow-up**:
- Consolidate shared logic in one place (repository or service)
- Reduce duplication

Not required for this PR. Address in a future refactor PR.
```

---

## Vague Naming

**When to use**: Function or class names don't clearly express what they do.

### Blocking Version

```
## Vague Naming — `file.py:line_range`

Function name `process_data()` doesn't clarify what it does.

**Current**:
```python
def process_data(items):
    return [x * 2 for x in items if x > 10]
```

You must read the code to understand the intent.

**Suggestion**:
Name the function after what it actually does.

```python
def filter_and_double_large_items(items):
    return [x * 2 for x in items if x > 10]

# Or
def double_items_over_threshold(items, threshold=10):
    return [x * 2 for x in items if x > threshold]
```

Now the intent is clear from the name alone.
```

### Non-Blocking Version

```
## Vague Naming — Consider for follow-up `file.py:line_range`

Method `convert()` is ambiguous. Convert to what?

**Suggestion**:
```python
# Better
def to_api_response(user):
    # Internal User type → API response format
    ...
```

With a clearer name, the intent becomes obvious.

Not required now, but consider renaming at the next opportunity.
```

---

## Special-Case Mixture

**When to use**: General logic mixed with special cases for specific callers or contexts.

### Blocking Version

```
## Special-Case Mixture — `file.py:line_range`

Function `save_user()` branches based on caller context.

**Situation**:
```python
def save_user(user, is_admin=False, skip_validation=False):
    if not skip_validation:
        validate(user)
    if is_admin:
        # Admin-only logic
        ...
    else:
        # Regular user logic
        ...
```

Callers pass flags to control behavior. New callers tend to add new special cases.

**Impact**:
- Test cases explode (flag combinations)
- Logic becomes harder to reason about
- New special cases accumulate

**Suggestion**:
Provide separate, explicit interfaces for each use case.

```python
def save_user(user):
    # Standard path, always validates
    validate(user)
    ...

def save_user_admin_override(user):
    # Admin-specific, clear intent
    # Logic unique to admins stays here
    ...
```

Callers use the interface that matches their intent.
```

### Non-Blocking Version

```
## Special-Case Mixture — Consider for follow-up `file.py:line_range`

Configuration branches for different scenarios.

**Pattern**:
```python
if config.enable_caching:
    # Caching behavior
    ...
elif config.enable_fallback:
    # Fallback behavior
    ...
```

**To consider in a follow-up**:
- Separate implementations into distinct classes
- Use a factory to instantiate based on config

Not required for this PR. Consider in a future refactor.
```

---

## Excessive Configuration

**When to use**: Too many parameters, flags, or options; callers make too many decisions.

### Blocking Version

```
## Excessive Configuration — `file.py:line_range`

Cache initialization requires 6 parameters. Callers make too many decisions.

**Situation**:
```python
cache = Cache(
    ttl=300,
    eviction_policy='lru',
    distributed=True,
    compression=True,
    encryption=True,
    metrics_enabled=True
)
```

**Suggestion**:
Make decisions inside the module, or split into two simpler classes.

```python
# After: simple, decisive
cache = LocalCache(ttl=300)

# Or, explicit but clearly different
cache = DistributedSecureCache(ttl=300)  # Encryption, compression enabled by default
```

Callers have fewer decisions to make.
```

### Non-Blocking Version

```
## Excessive Configuration — Consider for follow-up `file.py:line_range`

Adding a new boolean flag to control behavior.

**Suggestion**:
Before adding more configuration:
- Can you make a hard decision instead?
- Are there invalid combinations that should be prevented?

Not required now, but consider consolidating configuration in future work.
```

---

## Unnecessary Error-Path Complexity

**When to use**: Error handling is scattered, overly complicated, or errors are silently swallowed.

### Blocking Version

```
## Unnecessary Error-Path Complexity — `file.py:line_range`

Callers must handle multiple exception types for one operation.

**Situation**:
```python
try:
    user = repository.get_user(id)
except UserNotFoundError:
    user = None
except DatabaseError:
    raise
```

Callers are aware of `UserNotFoundError`, which is an implementation detail.

**Suggestion**:
Define errors out of existence. Return `None` or use `Optional` instead.

```python
# After
user = repository.find_user(id)  # Returns None if not found
if not user:
    # Caller decides what to do
    ...
```

Callers never see exception types for missing data.
```

### Non-Blocking Version

```
## Unnecessary Error-Path Complexity — Consider for follow-up `file.py:line_range`

Error handling is scattered across multiple layers.

**Pattern**:
- Service layer catches exceptions
- Controller layer catches exceptions too
- Same error handled in multiple places

**To consider in a follow-up**:
- Consolidate error handling in one place
- Reduce exception types; use richer error messages instead

Not required for this PR, but prioritize in future refactoring.
```

---

## Generic Templates

### Question Format (gentle inquiry)

```
## [Red Flag Name] — Question

I have a question about the design here.

**Situation**:
[Code example]

**Questions**:
- Would callers be able to do the same thing without this layer?
- What's the motivation for hiding this information from callers?

If there's a design reason, please explain. Understanding the intent helps me review better.
```

### Multi-Option Format (suggest alternatives)

```
## [Red Flag Name] — Possible Approaches

I see a few ways to improve this. Which sounds best?

**Option A**: [Description]
- Pros: [Benefits]
- Cons: [Downsides]

**Option B**: [Description]
- Pros: [Benefits]
- Cons: [Downsides]

**Option C**: [Description]
- Pros: [Benefits]
- Cons: [Downsides]

Happy to discuss. Let me know which direction you prefer.
```

### Praise and Improvement

```
## [Red Flag Name] — Good thinking, here's how to take it further

I like the information-hiding mindset in this code.

**To strengthen it**:
[Specific improvement]

Approving as-is. Consider this for a follow-up PR.
```

---

## Blocking vs Non-Blocking

### Blocking (must fix in this PR)
- Introduces new red flags not in the original code
- High risk with low effort (easy fix, big impact)
- Violates safety rules (broad rewrite, new layers, deferred decisions)
- Existing tests would fail or be misleading

### Non-Blocking (address later)
- Pre-existing red flags not made worse
- Would require broad changes (>1 PR)
- Requires separate architectural decisions
- Low friction; work can happen independently

### Follow-Up (optional but recommended)
- Provide a clear starting point for improvement
- Don't make it urgent; let the team prioritize
- Link to a refactor plan if one exists

---

## Style Guidelines

1. **Be specific**: "This causes a 5-file change when User schema updates" vs "This is bad design"
2. **Be actionable**: "Move the status check into the repository" vs "Refactor this"
3. **Assume good intent**: "This pattern is common, but here's a better way" vs "This is wrong"
4. **Respect constraints**: "Not blocking this PR, but worth a follow-up" vs "This must change now"
5. **Show, don't tell**: Include before/after code examples
6. **Don't overwhelm**: Limit to 3-4 issues per PR
7. **Focus on maintainability**: Design matters, not style
