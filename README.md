# React Hooks

---

## Introduction

React Hooks are functions that let you use state and other React features in functional components. They were introduced in React 16.8 and follow two main rules:
- Only call Hooks at the top level (not inside loops, conditions, or nested functions)
- Only call Hooks from React functions (components or custom hooks)

---

## State Hooks

### useState

**Purpose**: Add state to functional components

**Code Example**:
```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
      <button onClick={() => setCount(prev => prev - 1)}>-</button>
    </div>
  );
}
```

**Description**: Returns a state variable and a function to update it. The initial state is passed as an argument.

**Pros**:
- Simple and intuitive
- Handles primitive and complex state
- Batches multiple setState calls automatically
- Supports functional updates

**Cons**:
- Can lead to stale closures if not used carefully
- Multiple useState calls can make component logic scattered
- Re-renders entire component on state change

**Best Practices**:
- Use functional updates when new state depends on previous state
- Split unrelated state into multiple useState calls
- Use useReducer for complex state logic

---

### useReducer

**Purpose**: Manage complex state logic with a reducer function

**Code Example**:
```jsx
import React, { useReducer } from 'react';

const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return initialState;
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

**Description**: Alternative to useState for complex state logic. Takes a reducer function and initial state.

**Pros**:
- Predictable state updates
- Better for complex state logic
- Easier to test reducer functions
- Can optimize performance with useCallback on dispatch

**Cons**:
- More boilerplate code
- Overkill for simple state
- Requires understanding of reducer pattern

**When to Use**: Complex state logic, multiple sub-values, or when next state depends on previous state.

---

## Context Hooks

### useContext

**Purpose**: Consume context values without nesting

**Code Example**:
```jsx
import React, { createContext, useContext, useState } from 'react';

const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

function ThemedButton() {
  const { theme, setTheme } = useContext(ThemeContext);
  
  return (
    <button 
      style={{ 
        background: theme === 'dark' ? '#333' : '#fff',
        color: theme === 'dark' ? '#fff' : '#333'
      }}
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
    >
      Toggle Theme ({theme})
    </button>
  );
}
```

**Description**: Reads context value from the nearest context provider above in the tree.

**Pros**:
- Eliminates prop drilling
- Clean component tree
- Automatic re-rendering when context changes

**Cons**:
- All consumers re-render when context changes
- Can make components less reusable
- Debugging can be harder

**Best Practices**:
- Split contexts by concern
- Use multiple contexts instead of one large context
- Memoize context values to prevent unnecessary re-renders

---

## Ref Hooks

### useRef

**Purpose**: Create mutable references that persist across renders

**Code Example**:
```jsx
import React, { useRef, useEffect } from 'react';

function TextInput() {
  const inputRef = useRef(null);
  const renderCount = useRef(0);
  
  useEffect(() => {
    renderCount.current += 1;
  });
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" />
      <button onClick={focusInput}>Focus Input</button>
      <p>Render count: {renderCount.current}</p>
    </div>
  );
}
```

**Description**: Returns a mutable ref object with a `.current` property. Doesn't cause re-renders when mutated.

**Pros**:
- Direct DOM access
- Persists values across renders
- Doesn't trigger re-renders
- Useful for storing mutable values

**Cons**:
- Mutating `.current` doesn't trigger re-renders
- Can lead to imperative programming patterns
- Potential memory leaks if not cleaned up

**Common Use Cases**:
- Accessing DOM elements
- Storing mutable values
- Keeping references to intervals/timeouts
- Previous values storage

---

### useImperativeHandle

**Purpose**: Customize the instance value exposed to parent components when using ref

**Code Example**:
```jsx
import React, { forwardRef, useImperativeHandle, useRef } from 'react';

const FancyInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    },
    scrollIntoView: () => {
      inputRef.current.scrollIntoView();
    },
    getValue: () => {
      return inputRef.current.value;
    }
  }));
  
  return <input ref={inputRef} {...props} />;
});

function Parent() {
  const inputRef = useRef();
  
  return (
    <div>
      <FancyInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus</button>
      <button onClick={() => alert(inputRef.current.getValue())}>Get Value</button>
    </div>
  );
}
```

**Description**: Rarely used hook that lets you customize what ref exposes to parent components.

**Pros**:
- Fine-grained control over ref API
- Encapsulation of internal implementation
- Clean parent-child communication

**Cons**:
- Breaks React's declarative paradigm
- Can make components harder to understand
- Rarely needed in most applications

**When to Use**: Building reusable component libraries or when you need specific imperative APIs.

---

## Effect Hooks

### useEffect

**Purpose**: Perform side effects in functional components

**Code Example**:
```jsx
import React, { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        if (!cancelled) {
          setUser(userData);
        }
      } catch (error) {
        if (!cancelled) {
          console.error('Failed to fetch user:', error);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true;
    };
  }, [userId]);
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>Hello, {user.name}!</div>;
}
```

**Description**: Runs side effects after render. Can optionally clean up and specify dependencies.

**Pros**:
- Unified API for lifecycle methods
- Automatic cleanup
- Flexible dependency control
- Can be used multiple times

**Cons**:
- Can cause infinite loops if dependencies are wrong
- Runs after every render by default
- Cleanup timing can be tricky

**Common Patterns**:
- Data fetching
- Setting up subscriptions
- Manually changing DOM
- Cleanup on unmount

---

### useLayoutEffect

**Purpose**: Synchronous version of useEffect that fires before browser paint

**Code Example**:
```jsx
import React, { useState, useLayoutEffect, useRef } from 'react';

function MeasureExample() {
  const [height, setHeight] = useState(0);
  const divRef = useRef();
  
  useLayoutEffect(() => {
    if (divRef.current) {
      setHeight(divRef.current.offsetHeight);
    }
  });
  
  return (
    <div>
      <div ref={divRef} style={{ padding: '20px', background: '#f0f0f0' }}>
        This div's height is: {height}px
      </div>
    </div>
  );
}
```

**Description**: Identical to useEffect but fires synchronously before the browser paints.

**Pros**:
- Prevents visual flicker
- Synchronous execution
- Useful for DOM measurements

**Cons**:
- Blocks visual updates
- Can hurt performance
- Same dependency pitfalls as useEffect

**When to Use**: DOM measurements, preventing visual flicker, synchronous DOM mutations.

---

## Performance Hooks

### useMemo

**Purpose**: Memoize expensive calculations between renders

**Code Example**:
```jsx
import React, { useState, useMemo } from 'react';

function ExpensiveComponent({ items, filter }) {
  const [count, setCount] = useState(0);
  
  const filteredItems = useMemo(() => {
    console.log('Filtering items...');
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);
  
  const expensiveValue = useMemo(() => {
    console.log('Calculating expensive value...');
    return filteredItems.reduce((sum, item) => sum + item.price, 0);
  }, [filteredItems]);
  
  return (
    <div>
      <p>Total: ${expensiveValue}</p>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <ul>
        {filteredItems.map(item => (
          <li key={item.id}>{item.name} - ${item.price}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Description**: Returns a memoized value that only recalculates when dependencies change.

**Pros**:
- Prevents expensive recalculations
- Improves performance for complex computations
- Referential equality for objects/arrays

**Cons**:
- Memory overhead
- Can actually hurt performance if overused
- Adds complexity

**When to Use**: Expensive calculations, creating objects/arrays passed to child components, breaking referential equality chains.

---

### useCallback

**Purpose**: Memoize callback functions between renders

**Code Example**:
```jsx
import React, { useState, useCallback, memo } from 'react';

const ChildComponent = memo(({ onClick, name }) => {
  console.log(`Rendering ${name}`);
  return <button onClick={onClick}>Click {name}</button>;
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  const handleClick1 = useCallback(() => {
    console.log('Button 1 clicked');
  }, []);
  
  const handleClick2 = useCallback(() => {
    console.log(`Button 2 clicked with count: ${count}`);
  }, [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)}
        placeholder="Type to trigger re-render"
      />
      <ChildComponent onClick={handleClick1} name="1" />
      <ChildComponent onClick={handleClick2} name="2" />
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**Description**: Returns a memoized callback function that only changes when dependencies change.

**Pros**:
- Prevents unnecessary child re-renders
- Stable function references
- Optimizes performance with React.memo

**Cons**:
- Memory overhead
- Can be overused
- Adds complexity

**When to Use**: Passing callbacks to memoized child components, dependency arrays of other hooks, expensive event handlers.

---

## Transition Hooks

### useTransition

**Purpose**: Mark state updates as non-urgent transitions

**Code Example**:
```jsx
import React, { useState, useTransition } from 'react';

function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isPending, startTransition] = useTransition();
  
  const handleSearch = (value) => {
    setQuery(value);
    
    startTransition(() => {
      // This expensive operation won't block the input
      const filtered = heavySearchOperation(value);
      setResults(filtered);
    });
  };
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => handleSearch(e.target.value)}
        placeholder="Search..."
      />
      {isPending && <div>Searching...</div>}
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
}

function heavySearchOperation(query) {
  // Simulate expensive operation
  const items = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `Item ${i}`
  }));
  
  return items.filter(item => 
    item.name.toLowerCase().includes(query.toLowerCase())
  );
}
```

**Description**: Allows marking updates as transitions, keeping the UI responsive during expensive operations.

**Pros**:
- Keeps UI responsive
- Prevents blocking urgent updates
- Better user experience

**Cons**:
- Adds complexity
- Not always necessary
- Can delay important updates

**When to Use**: Expensive filtering, search operations, large list updates, navigation.

---

### useDeferredValue

**Purpose**: Defer updates to non-urgent values

**Code Example**:
```jsx
import React, { useState, useDeferredValue, memo } from 'react';

const ExpensiveList = memo(({ query }) => {
  const items = Array.from({ length: 10000 }, (_, i) => `Item ${i}`);
  const filtered = items.filter(item => 
    item.toLowerCase().includes(query.toLowerCase())
  );
  
  return (
    <ul>
      {filtered.slice(0, 100).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
});

function DeferredSearchApp() {
  const [query, setQuery] = useState('');
  const deferredQuery = useDeferredValue(query);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ExpensiveList query={deferredQuery} />
    </div>
  );
}
```

**Description**: Returns a deferred version of a value that lags behind during rapid updates.

**Pros**:
- Keeps input responsive
- Simpler than useTransition for some cases
- Automatic optimization

**Cons**:
- Can show stale data
- Adds complexity
- Not always predictable

**When to Use**: Search inputs, filtering, expensive renders that depend on user input.

---

## Other Hooks

### useId

**Purpose**: Generate unique IDs for accessibility attributes

**Code Example**:
```jsx
import React, { useId } from 'react';

function FormField({ label, type = 'text' }) {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} type={type} />
    </div>
  );
}

function LoginForm() {
  return (
    <form>
      <FormField label="Username" />
      <FormField label="Password" type="password" />
    </form>
  );
}
```

**Description**: Generates unique IDs that are consistent across server and client renders.

**Pros**:
- Accessibility compliance
- SSR compatible
- Unique across component instances

**Cons**:
- Limited use cases
- Only for accessibility attributes
- Not human-readable

**When to Use**: Form labels, ARIA attributes, accessibility features.

---

### useDebugValue

**Purpose**: Display labels for custom hooks in React DevTools

**Code Example**:
```jsx
import React, { useState, useEffect, useDebugValue } from 'react';

function useCounter(initialValue = 0) {
  const [count, setCount] = useState(initialValue);
  
  useDebugValue(count > 5 ? 'High' : 'Low');
  
  return [count, setCount];
}

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  useDebugValue(isOnline ? 'Online' : 'Offline');
  
  return isOnline;
}
```

**Description**: Only used for debugging custom hooks in React DevTools.

**Pros**:
- Better debugging experience
- No performance impact in production
- Helpful for complex custom hooks

**Cons**:
- Development-only feature
- Not useful for end users
- Can clutter DevTools if overused

**When to Use**: Complex custom hooks, debugging purposes, development tools.

---

### useSyncExternalStore

**Purpose**: Subscribe to external stores safely

**Code Example**:
```jsx
import React, { useSyncExternalStore } from 'react';

// External store example
const store = {
  state: { count: 0 },
  listeners: new Set(),
  
  getState() {
    return this.state;
  },
  
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener);
  },
  
  increment() {
    this.state = { count: this.state.count + 1 };
    this.listeners.forEach(listener => listener());
  }
};

function useStore() {
  return useSyncExternalStore(
    store.subscribe.bind(store),
    store.getState.bind(store)
  );
}

function Counter() {
  const state = useStore();
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => store.increment()}>Increment</button>
    </div>
  );
}
```

**Description**: Safely subscribes to external stores with proper SSR handling.

**Pros**:
- Safe external store integration
- SSR compatible
- Prevents tearing

**Cons**:
- Complex API
- Limited use cases
- Requires external store implementation

**When to Use**: Integrating with external state management libraries, browser APIs, global stores.

---

## Hook Categories Flowchart

```
React Hooks
├── State Management
│   ├── useState (simple state)
│   └── useReducer (complex state logic)
│
├── Context & Data Flow
│   └── useContext (consume context)
│
├── Side Effects
│   ├── useEffect (async effects)
│   └── useLayoutEffect (sync effects)
│
├── Performance Optimization
│   ├── useMemo (memoize values)
│   ├── useCallback (memoize functions)
│   ├── useTransition (non-urgent updates)
│   └── useDeferredValue (defer values)
│
├── Refs & DOM
│   ├── useRef (mutable refs)
│   └── useImperativeHandle (custom ref API)
│
├── External Integration
│   └── useSyncExternalStore (external stores)
│
└── Development & Utilities
    ├── useId (unique IDs)
    └── useDebugValue (debug labels)
```

## Hook Selection Guide

**For State Management:**
- Simple values → `useState`
- Complex state logic → `useReducer`
- Global state → `useContext`

**For Side Effects:**
- Data fetching, subscriptions → `useEffect`
- DOM measurements, prevent flicker → `useLayoutEffect`

**For Performance:**
- Expensive calculations → `useMemo`
- Stable callback references → `useCallback`
- Non-urgent updates → `useTransition`
- Deferred values → `useDeferredValue`

**For DOM & Refs:**
- DOM access, mutable values → `useRef`
- Custom component API → `useImperativeHandle`

**For Development:**
- Accessibility IDs → `useId`
- Debug custom hooks → `useDebugValue`
- External stores → `useSyncExternalStore`

## Best Practices Summary

1. **Follow the Rules of Hooks** - Only call at top level and from React functions
2. **Use the Right Hook** - Choose based on your specific use case
3. **Optimize Wisely** - Don't overuse performance hooks
4. **Handle Dependencies** - Be careful with dependency arrays
5. **Clean Up Effects** - Always clean up subscriptions and intervals
6. **Split Concerns** - Use multiple hooks for different concerns
7. **Test Hook Logic** - Extract complex logic into custom hooks for testing
