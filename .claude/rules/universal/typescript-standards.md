# Advanced TypeScript Standards - 2025 Edition

## META-PROMPTING FRAMEWORK
<thinking>
When writing TypeScript code:
1. Analyze type safety requirements
2. Apply strictest possible typing
3. Consider runtime safety implications
4. Validate type coverage and correctness
5. Optimize for developer experience
</thinking>

## CORE DIRECTIVES (NON-NEGOTIABLE)

### Type Safety Scaffold
```typescript
// ✅ MANDATORY: Explicit typing pattern
interface ConfiguredFunction<T, R> {
  (input: T): Promise<Result<R, Error>>
}

// ❌ FORBIDDEN: Any escape hatches
const process = (data: any) => { /* NO */ }
const result = apiCall() as SomeType // Avoid
```

### Chain-of-Thought Type Design
WHEN designing types, ALWAYS document reasoning:

```typescript
// 🧠 TYPE REASONING CHAIN
// Step 1: Domain analysis - what are we modeling?
// Step 2: Variance analysis - what can change?
// Step 3: Safety analysis - what can go wrong?
// Step 4: Evolution analysis - how will this grow?

interface UserProfile {
  // Core identity (immutable after creation)
  readonly id: UserId
  readonly createdAt: Timestamp
  
  // Mutable profile data
  displayName: string
  email: EmailAddress
  
  // Optional features (explicit undefined handling)
  avatar?: ImageUrl
  preferences?: UserPreferences
}

// WHY: Readonly prevents accidental mutation of identity
// WHY: Branded types (UserId, EmailAddress) prevent mixing concepts
// WHY: Optional with ? forces explicit undefined handling
```

## ADVANCED TYPE PATTERNS

### Result Pattern (Mandatory for Fallible Operations)
```typescript
// 🎯 PATTERN: Railway-Oriented Programming
type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E }

// Self-documenting error handling
const parseUserInput = (input: string): Result<User, ValidationError> => {
  // Implementation forces explicit error handling at call site
}

// Chain-of-thought usage
const processUser = async (rawInput: string) => {
  const parseResult = parseUserInput(rawInput)
  if (!parseResult.success) {
    // TypeScript forces us to handle the error case
    return handleValidationError(parseResult.error)
  }
  
  // TypeScript knows parseResult.data is User here
  return await saveUser(parseResult.data)
}
```

### Branded Types (Prevent Primitive Obsession)
```typescript
// 🛡️ SAFETY: Prevent mixing of conceptually different strings
type UserId = string & { readonly __brand: 'UserId' }
type EmailAddress = string & { readonly __brand: 'EmailAddress' }
type ProductId = string & { readonly __brand: 'ProductId' }

// Factory functions with validation
const createUserId = (value: string): Result<UserId, Error> => {
  if (!isValidUserId(value)) {
    return { success: false, error: new Error('Invalid user ID format') }
  }
  return { success: true, data: value as UserId }
}

// 🧠 REASONING: Prevents accidentally using ProductId where UserId expected
// Compile-time safety for runtime concepts
```

### Discriminated Unions (State Machine Types)
```typescript
// 🎯 PATTERN: Make illegal states unrepresentable
type AsyncData<T, E = Error> = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: E }

// Self-validating state transitions
const useAsyncData = <T>(fetcher: () => Promise<T>) => {
  const [state, setState] = useState<AsyncData<T>>({ status: 'idle' })
  
  const fetch = async () => {
    setState({ status: 'loading' })
    try {
      const data = await fetcher()
      setState({ status: 'success', data })
    } catch (error) {
      setState({ status: 'error', error: error as Error })
    }
  }
  
  return { state, fetch }
}

// 🧠 REASONING: Impossible to have data and error simultaneously
// TypeScript prevents accessing data in error state
```

## TYPE GUARDS (RUNTIME SAFETY)

### Structured Validation Pattern
```typescript
// 🛡️ SAFETY SCAFFOLD: Runtime type validation
const isUser = (obj: unknown): obj is User => {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'email' in obj &&
    typeof (obj as any).id === 'string' &&
    typeof (obj as any).email === 'string'
  )
}

// Advanced: Schema-based validation
import { z } from 'zod'

const UserSchema = z.object({
  id: z.string().min(1),
  email: z.string().email(),
  displayName: z.string().min(1),
  createdAt: z.date()
})

type User = z.infer<typeof UserSchema>

// Self-validating with detailed errors
const parseUser = (data: unknown): Result<User, z.ZodError> => {
  const result = UserSchema.safeParse(data)
  if (result.success) {
    return { success: true, data: result.data }
  }
  return { success: false, error: result.error }
}
```

## FUNCTION SIGNATURE DESIGN

### Self-Documenting APIs
```typescript
// 🎯 PATTERN: Make intentions explicit through types
interface UserRepository {
  // Explicit return types document expectations
  findById(id: UserId): Promise<Option<User>>
  findByEmail(email: EmailAddress): Promise<Option<User>>
  save(user: User): Promise<Result<User, SaveError>>
  delete(id: UserId): Promise<Result<void, DeleteError>>
}

// Option type prevents null/undefined confusion
type Option<T> = T | null

// Specific error types enable proper error handling
class SaveError extends Error {
  constructor(
    message: string,
    public readonly code: 'VALIDATION_FAILED' | 'DATABASE_ERROR'
  ) {
    super(message)
  }
}
```

### Generic Constraints (Smart Defaults)
```typescript
// 🎯 PATTERN: Constrain generics for safety and inference
interface ApiResponse<T = unknown> {
  data: T
  meta: ResponseMeta
  errors?: ApiError[]
}

// Constraint ensures only serializable types
interface SerializableApiResponse<T extends Record<string, unknown>> {
  data: T
  timestamp: string
  version: string
}

// Self-documenting utility types
type NonEmptyArray<T> = [T, ...T[]]
type AtLeastOne<T> = T & { [K in keyof T]: Required<Pick<T, K>> }[keyof T]
```

## ERROR HANDLING PATTERNS

### Layered Error Strategy
```typescript
// 🛡️ SAFETY: Structured error handling hierarchy
abstract class AppError extends Error {
  abstract readonly code: string
  abstract readonly layer: 'domain' | 'application' | 'infrastructure'
}

class ValidationError extends AppError {
  readonly code = 'VALIDATION_ERROR'
  readonly layer = 'domain'
  
  constructor(
    message: string,
    public readonly field: string,
    public readonly value: unknown
  ) {
    super(message)
  }
}

class NetworkError extends AppError {
  readonly code = 'NETWORK_ERROR'
  readonly layer = 'infrastructure'
  
  constructor(
    message: string,
    public readonly status?: number,
    public readonly cause?: Error
  ) {
    super(message)
  }
}

// Type-safe error handling
const handleError = (error: AppError): ErrorResponse => {
  switch (error.code) {
    case 'VALIDATION_ERROR':
      // TypeScript knows this is ValidationError
      return { 
        status: 400, 
        message: `Invalid ${error.field}: ${error.value}` 
      }
    case 'NETWORK_ERROR':
      // TypeScript knows this is NetworkError
      return { 
        status: error.status ?? 500, 
        message: 'Network operation failed' 
      }
    default:
      // Exhaustiveness check - TypeScript error if case missed
      const _exhaustive: never = error
      return { status: 500, message: 'Unknown error' }
  }
}
```

## CONFIGURATION & ENVIRONMENT

### Type-Safe Environment Variables
```typescript
// 🛡️ SAFETY: Validate environment at startup
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'staging', 'production']),
  DATABASE_URL: z.string().url(),
  API_SECRET: z.string().min(32),
  PORT: z.coerce.number().int().min(1000).max(65535).default(3000)
})

type Environment = z.infer<typeof envSchema>

// Fail fast on invalid environment
const env: Environment = envSchema.parse(process.env)

// Usage is now type-safe throughout application
const startServer = (port: number = env.PORT) => {
  // TypeScript knows port is a valid number
}
```

## INTEGRATION PATTERNS

### External API Integration
```typescript
// 🎯 PATTERN: Type-safe external integrations
interface ExternalUserApi {
  getUser(id: string): Promise<unknown> // External API returns unknown
}

// Internal domain model
interface User {
  id: UserId
  email: EmailAddress
  displayName: string
}

// Adapter with validation
class UserApiAdapter {
  constructor(private api: ExternalUserApi) {}
  
  async getUser(id: UserId): Promise<Result<User, Error>> {
    try {
      const rawData = await this.api.getUser(id)
      const parseResult = parseUser(rawData)
      
      if (!parseResult.success) {
        return { 
          success: false, 
          error: new Error(`Invalid user data: ${parseResult.error.message}`) 
        }
      }
      
      return { success: true, data: parseResult.data }
    } catch (error) {
      return { 
        success: false, 
        error: error instanceof Error ? error : new Error('Unknown error') 
      }
    }
  }
}
```

## QUALITY GATES

### Pre-commit Validation
EVERY TypeScript file must pass:
- [ ] `tsc --noEmit` (type checking)
- [ ] No `any` types (except in type guards)
- [ ] All public functions have explicit return types  
- [ ] Error handling uses Result pattern
- [ ] External data validated with type guards

### Code Review Checklist
- [ ] Branded types for domain concepts
- [ ] Discriminated unions for state machines
- [ ] Result types for fallible operations
- [ ] Explicit error types with proper hierarchy
- [ ] Type guards for runtime validation
- [ ] Generic constraints where applicable

## ACTIVATION
These standards apply automatically to ALL TypeScript code.
Type safety is not optional - it's the foundation of reliable software.