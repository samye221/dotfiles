# Claude Global Configuration

## Universal Development Standards

This configuration applies to ALL development work across all projects. Rules are organized by context and automatically applied based on the working directory.

## Core Principles
- **NO COMMENTS** in generated code unless explicitly requested
- Write self-documenting code with clear variable and function names
- Always prefer editing existing files over creating new ones
- Never create documentation files unless explicitly requested

## Context-Based Rule Application

### Universal Rules (Always Applied)
Located in `~/.claude/rules/universal/` (automatically loaded by Claude Code):
- **typescript-standards.md** - Strict TypeScript, no any types
- **react-standards.md** - Component patterns, performance, React 19
- **testing-standards.md** - Jest, RTL, accessibility testing
- **performance-standards.md** - Memoization, bundle optimization
- **teaching-agent-standards.md** - Learning through action

**Note:** All `.md` files in `.claude/rules/` are automatically detected and loaded recursively with the same priority as CLAUDE.md.

### Development Commands Registry

#### Frontend/React Projects
```bash
pnpm install         # Install dependencies
pnpm run dev        # Start dev server
pnpm run build      # Build for production
pnpm run test       # Run tests
pnpm run lint       # Lint code
pnpm run lint:fix   # Fix linting issues
```

#### Python Projects
```bash
python -m venv venv           # Create virtual environment
source venv/bin/activate      # Activate venv
pip install -r requirements.txt # Install dependencies
pytest                        # Run tests
```

## Quality Standards (All Contexts)

### TypeScript
- ❌ **NO** `any` types in production code
- ✅ **REQUIRED** explicit return types for public functions
- ✅ **USE** type guards over type assertions
- ✅ **IMPLEMENT** Result pattern for fallible operations

### React
- ✅ **MANDATORY** data-testid on interactive elements
- ✅ **REQUIRED** React.memo for expensive components
- ✅ **IMPLEMENT** proper accessibility (ARIA, semantic HTML)
- ✅ **MOBILE FIRST** responsive design

### Testing
- ✅ **REQUIRED** jest-axe for accessibility
- ✅ **FOCUS** on user behavior, not implementation
- ✅ **MOCK** external dependencies properly
- ✅ **MAINTAIN** high coverage on critical paths

### Performance
- ✅ **USE** useMemo/useCallback appropriately
- ✅ **IMPLEMENT** code splitting by feature
- ✅ **OPTIMIZE** bundle size regularly
- ✅ **MONITOR** Core Web Vitals

## File Organization

### Component Files (React)
- Components: PascalCase (`UserProfile.tsx`)
- Hooks: camelCase with "use" prefix (`useUserData.ts`)
- Utils: camelCase (`formatDate.ts`)
- Constants: SCREAMING_SNAKE_CASE (`API_ENDPOINTS.ts`)

### Python Files
- Modules: snake_case (`web_scraper.py`)
- Classes: PascalCase (`WebDriver`)
- Functions: snake_case (`get_element_text`)
- Constants: SCREAMING_SNAKE_CASE (`MAX_RETRIES`)

## Security Guidelines
- Never commit API keys or secrets
- Use environment variables for configuration
- Validate all user inputs
- Follow OWASP guidelines
- Implement proper error boundaries

## Important Instructions
- Do what has been asked; nothing more, nothing less
- NEVER create files unless absolutely necessary
- ALWAYS prefer editing existing files
- NEVER proactively create documentation files

## Agent System
Specialized agents are available globally for complex tasks. They activate automatically based on context and can be invoked manually when needed. See `~/.claude/agents/` for available agents.

## MCP Servers
Model Context Protocol servers provide extended capabilities. Configuration in `~/.claude/settings.json`
