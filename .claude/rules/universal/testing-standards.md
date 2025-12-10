# Testing Standards

## Core Requirements
- **MANDATORY** data-testid on all interactive elements
- Use Jest and React Testing Library
- Place tests in `__tests__` directories or `.spec.ts` files
- Mock external dependencies
- Focus on user behavior, not implementation
- **ENFORCE** accessibility tests with jest-axe

## Test Structure
```typescript
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { axe, toHaveNoViolations } from 'jest-axe'

expect.extend(toHaveNoViolations)

describe('ComponentName', () => {
  it('renders without accessibility violations', async () => {
    const { container } = render(<ComponentName />)
    const results = await axe(container)
    expect(results).toHaveNoViolations()
  })

  it('handles user interaction correctly', async () => {
    const user = userEvent.setup()
    render(<ComponentName />)
    
    const button = screen.getByTestId('submit-button')
    await user.click(button)
    
    expect(screen.getByText('Success message')).toBeInTheDocument()
  })
})
```

## Data Test IDs
- Use descriptive, kebab-case names: `data-testid="user-profile-form"`
- Include on all clickable elements, form inputs, and important content
- Make them stable (don't use dynamic values)

## Mocking
- Mock API calls and external services
- Use MSW for HTTP mocking when possible
- Create reusable mock factories for complex objects

## Coverage
- Aim for high coverage but focus on critical paths
- Test error states and edge cases
- Don't test implementation details

## Accessibility Testing
- Always include axe-core tests
- Test keyboard navigation
- Verify ARIA attributes
- Check color contrast and focus management