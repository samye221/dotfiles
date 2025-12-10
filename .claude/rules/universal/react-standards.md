# Advanced React Standards - React 19 Edition

## META-PROMPTING FRAMEWORK
<thinking>
When writing React code:
1. Analyze component responsibilities and boundaries
2. Apply composition over inheritance patterns
3. Optimize for performance and maintainability
4. Ensure accessibility by default
5. Leverage React 19 features appropriately
</thinking>

## ARCHITECTURAL SCAFFOLD

### Component Design Philosophy
```typescript
// 🎯 PATTERN: Single Responsibility Components
// Each component should have ONE clear purpose

// ✅ GOOD: Focused responsibility
interface UserAvatarProps {
  user: User
  size: 'small' | 'medium' | 'large'
  onClick?: () => void
}

export const UserAvatar: FC<UserAvatarProps> = ({ user, size, onClick }) => {
  // Single responsibility: Display user avatar with interaction
}

// ❌ BAD: Multiple responsibilities
interface UserDashboardProps {
  // Too many concerns in one component
  user: User
  posts: Post[]
  notifications: Notification[]
  settings: Settings
  onSettingsChange: (settings: Settings) => void
  onPostCreate: (post: Post) => void
}
```

### Chain-of-Thought Component Structure
EVERY component must follow this reasoning pattern:

```typescript
// 🧠 COMPONENT REASONING CHAIN
// Step 1: What is this component's single responsibility?
// Step 2: What are the minimum props needed?
// Step 3: What internal state is required?
// Step 4: How does this compose with other components?
// Step 5: What accessibility concerns exist?

interface ComponentProps {
  // Props that define WHAT this component does
  data: ComponentData
  
  // Props that define HOW it behaves
  variant?: 'primary' | 'secondary'
  
  // Props for composition and interaction
  onAction?: (result: ActionResult) => void
  children?: ReactNode
}

export const Component: FC<ComponentProps> = ({ 
  data, 
  variant = 'primary', 
  onAction,
  children 
}) => {
  // WHY: Default values prevent undefined behavior
  // WHY: Optional onAction enables flexible usage
  // WHY: children prop enables composition
}
```

## REACT 19 ADVANCED PATTERNS

### Actions and Form Handling
```typescript
// 🚀 REACT 19: Server Actions with useActionState
import { useActionState } from 'react'

interface ContactFormData {
  name: string
  email: string
  message: string
}

// Server action with proper error handling
async function submitContactForm(
  prevState: { success: boolean; error?: string },
  formData: FormData
): Promise<{ success: boolean; error?: string }> {
  try {
    const data = {
      name: formData.get('name') as string,
      email: formData.get('email') as string,
      message: formData.get('message') as string,
    }
    
    // Validation
    if (!data.name || !data.email || !data.message) {
      return { success: false, error: 'All fields are required' }
    }
    
    // Submit logic
    await submitToApi(data)
    return { success: true }
    
  } catch (error) {
    return { 
      success: false, 
      error: error instanceof Error ? error.message : 'Unknown error' 
    }
  }
}

// Component using React 19 patterns
export const ContactForm: FC = () => {
  const [state, formAction] = useActionState(
    submitContactForm,
    { success: false }
  )
  
  return (
    <form action={formAction} className="contact-form">
      {/* Built-in form status with useFormStatus */}
      <FormFields />
      {state.error && (
        <div role="alert" className="error-message">
          {state.error}
        </div>
      )}
      {state.success && (
        <div role="status" className="success-message">
          Message sent successfully!
        </div>
      )}
    </form>
  )
}

// Separate component for form fields with useFormStatus
const FormFields: FC = () => {
  const { pending } = useFormStatus()
  
  return (
    <>
      <input 
        name="name" 
        required 
        disabled={pending}
        aria-label="Your name"
      />
      <input 
        name="email" 
        type="email" 
        required 
        disabled={pending}
        aria-label="Your email"
      />
      <textarea 
        name="message" 
        required 
        disabled={pending}
        aria-label="Your message"
      />
      <button type="submit" disabled={pending}>
        {pending ? 'Sending...' : 'Send Message'}
      </button>
    </>
  )
}
```

### Optimistic Updates Pattern
```typescript
// 🚀 REACT 19: useOptimistic for better UX
interface TodoItem {
  id: string
  text: string
  completed: boolean
  optimistic?: boolean
}

export const TodoList: FC<{ initialTodos: TodoItem[] }> = ({ initialTodos }) => {
  const [todos, setTodos] = useState(initialTodos)
  const [optimisticTodos, addOptimisticTodo] = useOptimistic(
    todos,
    (state, newTodo: TodoItem) => [...state, newTodo]
  )
  
  const addTodo = async (text: string) => {
    const optimisticTodo: TodoItem = {
      id: crypto.randomUUID(),
      text,
      completed: false,
      optimistic: true
    }
    
    // Show optimistic update immediately
    addOptimisticTodo(optimisticTodo)
    
    try {
      const savedTodo = await saveTodoToServer(text)
      setTodos(prev => [...prev, savedTodo])
    } catch (error) {
      // Optimistic update automatically reverts on error
      console.error('Failed to save todo:', error)
    }
  }
  
  return (
    <div>
      {optimisticTodos.map(todo => (
        <TodoItem 
          key={todo.id} 
          todo={todo}
          className={todo.optimistic ? 'opacity-50' : ''}
        />
      ))}
      <AddTodoForm onAdd={addTodo} />
    </div>
  )
}
```

## COMPOSITION PATTERNS

### Compound Components with Context
```typescript
// 🎯 PATTERN: Compound Components for flexible APIs
interface AccordionContextValue {
  openItems: Set<string>
  toggle: (id: string) => void
}

const AccordionContext = createContext<AccordionContextValue | null>(null)

const useAccordion = () => {
  const context = useContext(AccordionContext)
  if (!context) {
    throw new Error('Accordion components must be used within Accordion')
  }
  return context
}

// Root component manages state
export const Accordion: FC<{ children: ReactNode; multiple?: boolean }> = ({ 
  children, 
  multiple = false 
}) => {
  const [openItems, setOpenItems] = useState<Set<string>>(new Set())
  
  const toggle = useCallback((id: string) => {
    setOpenItems(prev => {
      const next = new Set(prev)
      if (next.has(id)) {
        next.delete(id)
      } else {
        if (!multiple) next.clear()
        next.add(id)
      }
      return next
    })
  }, [multiple])
  
  const value = useMemo(() => ({ openItems, toggle }), [openItems, toggle])
  
  return (
    <AccordionContext.Provider value={value}>
      <div role="tablist">{children}</div>
    </AccordionContext.Provider>
  )
}

// Child components use context
export const AccordionItem: FC<{ id: string; children: ReactNode }> = ({ 
  id, 
  children 
}) => {
  const { openItems } = useAccordion()
  const isOpen = openItems.has(id)
  
  return (
    <div 
      className={`accordion-item ${isOpen ? 'open' : 'closed'}`}
      data-state={isOpen ? 'open' : 'closed'}
    >
      {children}
    </div>
  )
}

export const AccordionTrigger: FC<{ 
  id: string
  children: ReactNode 
}> = ({ id, children }) => {
  const { openItems, toggle } = useAccordion()
  const isOpen = openItems.has(id)
  
  return (
    <button
      type="button"
      role="tab"
      aria-expanded={isOpen}
      aria-controls={`panel-${id}`}
      onClick={() => toggle(id)}
      className="accordion-trigger"
    >
      {children}
      <span aria-hidden="true">{isOpen ? '−' : '+'}</span>
    </button>
  )
}

export const AccordionContent: FC<{ 
  id: string
  children: ReactNode 
}> = ({ id, children }) => {
  const { openItems } = useAccordion()
  const isOpen = openItems.has(id)
  
  if (!isOpen) return null
  
  return (
    <div
      role="tabpanel"
      id={`panel-${id}`}
      className="accordion-content"
    >
      {children}
    </div>
  )
}

// Usage shows the power of composition
const App = () => (
  <Accordion multiple>
    <AccordionItem id="item1">
      <AccordionTrigger id="item1">Section 1</AccordionTrigger>
      <AccordionContent id="item1">Content 1</AccordionContent>
    </AccordionItem>
    <AccordionItem id="item2">
      <AccordionTrigger id="item2">Section 2</AccordionTrigger>
      <AccordionContent id="item2">Content 2</AccordionContent>
    </AccordionItem>
  </Accordion>
)
```

## PERFORMANCE OPTIMIZATION

### Intelligent Memoization Strategy
```typescript
// 🎯 PATTERN: Strategic memoization based on actual need
interface ExpensiveComponentProps {
  data: ComplexData[]
  filters: FilterOptions
  onSelect: (item: ComplexData) => void
}

// Only memoize when props are truly expensive to compare
export const ExpensiveComponent = memo<ExpensiveComponentProps>(({
  data,
  filters, 
  onSelect
}) => {
  // Expensive computation - memoize this
  const processedData = useMemo(() => {
    return data
      .filter(item => matchesFilters(item, filters))
      .sort((a, b) => a.priority - b.priority)
      .map(item => enhanceWithMetadata(item))
  }, [data, filters])
  
  // Stable callback - prevent child re-renders
  const handleSelect = useCallback((item: ComplexData) => {
    // Add analytics or validation here
    trackSelection(item)
    onSelect(item)
  }, [onSelect])
  
  return (
    <div className="expensive-component">
      {processedData.map(item => (
        <ExpensiveItem
          key={item.id}
          item={item}
          onSelect={handleSelect}
        />
      ))}
    </div>
  )
}, 
// Custom comparison for complex props
(prevProps, nextProps) => {
  return (
    prevProps.data === nextProps.data &&
    shallowEqual(prevProps.filters, nextProps.filters) &&
    prevProps.onSelect === nextProps.onSelect
  )
})

ExpensiveComponent.displayName = 'ExpensiveComponent'
```

### Concurrent Features Integration
```typescript
// 🚀 REACT 19: Leveraging concurrent features
import { use, Suspense, startTransition } from 'react'

// Resource pattern for data fetching
const createResource = <T,>(promise: Promise<T>) => {
  let status = 'pending'
  let result: T
  let error: Error
  
  const suspender = promise.then(
    (res) => {
      status = 'success'
      result = res
    },
    (err) => {
      status = 'error'
      error = err
    }
  )
  
  return () => {
    if (status === 'pending') throw suspender
    if (status === 'error') throw error
    return result
  }
}

// Component using concurrent features
const UserProfile: FC<{ userId: string }> = ({ userId }) => {
  const userResource = useMemo(
    () => createResource(fetchUser(userId)),
    [userId]
  )
  
  const user = use(userResource())
  
  return (
    <div className="user-profile">
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  )
}

// App with proper Suspense boundaries
export const App: FC = () => {
  const [userId, setUserId] = useState('1')
  
  const handleUserChange = (newUserId: string) => {
    startTransition(() => {
      setUserId(newUserId)
    })
  }
  
  return (
    <div>
      <UserSelector onSelect={handleUserChange} />
      <Suspense fallback={<UserProfileSkeleton />}>
        <UserProfile userId={userId} />
      </Suspense>
    </div>
  )
}
```

## ACCESSIBILITY FRAMEWORK

### Built-in Accessibility Patterns
```typescript
// 🛡️ ACCESSIBILITY: Default accessible components
interface ButtonProps extends ComponentPropsWithoutRef<'button'> {
  variant?: 'primary' | 'secondary' | 'danger'
  size?: 'small' | 'medium' | 'large'
  loading?: boolean
  children: ReactNode
}

export const Button = forwardRef<HTMLButtonElement, ButtonProps>(({
  variant = 'primary',
  size = 'medium', 
  loading = false,
  disabled,
  children,
  className,
  ...props
}, ref) => {
  const classes = cn(
    'button',
    `button--${variant}`,
    `button--${size}`,
    loading && 'button--loading',
    className
  )
  
  return (
    <button
      ref={ref}
      className={classes}
      disabled={disabled || loading}
      aria-disabled={disabled || loading}
      {...props}
    >
      {loading && (
        <>
          <span className="button__spinner" aria-hidden="true" />
          <span className="sr-only">Loading...</span>
        </>
      )}
      <span className={loading ? 'opacity-0' : undefined}>
        {children}
      </span>
    </button>
  )
})

Button.displayName = 'Button'

// Form components with built-in accessibility
interface InputProps extends ComponentPropsWithoutRef<'input'> {
  label: string
  error?: string
  hint?: string
}

export const Input = forwardRef<HTMLInputElement, InputProps>(({
  label,
  error,
  hint,
  id,
  className,
  ...props
}, ref) => {
  const inputId = useId()
  const finalId = id || inputId
  const hintId = `${finalId}-hint`
  const errorId = `${finalId}-error`
  
  return (
    <div className="input-group">
      <label htmlFor={finalId} className="input-label">
        {label}
        {props.required && <span aria-label="required">*</span>}
      </label>
      
      {hint && (
        <div id={hintId} className="input-hint">
          {hint}
        </div>
      )}
      
      <input
        ref={ref}
        id={finalId}
        className={cn(
          'input',
          error && 'input--error',
          className
        )}
        aria-describedby={cn(
          hint && hintId,
          error && errorId
        )}
        aria-invalid={!!error}
        {...props}
      />
      
      {error && (
        <div id={errorId} role="alert" className="input-error">
          {error}
        </div>
      )}
    </div>
  )
})

Input.displayName = 'Input'
```

## TESTING INTEGRATION

### Test-Friendly Component Design
```typescript
// 🧪 TESTING: Components designed for testability
interface SearchableListProps<T> {
  items: T[]
  renderItem: (item: T) => ReactNode
  onItemSelect: (item: T) => void
  searchField: keyof T
  placeholder?: string
  'data-testid'?: string
}

export const SearchableList = <T extends Record<string, any>>({
  items,
  renderItem,
  onItemSelect,
  searchField,
  placeholder = 'Search...',
  'data-testid': testId = 'searchable-list'
}: SearchableListProps<T>) => {
  const [searchTerm, setSearchTerm] = useState('')
  
  const filteredItems = useMemo(() => {
    if (!searchTerm) return items
    return items.filter(item => 
      String(item[searchField])
        .toLowerCase()
        .includes(searchTerm.toLowerCase())
    )
  }, [items, searchField, searchTerm])
  
  return (
    <div data-testid={testId} className="searchable-list">
      <input
        data-testid={`${testId}-search`}
        type="search"
        placeholder={placeholder}
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="search-input"
      />
      
      <div 
        data-testid={`${testId}-results`}
        role="listbox"
        aria-label="Search results"
      >
        {filteredItems.length === 0 ? (
          <div 
            data-testid={`${testId}-no-results`}
            className="no-results"
          >
            No items found
          </div>
        ) : (
          filteredItems.map((item, index) => (
            <div
              key={item.id || index}
              data-testid={`${testId}-item-${index}`}
              role="option"
              tabIndex={0}
              onClick={() => onItemSelect(item)}
              onKeyDown={(e) => {
                if (e.key === 'Enter' || e.key === ' ') {
                  e.preventDefault()
                  onItemSelect(item)
                }
              }}
              className="list-item"
            >
              {renderItem(item)}
            </div>
          ))
        )}
      </div>
    </div>
  )
}
```

## QUALITY GATES

### Component Checklist
EVERY component must satisfy:
- [ ] Single, clear responsibility
- [ ] Proper TypeScript typing with no `any`
- [ ] forwardRef for DOM components
- [ ] displayName for debugging
- [ ] Proper accessibility attributes
- [ ] data-testid for testing
- [ ] Error boundaries where appropriate
- [ ] Performance optimization if needed

### Code Review Standards
- [ ] Composition over inheritance
- [ ] Proper hook dependencies
- [ ] Stable references for callbacks
- [ ] Accessibility compliance
- [ ] Test-friendly structure
- [ ] React 19 features used appropriately

## ACTIVATION
These patterns apply automatically to ALL React components.
Every component should be accessible, performant, and maintainable by default.