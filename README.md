# 🚀 JavaScript & React Interview Questions - PART 2

**Continuation of Complete Interview Guide**

---

## Basic React Continued (Questions 106-130)

### Q106. What is useEffect hook?
**Hindi:** useEffect hook side effects handle karne ke liye use hota hai.

**Answer:**
`useEffect` lets you perform side effects in functional components (API calls, subscriptions, DOM manipulation).

**Syntax:**
```jsx
useEffect(() => {
  // Effect code
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

**Example:**
```jsx
import { useState, useEffect } from 'react';

// 1. Run on every render
function Component() {
  useEffect(() => {
    console.log('Runs on every render');
  });
}

// 2. Run once on mount (empty dependency array)
function Component() {
  useEffect(() => {
    console.log('Runs once on mount');
  }, []);
}

// 3. Run when specific values change
function Component() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]); // Runs when count changes
}

// 4. Cleanup function
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // Cleanup - runs before next effect and on unmount
    return () => {
      clearInterval(interval);
      console.log('Timer cleaned up');
    };
  }, []);
  
  return <div>Seconds: {seconds}</div>;
}

// 5. Fetch data
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    setLoading(true);
    
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(error => {
        console.error(error);
        setLoading(false);
      });
  }, [userId]); // Re-fetch when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 6. Multiple useEffects
function Component() {
  useEffect(() => {
    console.log('Effect 1');
  }, []);
  
  useEffect(() => {
    console.log('Effect 2');
  }, []);
  
  // Effects run in order they're defined
}

// 7. Document title update
function PageTitle() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Clicked {count} times
    </button>
  );
}

// 8. Event listeners
function WindowSize() {
  const [width, setWidth] = useState(window.innerWidth);
  
  useEffect(() => {
    const handleResize = () => setWidth(window.innerWidth);
    
    window.addEventListener('resize', handleResize);
    
    // Cleanup
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <div>Window width: {width}px</div>;
}
```

---

### Q107. What is props drilling?
**Hindi:** Props drilling ka matlab hai props ko multiple levels tak pass karna.

**Answer:**
Props drilling is passing props through multiple component layers to reach a deeply nested component.

**Problem:**
```jsx
// Grandparent
function App() {
  const user = { name: "John", role: "Admin" };
  return <Parent user={user} />;
}

// Parent (doesn't use user, just passes it)
function Parent({ user }) {
  return <Child user={user} />;
}

// Child (doesn't use user, just passes it)
function Child({ user }) {
  return <GrandChild user={user} />;
}

// Finally uses it
function GrandChild({ user }) {
  return <div>Hello, {user.name}</div>;
}
```

**Solutions:**

**1. Context API**
```jsx
import { createContext, useContext } from 'react';

const UserContext = createContext();

function App() {
  const user = { name: "John", role: "Admin" };
  
  return (
    <UserContext.Provider value={user}>
      <Parent />
    </UserContext.Provider>
  );
}

function Parent() {
  return <Child />; // No props!
}

function Child() {
  return <GrandChild />; // No props!
}

function GrandChild() {
  const user = useContext(UserContext); // Direct access!
  return <div>Hello, {user.name}</div>;
}
```

**2. Component Composition**
```jsx
function App() {
  const user = { name: "John", role: "Admin" };
  
  return (
    <Parent>
      <Child>
        <GrandChild user={user} />
      </Child>
    </Parent>
  );
}
```

---

### Q108. What is Context API?
**Hindi:** Context API global state manage karne ka tarika hai bina props drilling ke.

**Answer:**
Context provides a way to pass data through component tree without passing props manually at every level.

**Example:**
```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create Context
const ThemeContext = createContext();

// 2. Create Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Create Custom Hook (optional but recommended)
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// 4. Use in Components
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
      <Footer />
    </ThemeProvider>
  );
}

function Header() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <header className={theme}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} mode
      </button>
    </header>
  );
}

function Main() {
  const { theme } = useTheme();
  return <main className={theme}>Content...</main>;
}

// Real-world example: Auth Context
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Check if user is logged in
    const token = localStorage.getItem('token');
    if (token) {
      fetchUser(token).then(setUser);
    }
    setLoading(false);
  }, []);
  
  const login = async (email, password) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify({ email, password })
    });
    const data = await response.json();
    setUser(data.user);
    localStorage.setItem('token', data.token);
  };
  
  const logout = () => {
    setUser(null);
    localStorage.removeItem('token');
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout, loading }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

// Usage
function Dashboard() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <h1>Welcome, {user.name}!</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

---

### Q109. What are React Hooks?
**Hindi:** Hooks functional components mein state aur lifecycle features use karne dete hain.

**Answer:**
Hooks are functions that let you use state and other React features in functional components.

**Built-in Hooks:**

**1. useState** - Add state
**2. useEffect** - Side effects
**3. useContext** - Access context
**4. useReducer** - Complex state logic
**5. useCallback** - Memoize functions
**6. useMemo** - Memoize values
**7. useRef** - Reference values/DOM
**8. useLayoutEffect** - Sync layout effects
**9. useImperativeHandle** - Customize ref
**10. useDebugValue** - Debug custom hooks

**Example:**
```jsx
import {
  useState,
  useEffect,
  useContext,
  useReducer,
  useCallback,
  useMemo,
  useRef
} from 'react';

function ExampleComponent() {
  // useState
  const [count, setCount] = useState(0);
  
  // useEffect
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);
  
  // useContext
  const theme = useContext(ThemeContext);
  
  // useRef
  const inputRef = useRef(null);
  const focusInput = () => inputRef.current.focus();
  
  // useCallback
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []);
  
  // useMemo
  const expensiveValue = useMemo(() => {
    return computeExpensiveValue(count);
  }, [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleClick}>Increment</button>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}
```

**Rules of Hooks:**
1. Only call hooks at the top level (not in loops/conditions)
2. Only call hooks from React functions
3. Custom hooks start with "use"

---

### Q110. What is useRef?
**Hindi:** useRef DOM elements ko access karne ya values ko persist karne ke liye use hota hai.

**Answer:**
`useRef` creates a mutable reference that persists across renders without causing re-renders.

**Use Cases:**
1. Accessing DOM elements
2. Storing mutable values
3. Keeping previous values
4. Avoiding re-renders

**Example:**
```jsx
import { useRef, useState, useEffect } from 'react';

// 1. Access DOM elements
function FocusInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} />
      <button onClick={focusInput}>Focus Input</button>
    </div>
  );
}

// 2. Store mutable value (doesn't cause re-render)
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };
  
  const stop = () => {
    clearInterval(intervalRef.current);
  };
  
  useEffect(() => {
    return () => clearInterval(intervalRef.current);
  }, []);
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}

// 3. Keep previous value
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}

// 4. Scroll to element
function ScrollToTop() {
  const topRef = useRef(null);
  
  const scrollToTop = () => {
    topRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  return (
    <div>
      <div ref={topRef}>Top of page</div>
      {/* Long content */}
      <button onClick={scrollToTop}>Scroll to Top</button>
    </div>
  );
}

// 5. Measure element
function MeasureElement() {
  const divRef = useRef(null);
  const [dimensions, setDimensions] = useState({});
  
  useEffect(() => {
    if (divRef.current) {
      const { width, height } = divRef.current.getBoundingClientRect();
      setDimensions({ width, height });
    }
  }, []);
  
  return (
    <div ref={divRef}>
      Size: {dimensions.width} x {dimensions.height}
    </div>
  );
}
```

---

### Q111-130: Quick Fire Basic React

**Q111. What is Virtual DOM?**
Virtual DOM is a lightweight copy of the actual DOM. React updates Virtual DOM first, then efficiently updates real DOM.

**Q112. What is reconciliation?**
Process of updating the DOM by comparing Virtual DOM with previous version and applying minimal changes.

**Q113. What are keys in React?**
```jsx
// Keys help React identify which items changed
const items = ['Apple', 'Banana', 'Orange'];

// Good
{items.map((item, index) => (
  <li key={item}>{item}</li>
))}

// Bad (avoid index as key if list can change)
{items.map((item, index) => (
  <li key={index}>{item}</li>
))}
```

**Q114. What is React.Fragment?**
```jsx
// Avoids extra div wrapper
return (
  <React.Fragment>
    <h1>Title</h1>
    <p>Content</p>
  </React.Fragment>
);

// Short syntax
return (
  <>
    <h1>Title</h1>
    <p>Content</p>
  </>
);
```

**Q115. What are controlled components?**
```jsx
// Controlled - React controls input value
function Form() {
  const [name, setName] = useState('');
  
  return (
    <input 
      value={name}
      onChange={(e) => setName(e.target.value)}
    />
  );
}
```

**Q116. What are uncontrolled components?**
```jsx
// Uncontrolled - DOM controls input value
function Form() {
  const inputRef = useRef();
  
  const handleSubmit = () => {
    console.log(inputRef.current.value);
  };
  
  return <input ref={inputRef} />;
}
```

**Q117. What is conditional rendering?**
```jsx
function Greeting({ isLoggedIn }) {
  // If-else
  if (isLoggedIn) {
    return <h1>Welcome back!</h1>;
  }
  return <h1>Please sign in</h1>;
  
  // Ternary
  return isLoggedIn ? <h1>Welcome!</h1> : <h1>Sign in</h1>;
  
  // && operator
  return isLoggedIn && <h1>Welcome!</h1>;
}
```

**Q118. What is list rendering?**
```jsx
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>{todo.text}</li>
      ))}
    </ul>
  );
}
```

**Q119. How to handle events?**
```jsx
function Button() {
  const handleClick = (e) => {
    e.preventDefault();
    console.log('Clicked!');
  };
  
  return <button onClick={handleClick}>Click me</button>;
}
```

**Q120. What is lifting state up?**
Moving state to common parent component to share between children.

```jsx
function Parent() {
  const [value, setValue] = useState('');
  
  return (
    <div>
      <Child1 value={value} onChange={setValue} />
      <Child2 value={value} />
    </div>
  );
}
```

---

## Intermediate React (Questions 131-160)

### Q131. What is useReducer?
**Hindi:** useReducer complex state logic handle karne ke liye use hota hai.

**Answer:**
`useReducer` is an alternative to `useState` for managing complex state logic.

**When to use:**
- Multiple sub-values in state
- Complex state transitions
- Next state depends on previous
- Redux-like pattern

**Example:**
```jsx
import { useReducer } from 'react';

// 1. Define reducer
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      throw new Error('Unknown action');
  }
}

// 2. Use in component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </div>
  );
}

// Real-world example: Todo List
const initialState = {
  todos: [],
  filter: 'all'
};

function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [...state.todos, {
          id: Date.now(),
          text: action.payload,
          completed: false
        }]
      };
      
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
      
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
      
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload
      };
      
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');
  
  const addTodo = () => {
    if (input.trim()) {
      dispatch({ type: 'ADD_TODO', payload: input });
      setInput('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'completed') return todo.completed;
    if (state.filter === 'active') return !todo.completed;
    return true;
  });
  
  return (
    <div>
      <input 
        value={input}
        onChange={(e) => setInput(e.target.value)}
      />
      <button onClick={addTodo}>Add</button>
      
      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'all' })}>
          All
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'active' })}>
          Active
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: 'completed' })}>
          Completed
        </button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input 
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: todo.id })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: todo.id })}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Q132. What is useCallback?
**Hindi:** useCallback functions ko memoize karta hai re-renders avoid karne ke liye.

**Answer:**
`useCallback` returns a memoized version of callback that only changes if dependencies change.

**When to use:**
- Passing callbacks to optimized child components
- Dependency of useEffect
- Expensive function creation

**Example:**
```jsx
import { useState, useCallback, memo } from 'react';

// Without useCallback - Child re-renders unnecessarily
function Parent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);
  
  // New function created on every render
  const handleClick = () => {
    console.log('Clicked');
  };
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setOther(other + 1)}>Increment Other</button>
      
      {/* Child re-renders even when count changes (not needed) */}
      <Child onClick={handleClick} />
    </div>
  );
}

// With useCallback - Child only re-renders when needed
function Parent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);
  
  // Same function reference unless dependencies change
  const handleClick = useCallback(() => {
    console.log('Clicked', count);
  }, [count]); // Re-create only when count changes
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setOther(other + 1)}>Increment Other</button>
      
      {/* Child doesn't re-render when 'other' changes */}
      <Child onClick={handleClick} />
    </div>
  );
}

// Memoized child component
const Child = memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click me</button>;
});

// Real-world example: Search with debounce
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  // Memoize search function
  const search = useCallback(async (searchTerm) => {
    if (!searchTerm) {
      setResults([]);
      return;
    }
    
    const response = await fetch(`/api/search?q=${searchTerm}`);
    const data = await response.json();
    setResults(data);
  }, []);
  
  // Debounced search
  useEffect(() => {
    const timer = setTimeout(() => {
      search(query);
    }, 500);
    
    return () => clearTimeout(timer);
  }, [query, search]);
  
  return (
    <div>
      <input 
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Q133. What is useMemo?
**Hindi:** useMemo expensive calculations ko memoize karta hai.

**Answer:**
`useMemo` returns a memoized value that only recomputes when dependencies change.

**Example:**
```jsx
import { useState, useMemo } from 'react';

function ExpensiveComponent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);
  
  // Without useMemo - Recalculates on every render (slow!)
  const expensiveValue = calculateExpensiveValue(count);
  
  // With useMemo - Only recalculates when count changes
  const expensiveValue = useMemo(() => {
    console.log('Calculating...');
    return calculateExpensiveValue(count);
  }, [count]);
  
  return (
    <div>
      <p>Expensive Value: {expensiveValue}</p>
      <p>Count: {count}</p>
      <p>Other: {other}</p>
      <button onClick={() => setCount(count + 1)}>Increment Count</button>
      <button onClick={() => setOther(other + 1)}>Increment Other</button>
      {/* When 'other' changes, expensive calculation doesn't run! */}
    </div>
  );
}

function calculateExpensiveValue(num) {
  // Simulate expensive calculation
  let result = 0;
  for (let i = 0; i < 1000000000; i++) {
    result += i;
  }
  return result + num;
}

// Real example: Filtered list
function UserList() {
  const [users, setUsers] = useState([/* large array */]);
  const [filter, setFilter] = useState('');
  const [sortBy, setSortBy] = useState('name');
  
  // Memoize filtered and sorted list
  const filteredUsers = useMemo(() => {
    console.log('Filtering and sorting...');
    
    return users
      .filter(user => 
        user.name.toLowerCase().includes(filter.toLowerCase())
      )
      .sort((a, b) => a[sortBy].localeCompare(b[sortBy]));
  }, [users, filter, sortBy]);
  
  return (
    <div>
      <input 
        value={filter}
        onChange={(e) => setFilter(e.target.value)}
        placeholder="Filter users..."
      />
      <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
        <option value="name">Name</option>
        <option value="email">Email</option>
      </select>
      
      <ul>
        {filteredUsers.map(user => (
          <li key={user.id}>{user.name} - {user.email}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

### Q134. What is React.memo?
**Hindi:** React.memo component ko memoize karta hai unnecessary re-renders avoid karne ke liye.

**Answer:**
`React.memo` is a Higher Order Component that memoizes component output.

**Example:**
```jsx
import { memo } from 'react';

// Without memo - Re-renders even if props don't change
function Child({ name }) {
  console.log('Child rendered');
  return <div>Hello, {name}!</div>;
}

// With memo - Only re-renders if props change
const Child = memo(function Child({ name }) {
  console.log('Child rendered');
  return <div>Hello, {name}!</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      {/* Child doesn't re-render when count changes */}
      <Child name="John" />
    </div>
  );
}

// Custom comparison function
const Child = memo(
  function Child({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    return prevProps.user.id === nextProps.user.id;
  }
);
```

---

### Q135-160: Quick Fire Intermediate React

**Q135. What is lazy loading?**
```jsx
import { lazy, Suspense } from 'react';

const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <HeavyComponent />
    </Suspense>
  );
}
```

**Q136. What is code splitting?**
Breaking bundle into smaller chunks that load on demand.

**Q137. What are Error Boundaries?**
```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.log(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong!</h1>;
    }
    return this.props.children;
  }
}
```

**Q138. What is React Router?**
Library for routing in React applications.
```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users/:id" element={<User />} />
      </Routes>
    </BrowserRouter>
  );
}
```

**Q139. What are custom hooks?**
```jsx
// Custom hook for form handling
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);
  
  const handleChange = (e) => {
    setValues({
      ...values,
      [e.target.name]: e.target.value
    });
  };
  
  const reset = () => setValues(initialValues);
  
  return [values, handleChange, reset];
}

// Usage
function Form() {
  const [values, handleChange, reset] = useForm({
    name: '',
    email: ''
  });
  
  return (
    <form>
      <input name="name" value={values.name} onChange={handleChange} />
      <input name="email" value={values.email} onChange={handleChange} />
      <button type="button" onClick={reset}>Reset</button>
    </form>
  );
}
```

**Q140. What is useLayoutEffect?**
Fires synchronously after DOM mutations, before browser paint.

**Q141-160:** Performance optimization, Redux, state management, testing, etc.

---

## 💻 Coding Challenges

### Challenge 1: Counter with useReducer
```jsx
// Create a counter with increment, decrement, and reset using useReducer
```

### Challenge 2: Todo List
```jsx
// Build a complete todo list with add, delete, toggle, filter
```

### Challenge 3: Custom Hook - useFetch
```jsx
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(url)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [url]);
  
  return { data, loading, error };
}
```

### Challenge 4: Debounce Search
```jsx
// Implement search with debouncing
```

### Challenge 5: Infinite Scroll
```jsx
// Implement infinite scroll with pagination
```

---

## 🎯 Interview Tips

1. **Understand Fundamentals** - Don't just memorize
2. **Practice Coding** - Build real projects
3. **Explain Your Thinking** - Talk through solutions
4. **Ask Questions** - Clarify requirements
5. **Test Your Code** - Think about edge cases

---

**Full guide continues with Advanced React (Q161-200) and more challenges in next file if needed!**
