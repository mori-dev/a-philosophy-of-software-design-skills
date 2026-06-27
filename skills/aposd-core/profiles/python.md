# APoSD Python Profile

Python-specific red flags and patterns to watch for during code review.

## Common Shallow Module Patterns

### Pass-Through Repository or Service

**Pattern**:
```python
class UserRepository:
    def __init__(self, db):
        self.db = db
    
    def get_by_id(self, id):
        return self.db.query(User).filter_by(id=id).first()
    
    def get_by_email(self, email):
        return self.db.query(User).filter_by(email=email).first()

class UserService:
    def __init__(self, repo):
        self.repo = repo
    
    def get_user(self, id):
        return self.repo.get_by_id(id)  # Just forwarding
```

**Problem**: 
- Repository exposes each query method separately
- Service just wraps repository
- Multiple layers with no real separation of concerns

**Deepen It**:
```python
class UserRepository:
    def __init__(self, db):
        self.db = db
        self._cache = {}
    
    def find(self, **filters):
        # Repository hides: SQL, caching, ORM details, validation
        user = self._cache.get(filters['id'])
        if user:
            return user
        
        user = self._query_users(**filters)
        if user:
            self._cache[filters['id']] = user
        return user
    
    def _query_users(self, **filters):
        # All SQL stays internal
        query = self.db.query(User)
        for key, value in filters.items():
            query = query.filter_by(**{key: value})
        return query.first()
```

---

## Information Leakage Patterns

### Dataclass/Pydantic Leaking Storage Details

**Pattern**:
```python
from dataclasses import dataclass
from datetime import datetime

@dataclass
class User:
    id: int
    name: str
    email: str
    created_at: datetime  # Database column
    updated_at: datetime  # Database column
    password_hash: str    # Storage detail!
    
# Caller:
user = repo.get_user(123)
if user.password_hash:  # Caller knows internal storage
    ...
```

**Problem**:
- Caller can access `password_hash` directly
- Changes to storage format ripple to callers
- Domain model mixed with storage details

**Fix**:
```python
@dataclass
class User:
    """Domain model, not storage detail"""
    id: int
    name: str
    email: str
    
    def is_verified(self) -> bool:
        # Logic hidden inside; caller doesn't care about password_hash
        ...

class UserRepository:
    """Storage layer hides passwords, timestamps, everything"""
    def find_verified(self, email: str) -> Optional[User]:
        # Internal query including password check
        db_user = self.db.query(UserRecord).filter_by(email=email).first()
        if db_user and self._check_password(db_user.password_hash):
            return User(id=db_user.id, name=db_user.name, email=db_user.email)
        return None
```

---

## Hidden Mutable State

**Pattern**:
```python
class OrderService:
    def __init__(self):
        self.orders = []  # Shared state!
    
    def add_order(self, order):
        self.orders.append(order)
    
    def get_orders(self):
        return self.orders  # Returns mutable reference!

# Caller:
orders = service.get_orders()
orders[0].status = "cancelled"  # Modifies service state silently!
```

**Problem**:
- Caller can modify service state directly
- No control over invariants
- Silent side effects

**Fix**:
```python
class OrderService:
    def __init__(self, db):
        self.db = db
    
    def get_orders(self, user_id: int) -> List[OrderInfo]:
        # Return read-only representation
        records = self.db.query(Order).filter_by(user_id=user_id).all()
        return [OrderInfo.from_record(r) for r in records]
    
    def update_order_status(self, order_id: int, status: str) -> None:
        # Caller can't modify state directly; must go through methods
        if status not in VALID_STATUSES:
            raise ValueError(f"Invalid status: {status}")
        self.db.execute(
            update(Order).where(Order.id == order_id).values(status=status)
        )
```

---

## Exception Path Complexity

### Catch Blocks Scattered Across Layers

**Pattern**:
```python
# repository.py
def get_user(self, id):
    try:
        return self.db.query(User).filter_by(id=id).first()
    except SQLAlchemyError:
        raise UserNotFoundError()

# service.py
def verify_user(self, id):
    try:
        user = self.repo.get_user(id)
    except UserNotFoundError:
        return False
    # More logic...

# controller.py
def get_profile(self, id):
    try:
        user = self.service.verify_user(id)
    except Exception:
        return None
```

**Problem**:
- Error handling scattered across layers
- Same error caught and re-raised multiple times
- Caller must understand all exception types
- Context lost as exception bubbles up

**Fix**:
```python
# repository.py — define errors out of existence
def find_user(self, id: int) -> Optional[UserRecord]:
    # No exceptions; returns None if not found
    return self.db.query(User).filter_by(id=id).first()

# service.py — single point of handling
def verify_user(self, id: int) -> Optional[User]:
    record = self.repo.find_user(id)
    if not record:
        logger.info(f"User not found: {id}")
        return None
    # Verify logic here
    ...
    return User.from_record(record)

# controller.py — no try-catch needed
def get_profile(self, id: int):
    user = self.service.verify_user(id)
    if user:
        return {"id": user.id, "name": user.name}
    return {"error": "User not found"}
```

---

## Special-Case Spread

### Boolean Parameters Controlling Behavior

**Pattern**:
```python
def process_payment(
    amount: float,
    user_id: int,
    is_subscription: bool = False,
    is_admin: bool = False,
    skip_fraud_check: bool = False,
    use_backup_processor: bool = False,
) -> bool:
    if skip_fraud_check:
        # Skip fraud detection
        ...
    elif is_admin:
        # Admins bypass certain checks
        ...
    elif is_subscription:
        # Subscriptions use different processor
        ...
    else:
        # Regular payment flow
        ...
    
    if use_backup_processor:
        # Use backup processor
        return backup_processor.charge(...)
    return primary_processor.charge(...)
```

**Problem**:
- Callers choose the behavior
- 2^5 = 32 possible combinations, many invalid
- Logic scattered across conditions
- New features add more flags

**Fix**:
```python
class PaymentProcessor:
    def process_regular_payment(self, amount: float, user_id: int) -> bool:
        """Standard payment flow with fraud check and primary processor"""
        self._check_fraud(user_id)
        return self.primary_processor.charge(amount)

class SubscriptionPaymentProcessor:
    def process_subscription(self, amount: float, user_id: int) -> bool:
        """Subscriptions bypass fraud check, use subscription processor"""
        return self.subscription_processor.charge(amount)

class AdminPaymentProcessor:
    def process_admin_override(self, amount: float, user_id: int) -> bool:
        """Admins bypass all checks, use primary processor"""
        return self.primary_processor.charge(amount)

# Caller chooses the right processor, not flags
processor = AdminPaymentProcessor() if user.is_admin else PaymentProcessor()
processor.process_regular_payment(100, user_id)
```

---

## Vague Naming

### Common Offenders

| Avoid | Use Instead |
|-------|------------|
| `process_data` | `filter_active_users`, `convert_csv_to_json`, `aggregate_sales_by_region` |
| `utils` | `date_formatting`, `string_validation`, `retry_logic` |
| `handler` | `validate_user_input`, `log_error`, `send_notification` |
| `manager` | `order_repository`, `user_service`, `payment_processor` |
| `data` | `user`, `order`, `transaction_record` |
| `convert` | `user_to_api_response`, `json_to_model`, `model_to_dto` |
| `check` | `is_verified`, `has_permission`, `can_process` |

**Python-specific**: Comprehensions hide intent; don't rely on them alone:

```python
# Bad: what does this do?
result = [x * 2 for x in items if x > 10]

# Good: name expresses intent
doubled_large_items = [x * 2 for x in items if x > 10]

# Or better: use a function name
def double_items_over_threshold(items, threshold=10):
    return [x * 2 for x in items if x > threshold]
```

---

## Configuration Explosion

### Too Many Config Classes

**Pattern**:
```python
@dataclass
class DatabaseConfig:
    host: str
    port: int
    username: str
    password: str
    pool_size: int
    echo: bool

@dataclass
class CacheConfig:
    backend: str
    ttl: int
    max_size: int
    eviction_policy: str

@dataclass
class LoggingConfig:
    level: str
    format: str
    file: str
    rotate: bool

@dataclass
class AppConfig:
    database: DatabaseConfig
    cache: CacheConfig
    logging: LoggingConfig
    # Plus 10 more configs...

app = App(
    AppConfig(
        database=DatabaseConfig(...),
        cache=CacheConfig(...),
        logging=LoggingConfig(...)
    )
)
```

**Problem**:
- Callers must construct deeply nested configs
- Invalid combinations not prevented
- Too many decisions deferred

**Fix**:
```python
# Make hard decisions; expose only what must vary
class AppFactory:
    @staticmethod
    def create_production():
        return App(
            db_url="postgresql://prod.db",
            cache_type="redis",
            log_level="INFO"
        )
    
    @staticmethod
    def create_development():
        return App(
            db_url="sqlite:///:memory:",
            cache_type="memory",
            log_level="DEBUG"
        )

# Caller: one decision
app = AppFactory.create_production()
```

---

## Module Cohesion Check

**For Python modules, ask**:

1. **Can I describe this module's responsibility in one sentence without "and"?**
   - ✓ "This module queries users from the database and caches results"
   - ✗ "This module handles users, orders, and payments" (three responsibilities)

2. **If I moved a function to another module, would the module still be coherent?**
   - If yes, it probably belongs elsewhere
   - If no, it's part of the core responsibility

3. **Do callers use all the functions in this module, or just a subset?**
   - If subset, consider splitting along usage lines
   - If all, it's likely cohesive

4. **Do functions share state or call each other?**
   - If yes, they belong together
   - If no, they might be separate concerns

---

## When to Review Python Code

Focus on these areas when reviewing Python:

- **Repository/ORM layer**: Are queries scattered? Is the ORM API leaking to callers?
- **Service layer**: Does it add logic or just forward calls?
- **API request/response**: Does it mirror the domain model exactly? Should they be separate?
- **Exception handling**: Are exceptions caught and re-raised? Can we return `None` instead?
- **Dataclass/Pydantic**: Are domain models leaking storage details (password, timestamps)?
- **Typing**: Are `Any`, `dict`, `list` used when more specific types could help?
- **Module structure**: Is the module easy to describe in one sentence?

---

## Python-Specific Tools to Help

- **Type hints**: Use `Optional[T]` instead of None-checking to express intent
- **dataclasses.field(init=False)**: Hide derived fields from callers
- **property decorators**: Hide attribute access; control mutations
- **__slots__**: Prevent accidental attribute assignment; clarify allowed state
- **frozen=True**: Make immutable dataclasses to prevent silent mutations
- **contextlib.contextmanager**: Clean up resource management without exposing details
- **enum.Enum**: Use instead of string/int constants for fixed sets
