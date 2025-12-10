# Performance Standards

## React Performance
- **REQUIRED** React.memo for expensive components
- Use useMemo for expensive calculations
- Use useCallback for event handlers passed to child components
- Implement proper dependency arrays

## Bundle Optimization
- Code splitting by domain and routes
- Lazy loading for non-critical components
- Tree shaking for unused code
- Monitor bundle size regularly

## Memoization Guidelines
```typescript
// ✅ Good - expensive calculation
const expensiveValue = useMemo(() => {
  return heavyComputation(data)
}, [data])

// ✅ Good - callback passed to children
const handleClick = useCallback((id: string) => {
  onItemClick(id)
}, [onItemClick])

// ❌ Bad - unnecessary memoization
const simpleValue = useMemo(() => value + 1, [value])
```

## Image Optimization
- Use Next.js Image component for all images
- Implement proper lazy loading
- Use appropriate image formats (WebP, AVIF)
- Optimize image sizes for different viewports

## Loading States
- Implement skeleton screens for better perceived performance
- Use proper loading indicators
- Avoid layout shifts during content loading

## Network Optimization
- Implement request deduplication
- Use appropriate caching strategies
- Minimize API calls through smart data fetching
- Implement pagination for large datasets

## Monitoring
- Track Core Web Vitals
- Monitor bundle size changes
- Use performance profiling tools
- Set up performance budgets