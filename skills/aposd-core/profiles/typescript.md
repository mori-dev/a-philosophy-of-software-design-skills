# APoSD TypeScript Profile

TypeScript-specific red flags and patterns to watch for during code review.

## Common Shallow Module Patterns

### API Payload Mirror (No Data Transformation)

**Pattern**:
```typescript
// API response mirrors database schema exactly
interface User {
  user_id: number;
  user_name: string;
  created_at: string;
  updated_at: string;
  password_hash: string;
}

// Controller
const user = await db.query('SELECT * FROM users WHERE id = ?', id);
return res.json(user);  // Send DB row directly
```

**Problem**:
- API contract tightly coupled to database schema
- Callers can access storage details (password_hash, timestamps)
- Change to DB schema requires API versioning

**Deepen It**:
```typescript
// Domain model, not storage detail
interface User {
  id: number;
  name: string;
  email: string;
}

// API response, separate from storage
interface UserResponse {
  id: number;
  name: string;
  email: string;
  // Storage fields hidden; caller doesn't see password_hash
}

// Repository hides database details
const user = await getUserFromDB(id);
return UserResponse.fromUser(user);  // Transform, don't expose
```

---

## Pass-Through Hooks and Services

**Pattern**:
```typescript
// Custom hook that just calls API
function useUser(id: string) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    userService.getUser(id).then(setUser);  // Just calling service
  }, [id]);
  
  return user;
}

// Service that just calls API
class UserService {
  getUser(id: string): Promise<User> {
    return this.api.get(`/users/${id}`);  // Just forwarding to API
  }
}

// API client
class UserAPI {
  get(url: string): Promise<any> {
    return fetch(url).then(r => r.json());  // Just HTTP wrapper
  }
}
```

**Problem**:
- Multiple layers with no real logic
- Caller must know about service, hook, API, all doing the same thing
- Changes ripple through layers

**Deepen It**:
```typescript
// Hook hides caching, error handling, loading state
function useUser(id: string) {
  const cache = useRef(new Map());
  const [state, dispatch] = useReducer(userReducer, initialState);
  
  useEffect(() => {
    // Caching, retries, error handling all here
    if (cache.current.has(id)) {
      dispatch({ type: 'SET_USER', user: cache.current.get(id) });
      return;
    }
    
    dispatch({ type: 'LOADING' });
    fetchUser(id)
      .then(user => {
        cache.current.set(id, user);
        dispatch({ type: 'SET_USER', user });
      })
      .catch(err => dispatch({ type: 'ERROR', error: err }));
  }, [id]);
  
  return state;  // Caller gets loading/error/user all in one place
}

// No separate service layer needed; logic is in the hook
```

---

## Boolean Parameter Explosion

**Pattern**:
```typescript
function fetchUserData(
  userId: string,
  includeProfile: boolean = false,
  includeOrders: boolean = false,
  includeSubscription: boolean = false,
  includePaymentMethods: boolean = false,
  includeAuditLog: boolean = false,
  useCache: boolean = true,
  retryOnFailure: boolean = true,
) {
  // 2^7 = 128 possible combinations
  if (includeProfile && includeOrders && !includeSubscription) {
    // specific logic
  } else if (includeAuditLog && !useCache) {
    // different logic
  }
  // ...
}

// Caller must make many decisions
const data = await fetchUserData(
  userId,
  true,
  true,
  false,
  true,
  false,
  true,
  true
);  // What does this even do?
```

**Problem**:
- Boolean parameters hide intent
- Invalid combinations not prevented
- Cognitive load on caller

**Fix**:
```typescript
// Separate, clearly-named functions
async function getUserProfile(userId: string) {
  return cached(() => api.get(`/users/${userId}`));
}

async function getUserWithOrders(userId: string) {
  return cached(() => api.get(`/users/${userId}/orders`));
}

async function getUserAuditLog(userId: string, skipCache: boolean = false) {
  const fetcher = () => api.get(`/users/${userId}/audit`);
  return skipCache ? fetcher() : cached(fetcher);
}

// Or use object parameter with explicit interface
interface UserQueryOptions {
  profile?: boolean;
  orders?: boolean;
  subscription?: boolean;
  cache?: boolean;
}

async function fetchUserData(userId: string, options: UserQueryOptions) {
  // Callers are explicit about what they want
  const queries = [];
  if (options.profile) queries.push(api.get(`/users/${userId}`));
  if (options.orders) queries.push(api.get(`/users/${userId}/orders`));
  // ...
  return Promise.all(queries);
}
```

---

## Information Leakage: Component Domain Confusion

**Pattern**:
```typescript
// Component mixes API contracts with business logic
interface Product {
  id: string;
  name: string;
  price: number;
  inventory_count: number;     // DB column
  warehouse_location: string;   // Business detail!
  created_at: string;           // Storage timestamp
}

function ProductCard({ product }: { product: Product }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      {product.inventory_count > 0 ? (
        <button>Add to Cart</button>
      ) : (
        <span>Out of Stock</span>
      )}
      {product.warehouse_location && (
        <small>Available at: {product.warehouse_location}</small>
      )}
    </div>
  );
}
```

**Problem**:
- Component knows about warehouse internals
- If warehouse location storage changes, component breaks
- API contract is tightly coupled to storage

**Deepen It**:
```typescript
// Domain model, separate from API/storage
interface Product {
  id: string;
  name: string;
  price: number;
  isAvailable: boolean;  // Boolean, not count
  availabilityMessage?: string;  // Pre-formatted in API layer
}

// Conversion class hides storage details
class ProductResponse implements Product {
  id: string;
  name: string;
  price: number;
  isAvailable: boolean;
  availabilityMessage?: string;

  static fromDB(dbRow: any): ProductResponse {
    const response = new ProductResponse();
    response.id = dbRow.id;
    response.name = dbRow.name;
    response.price = dbRow.price;
    response.isAvailable = dbRow.inventory_count > 0;
    response.availabilityMessage = dbRow.warehouse_location
      ? `Available at: ${dbRow.warehouse_location}`
      : undefined;
    return response;
  }
}

// Component uses only domain model
function ProductCard({ product }: { product: Product }) {
  return (
    <div>
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      {product.isAvailable ? (
        <button>Add to Cart</button>
      ) : (
        <span>Out of Stock</span>
      )}
      {product.availabilityMessage && (
        <small>{product.availabilityMessage}</small>
      )}
    </div>
  );
}

// API layer transforms storage into domain model
const product = ProductResponse.fromDB(dbRow);
```

---

## Vague Naming

### Common Offenders in TypeScript

| Avoid | Use Instead |
|-------|------------|
| `handle` | `onSubmit`, `processFormSubmission`, `validateAndSave` |
| `process` | `convertCSVToJSON`, `aggregateSalesData`, `filterActiveUsers` |
| `data` | `users`, `orders`, `response`, `payload` |
| `state` | `userSession`, `formState`, `cartItems` |
| `utils` | `dateFormatting`, `stringValidation`, `retryLogic` |
| `helper` | (reconsider; most helpers are really domain logic) |
| `get`, `set` | `fetchUser`, `updateUserEmail`, `addToCart` |
| `fetch` | (only for HTTP; use `query`, `load`, `search` for data operations) |

**React-specific**: Prefer clear handler names over `handleX`:

```typescript
// Less clear
function handleClick() {}
function handleChange(event) {}
function handleSubmit() {}

// Clearer: expresses intent
function addItemToCart() {}
function updateSearchQuery(query: string) {}
function submitOrderForm() {}
```

---

## Error Handling Scattered

**Pattern**:
```typescript
// Multiple try-catch blocks for the same operation
async function loadUser(id: string) {
  try {
    const user = await fetch(`/api/users/${id}`).then(r => r.json());
    return user;
  } catch (error) {
    console.error("Failed to load user");
    return null;
  }
}

// Hook also catches
function useUser(id: string) {
  useEffect(() => {
    loadUser(id)
      .catch(err => {
        console.error("Error in useUser:", err);
        setError(err);
      });
  }, [id]);
}

// Component also wraps in error boundary
<ErrorBoundary>
  <UserProfile userId={id} />
</ErrorBoundary>
```

**Problem**:
- Error handling scattered across layers
- Context lost as it bubbles
- Unclear where error should be caught

**Fix**:
```typescript
// Single point of error handling: the hook
function useUser(id: string) {
  const [state, setState] = useState<UserState>({
    status: 'idle',
    data: null,
    error: null,
  });

  useEffect(() => {
    setState({ status: 'loading', data: null, error: null });
    
    fetch(`/api/users/${id}`)
      .then(r => r.json())
      .then(user => setState({ status: 'success', data: user, error: null }))
      .catch(error => {
        console.error(`Failed to load user ${id}:`, error);
        setState({ status: 'error', data: null, error });
      });
  }, [id]);

  return state;
}

// Component doesn't need error boundary; hook handles it
function UserProfile({ userId }: { userId: string }) {
  const user = useUser(userId);
  
  if (user.status === 'loading') return <div>Loading...</div>;
  if (user.status === 'error') return <div>Error: {user.error.message}</div>;
  if (!user.data) return null;
  
  return <div>{user.data.name}</div>;
}
```

---

## Change Amplification: Updating Data Models

**Pattern**:
```typescript
// Add a field to User, and it ripples everywhere
interface User {
  id: string;
  name: string;
  email: string;
  subscriptionStatus: string;  // New field
}

// API endpoint returns User directly
export async function getUser(id: string): Promise<User> {
  const dbUser = await db.users.findOne({ id });
  return dbUser;
}

// Component uses User directly
function UserProfile({ user }: { user: User }) {
  return (
    <div>
      {user.name}
      <span className={user.subscriptionStatus}>
        {user.subscriptionStatus === 'active' ? 'Active' : 'Inactive'}
      </span>
    </div>
  );
}
```

**Problem**:
- Adding one User field changes API contract, component, tests, etc.
- Multiple layers tightly coupled

**Fix**: Separate data layers

```typescript
// Database model (internal)
interface UserRecord {
  id: string;
  name: string;
  email: string;
  subscription_status: string;
  created_at: string;
  updated_at: string;
}

// Domain model (what the app thinks)
interface User {
  id: string;
  name: string;
  isSubscriptionActive: boolean;
}

// API only talks about domain model
export async function getUser(id: string): Promise<User> {
  const dbUser = await db.users.findOne({ id });
  return {
    id: dbUser.id,
    name: dbUser.name,
    isSubscriptionActive: dbUser.subscription_status === 'active',
  };
}

// Component doesn't care about subscription_status naming
function UserProfile({ user }: { user: User }) {
  return (
    <div>
      {user.name}
      <span className={user.isSubscriptionActive ? 'active' : 'inactive'}>
        {user.isSubscriptionActive ? 'Active' : 'Inactive'}
      </span>
    </div>
  );
}
```

Now, DB schema changes don't ripple to components.

---

## TypeScript-Specific Patterns

### Any/Unknown Overflow

**Pattern**:
```typescript
// Too generic
function processData(data: any): any {
  return data;
}

// Component accepts anything
interface Props {
  data: any;
  onUpdate: (value: any) => void;
}
```

**Problem**:
- Loss of type safety
- Caller doesn't know what's expected
- Errors caught at runtime, not build time

**Fix**: Be specific

```typescript
// Clear input and output
function processUserData(data: User): UserResponse {
  return {
    id: data.id,
    name: data.name,
  };
}

// Props typed clearly
interface UserCardProps {
  user: User;
  onEmailChange: (email: string) => void;
}

// If you need generic code, use generics with constraints
function fetchData<T extends { id: string }>(id: string): Promise<T> {
  return fetch(`/api/${id}`).then(r => r.json());
}
```

### Discriminated Unions for Complexity

**Pattern**: Handle state/response types explicitly

```typescript
// Unclear what happens when
type UserResponse = User | null | Error;

// Clear: exhaustive cases
type UserLoadState =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; user: User }
  | { status: 'error'; error: string };

function renderUser(state: UserLoadState) {
  switch (state.status) {
    case 'idle':
      return null;
    case 'loading':
      return <div>Loading...</div>;
    case 'success':
      return <div>{state.user.name}</div>;
    case 'error':
      return <div>Error: {state.error}</div>;
  }
}
```

---

## When to Review TypeScript Code

Focus on these areas:

- **API contracts**: Does the API response mirror database schema? Should they be separate?
- **Component props**: Are props too generic (object, any)? Can types be more specific?
- **Service/hook layers**: Do they add logic or just forward calls?
- **Error handling**: Are errors caught and re-thrown? Can operations return `null` instead?
- **State management**: Is state scattered across multiple places? Should it be centralized?
- **Type generics**: Is `any` used when specific types could help?

---

## TypeScript-Specific Tools

- **Discriminated unions**: Express state machines explicitly; `switch` over status
- **never type**: Catch missing cases at compile time
- **Readonly**: Prevent accidental mutations; signal intent
- **Branded types**: Distinguish similar strings (UserId vs OrderId)
- **as const**: Lock object types and prevent type widening
- **Type predicates**: Write custom type guards; `function isUser(x: any): x is User { ... }`
- **Exhaustiveness checks**: Use `const _exhaustiveness: never = state;` to catch missing cases
