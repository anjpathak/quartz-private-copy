---
{"publish":true,"title":"Tabs LLD","created":"2025-12-21T09:39:43.000Z","modified":"2026-08-16T11:31:25.525Z"}
---

## R : Requirement

```
Functional
	Number of tabs fixed or Dynamic (means coming from an API response in the form of
	some array like [{title: tab1, content: content, disabled?: flase}].........)
	Content Types : Simple Text Panels or complex components
	Selection Behaviour: Single or Multi-Tab 
Non - functional
	Aria Roles
	Key Board Nav
	Responsiveness / Device Suport
USE SCALE :
	Scope: Default Active Tab?
	Cnstraints : Max Tabs??
	Assumptions : Controlled vs Uncontrolled
	Limitations : no Drag-Drop
	Edge Cases : Empty Tabs, Lazy-loading for Heavy Tabs, Rapid Tab Switches,
	
```

## A : Architecture

```
Outline high-level Components : Tabs (parent: manages state), TabList (renders Tab
								buttons), TabPanels (renders content), Tab (button),
								TabPanel (content container)
Define Data Flow : state in Tabs → props to children; events bubble up
					(onTabSelect(index)). Consider patterns: Context API or Compound
					Components for flexibility; hooks like useTabs() for encapsulation
```

## D : Data Model

```
Define core entities: tabs array (server/client data: {id: string, label: string, disabled?: boolean, content?: ReactNode}); uiState (activeIndex: number, previousIndex?: number for animations). Client-only: loading states per tab, error flags. Persistence: none typically, but localStorage for last active tab. Map to React: useState<TabsState> or Zustand slice.
```

## I : Interface

```
Specify APIs: Props for Tabs ({children: Tab[], defaultIndex?: number, onChange?: (index) => void, orientation?: 'horizontal'|'vertical'}); Tab({label, disabled}); internal hooks (useTabsState(): {activeIndex, setActiveIndex, Tab components}). Events: keyboard (ArrowLeft/Right, Home/End), mouse (click/focus). Server APIs if async: fetchTabContent(tabId). Error responses: fallback UI.
```

## O : Optimisations

```
Address perf: lazy-load TabPanel content (Suspense/lazy), memoize TabList to avoid re-renders, virtualization if 100+ tabs. UX: focus management (trap focus in active panel?), animations (framer-motion for smooth transitions), a11y (role="tablist", aria-selected/expanded, roving tabindex). Trade-offs: controlled (parent state) vs uncontrolled (internal state); test scenarios (concurrent programmatic + user changes, slow networks). Dive into React specifics: useCallback for handlers, React.memo for Tab.
```

# SIMULATION

**Jack (Interviewer):** Alright, Rose, let's dive into this design exercise. I want you to implement a Tabs component in React.js. Think about building a reusable, accessible Tabs system that can handle dynamic tabs—meaning the tabs come from an array of data—lazy loading for the content in each tab panel, and full keyboard navigation support. This is the kind of component you'd see in a dashboard or settings page in our product. Walk me through your thought process step by step using a structured framework like RADIO. Take your time, and feel free to ask clarifying questions as we go.

**Rose (Candidate):** Absolutely, Jack, thanks for the clear prompt. This sounds like a great real-world component for a product dashboard. Since we're aiming for a Frontend Architect role, I'll structure my approach using the RADIO framework—Requirements, Architecture, Data Model, Interface, and Optimizations—to make sure we're aligned before jumping into any code. That way, we build something scalable and production-ready.

To start with **Requirements Exploration**, I always like to clarify the functional and non-functional details upfront. Could you help me understand the core scope a bit more? For instance, are the tabs fixed in number, or should they be fully dynamic, like coming from an API response as an array of objects? Does it need to support single tab selection only, or could there be multi-select scenarios down the line? What kinds of content might go inside the tab panels—simple text, forms with inputs, heavy charts like those from Recharts, or even embedded iframes?

On the non-functional side, what's the performance expectation? For example, should it handle 20-50 tabs smoothly without lag, especially on mobile devices? Do we need full responsiveness, like swipe gestures on touch devices? Accessibility is crucial—full ARIA compliance and keyboard navigation, right? Also, lazy loading makes sense for perf, but should that include async content fetching per tab? What's the default behavior for the active tab—first one by default? And for edge cases, how should it handle an empty tabs list, switching to a disabled tab, or programmatic changes from a parent component, like via a ref? Any browser constraints, like modern browsers only, or IE11 support?[](https://www.greatfrontend.com/front-end-system-design-playbook/framework)​

**Jack:** Excellent, Rose—love that you're using RADIO right off the bat. It shows you've got a systematic approach, which is key for architect-level thinking. Let me clarify those points to keep us moving.

Yes, dynamic tabs from props—expect an array like \[{label: 'Dashboard', content: some React node, disabled: false}]. Single selection only for now; no multi-select. Content can be anything—forms, charts, tables—so it needs to be flexible for complex React children. Performance-wise, aim for smooth rendering with 20-50 tabs max; lazy-load panels so only the active one (and maybe prefetch neighbors) loads initially. Fully responsive: horizontal on desktop, stack or swipe on mobile. A11y is non-negotiable—WCAG 2.1 AA, full keyboard nav with arrows, home/end, etc.

Default to index 0 if not specified. Edge cases: show empty state if no tabs; if active tab is disabled, skip to next valid one; support programmatic changes via a ref or callback. React 18+, modern browsers (Chrome 90+, Firefox 90+, Safari 14+). No offline or persistence needed. Primarily controlled component from parent state, but fallback to uncontrolled with defaultIndex prop. Orientation horizontal by default, but vertical option for sidebars. Sound good? What's your next step in the framework?

**Rose:** Perfect, Jack—that gives me a solid foundation. Assumptions noted: controlled primary with onChange callback, out-of-scope items like drag-and-drop reordering or nested tabs for now. Quick summary: dynamic single-select tabs, lazy complex content, responsive a11y-focused, perf-optimized for dozens of tabs.

Moving to **Architecture**. At a high level, I'd structure this as a compound component system: a `<Tabs>` parent that manages the overall state and provides context to its children—`<TabList>` for the tab buttons, `<TabPanels>` for the content areas, individual `<Tab>` for each button, and `<TabPanel>` for content. This pattern, inspired by libraries like Headless UI or Radix, keeps it flexible—users can compose like `<Tabs><Tab label="One" /><TabPanel>Content</TabPanel></Tabs>`.

Data flow would be unidirectional: parent state (activeIndex) flows down via Context to children; events like tab clicks bubble up via callbacks. I'd encapsulate logic in a custom `useTabs` hook for reusability. For rendering, `<TabList>` maps over tabs rendering `<Tab>`s, and `<TabPanels>` conditionally renders only the active `<TabPanel>` with lazy loading. Quick mental diagram: Tabs (Context.Provider) wraps TabList (role="tablist") parallel to TabPanels (only active visible). Does this architecture align with what you have in mind, or should I adjust for something like Redux integration?

**Jack:** That's a strong architecture, Rose—compound components with Context is spot-on for React flexibility and avoids prop drilling. Yes, align perfectly; no Redux needed here since it's a self-contained UI primitive. The unidirectional flow and lazy panels sound right. One addition: support for badges or counts on tabs, like unread notifications—maybe optional per-tab prop. Now, let's drill into the **Data Model**. What do the entities look like? Define the shapes precisely.

**Rose:** Great feedback on badges—I'll include that as optional. For **Data Model**, let's define the core entities clearly.

The primary data comes from props: `tabs: Array<TabData>` where `interface TabData { id: string; label: string; disabled?: boolean; badge?: number; content: React.ReactNode; }`. IDs ensure uniqueness, falling back to generated useId() if missing. Client-side UI state managed internally or by parent: `{ activeIndex: number; previousIndex?: number; isTransitioning: boolean; perTabStates?: { loading: boolean; error?: string }[] }` for lazy/async handling.

No server persistence needed, but a hook could sync last activeIndex to localStorage. In React terms, I'd use `useReducer` for state transitions: actions like `{ type: 'SET_ACTIVE', payload: index }`, `{ type: 'LOAD_CONTENT', payload: { index, status } }`. This keeps it predictable and testable. For scaling, it maps nicely to Zustand slices if embedded in a larger app. Does this state shape cover the dynamic/lazy needs, or do we need more like tab order metadata?

**Jack:** Spot-on data model—clean shapes, reducer for complex state is architect-thinking. Badge fits perfectly. Per-tab loading states are crucial for lazy content. Good call on useId fallback. Next up: **Interface Definition**. Lay out the exact props, hooks, and events. Be precise—interviewer loves seeing TypeScript interfaces.

**Rose:** Thanks, Jack. Onto **Interface Definition**. Here's the public API, TypeScript-typed for safety:

typescript

```
interface TabsProps {
  tabs?: TabData[]; // Optional if using compound children
  activeIndex: number;
  onChange: (index: number) => void;
  orientation?: 'horizontal' | 'vertical';
  className?: string;
}

interface TabProps {
  label: string;
  disabled?: boolean;
  badge?: number;
}

// Internal Context hook for children:
interface TabsContextValue {
  activeIndex: number;
  selectTab: (index: number) => void;
  orientation: 'horizontal' | 'vertical';
  tabsLength: number;
}

```

Usage example: `<Tabs activeIndex={state.index} onChange={setIndex}><Tab label="Dashboard" /><TabPanel>Heavy chart</TabPanel></Tabs>`. Events: `onClick`/`onKeyDown` on Tab—arrows cycle valid tabs (skip disabled), Home/End jump ends, Enter selects. For panels: auto-links via `aria-controls`/`aria-labelledby`. Async APIs if needed: internal `preloadTab(index)` prefetch. Keeps params minimal (3-5 max), with fallbacks like "Loading..." spinner. Solid?

**Jack:** Beautifully defined interfaces—TypeScript precision shines, and Context hook enables great DX. Love the aria linking for a11y. Now, **Optimizations**—this is where architects differentiate. Hit perf, UX, a11y, trade-offs. Code sketch if ready, and validate with scenarios.

**Rose:** **Optimizations** time—my favorite part. Performance: Wrap `<TabPanel>` in `React.lazy` + `Suspense` for code-split content; `React.memo(Tab)` + `useCallback(selectTab)` prevent needless re-renders; virtualize `<TabList>` with `react-window` if >50 tabs. Prefetch adjacent tabs via `useEffect`.

A11y/UX: `role="tablist"` on list, `aria-selected="true"`/`aria-expanded` on active, roving `tabindex=0/-1` for keyboard (only active focusable). Smooth CSS transitions (fade/slide) or Framer Motion; focus trap in active panel via `useFocusTrap`. Mobile: touch swipe with `react-swipeable`.

Trade-offs: Controlled (parent owns state, great for forms) vs uncontrolled (internal state, simpler embeds)—props handle both. Context scale: fine for shallow trees; prop drill or render props if deep.

Quick code sketch for core:

jsx

```
const TabsContext = createContext(null);

function Tabs({ activeIndex, onChange, orientation = 'horizontal', children }) {
  const selectTab = useCallback((index) => {
    if (!tabs[index]?.disabled) onChange(index);
  }, [onChange]);
  
  const value = { activeIndex, selectTab, orientation };
  return <TabsContext.Provider value={value}>{children}</TabsContext.Provider>;
}

function Tab({ label, disabled, badge, index }) { // index from parent map or Context
  const { activeIndex, selectTab } = useContext(TabsContext);
  const isActive = activeIndex === index;
  return (
    <button
      role="tab"
      aria-selected={isActive}
      tabIndex={isActive ? 0 : -1}
      onClick={() => selectTab(index)}
      disabled={disabled}
    >
      {label} {badge && <span>{badge}</span>}
    </button>
  );
}

```

Happy path validation: Click tab 2 (valid) → selectTab(2) → onChange → re-render, lazy-load Panel 2. Edge: Arrow on disabled → skip to 3; empty tabs → "No tabs available". Concurrent: Programmatic set + user click → last wins, with debounce if needed. Tests: Jest/RTL for hooks (selectTab skips disabled), Cypress for e2e flows. Thoughts or scenarios to stress-test?

**Jack:** Outstanding, Rose—this is architect caliber. Perf/a11y trade-offs nailed, code is clean and extensible. Badge integration seamless, lazy perfect for charts. Critique: For dynamic tabs without compound children, how do you map index in Tab? Context needs tabs array? Nested tabs?

**Rose:** Sharp catch, Jack— for non-compound (tabs prop), Tabs renders TabList internally, passing index via render prop or Context with tabsLength. Context adds `getTabIndex(label)` or assumes sequential. Nested: Composition—`<TabPanel><Tabs>...</Tabs></TabPanel>` with isolated Contexts. I'd prototype that next. Any final tweaks before "code complete"?[](https://www.greatfrontend.com/front-end-system-design-playbook/framework)​

# Commond Problems

| Problem                 | Impact                                                                                                                                          | Solution                                                                                      |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| Prop drilling for state | Boilerplate, inflexible [](https://dev.to/bnevilleoneill/guide-to-react-compound-components-1237)​                                              | Context API in compound pattern [](https://www.patterns.dev/react/compound-pattern/)​         |
| Focus management        | Poor keyboard UX [](https://web.dev/articles/building/a-tabs-component)​                                                                        | `useEffect` + `ref` to focus active tab, `tabIndex` control                                   |
| Content re-mounting     | State loss, perf hit [](https://stackoverflow.com/questions/76035035/how-to-allow-multiple-tabs-of-the-same-component-while-keeping-its-state)​ | Conditional render with keys, cache panels via `Map`                                          |
| SSR hydration mismatch  | Flicker, errors                                                                                                                                 | `useEffect` for client-only state init [](https://sandroroth.com/blog/react-tabs-component/)​ |
| Large tab counts        | Slow renders                                                                                                                                    | Virtualized lists (`react-window`) for TabList                                                |
