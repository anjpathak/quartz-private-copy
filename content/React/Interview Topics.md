---
{"publish":true,"created":"2026-08-12T16:22:11.369Z","modified":"2026-08-17T03:03:42.735Z"}
---

# React Concepts for Senior-Level Interviews

As a senior engineer, you'll want to go beyond basics. Here are the key concepts interviewers focus on:

## **Core Fundamentals (Assume You Know These)**

- **JSX & Component Model**: Understand the virtual DOM, reconciliation, and why React's component model matters
- **Props vs State**: Data flow, immutability, and uni-directional data binding
- **Component Types**: Functional components (now standard), class components, and why the shift happened

## **Hooks (Critical for Modern React)**

**Essential:**

- **useState**: State management, closure issues, multiple states
- **useEffect**: Dependency arrays, cleanup functions, side effect patterns, why it runs twice in dev mode
- **useContext**: Context API alternatives to prop drilling, performance implications

**Advanced:**

- **useReducer**: Complex state logic, predictable updates, debugging with Redux DevTools pattern
- **useMemo & useCallback**: Performance optimization, when to use (and when NOT to—many over-optimize)
- **useRef**: DOM access, mutable values, forwarding refs
- **Custom Hooks**: Composition, reusability, logic extraction—be ready to write one in an interview

## **Performance Optimization**

- **React.memo**: When component re-renders, shallow vs deep comparison
- **Code Splitting & Lazy Loading**: React.lazy, Suspense, bundle impact
- **Profiler & DevTools**: How to identify bottlenecks, what metrics matter
- **Key Prop**: Why it matters in lists, reconciliation impact

## **Advanced Patterns**

- **Render Props**: Alternative to HOCs, trade-offs
- **Higher-Order Components (HOCs)**: Cross-cutting concerns, when to use vs hooks
- **Compound Components**: Building flexible, composable component APIs
- **Provider Pattern**: Global state management without Redux

## **State Management Scaling**

- **Context vs Redux vs Zustand**: When each makes sense, trade-offs
- **Derived State**: Why storing computed values is often wrong
- **Lifting State Up**: When and how, performance implications

## **Testing & Debugging**

- **Testing Library**: Unit testing components (React Testing Library > Enzyme now)
- **Mocking & Isolation**: Testing hooks, async behavior
- **Debugging**: Chrome DevTools, React DevTools profiler, common bugs

## **Common Pitfalls (Interviewers Love These)**

1. **Stale Closures**: useEffect dependencies, useCallback gotchas
2. **Infinite Loops**: Missing dependencies, improper effect cleanup
3. **Memory Leaks**: Subscriptions, timers not cleaned up
4. **Reconciliation Mistakes**: Key prop misuse, unnecessary re-renders
5. **Over-engineering**: Using Context/Redux when local state suffices

## **Questions to Prepare For**

Be ready to discuss:

- **"Walk me through a complex component you've built"** → Explain state architecture, optimization choices
- **"How would you optimize this component?"** → Profiling methodology, identifying real bottlenecks
- **"Explain the dependency array in useEffect"** → Deep understanding required
- **"When would you use Context vs Redux?"** → Trade-off analysis
- **"How does React reconciliation work?"** → Virtual DOM, keys, performance implications
- **"Write a custom hook that..."** → Usually something with state + side effects

## **Bonus: What You Should NOT Over-Emphasize**

- Class components (except for error boundaries, which are still class-only)
- Lifecycle methods (useEffect is the modern equivalent)
- PropTypes (TypeScript has largely replaced it for typed code)

## **Quick Prep Strategy**

1. **Review your own code**: Be ready to explain architectural decisions in projects you've built
2. **Hands-on**: Write a small component with hooks, custom hooks, and performance optimization
3. **Know the "why"**: Don't just memorize—understand when and why to use each pattern
4. **Be honest**: If you haven't used something recently, say so but explain how you'd approach it

Given your 10 years and senior title, interviewers will dig into _decisions and trade-offs_, not just syntax. Focus on demonstrating strong architectural thinking and practical problem-solving.

Any specific React concepts you'd like me to dive deeper into, or would you like help prepping for a particular company's technical round?
