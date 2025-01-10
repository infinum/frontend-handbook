*Guides are not rules and should not be followed blindly. Use your head and think.*

### Split components in smaller ones to increase reusability

Splitting components into smaller, single-responsibility components improves:

* Readability: Easier to understand and maintain each piece.
* Reusability: Smaller pieces can be reused across different parts of your application.
* Bundle Size: Potentially reduces bundle size via code splitting and tree-shaking.

```jsx
// BAD
const UserProfile = ({ user }) => (
  <div>
    <div>
      <img src={avatar} alt="User Avatar" />
      <h1>{name}</h1>
      <p>{bio}</p>
    </div>
    {user.posts.map(post => <p key={post.id}>{post.content}</p>)}
  </div>
);
```

```jsx
// GOOD
const UserProfileHeader = ({ avatar, name, bio }) => (
  <div>
    <img src={avatar} alt="User Avatar" />
    <h1>{name}</h1>
    <p>{bio}</p>
  </div>
);

const UserPosts = ({ posts }) => posts.map(post => <p key={post.id}>{post.content}</p>);

const UserProfile = ({ user }) => (
  <div>
    <UserProfileHeader
      avatar={user.avatar}
      name={user.name}
      bio={user.bio}
    />
    <UserPosts posts={user.posts} />
  </div>
);
```

**Additional tips**

* Keep each component focused on a single task.
* Use composition over inheritance.
* Avoid massive components that do “everything.”

### Avoid Inline Functions and Objects

Defining functions or object literals inside the component render can trigger unnecessary re-renders because each render creates new references.

```jsx
// BAD
const MyComponent = () => {
  const handleClick = () => {
    console.log('Clicked');
  };

  return <button onClick={handleClick}>Click me</button>;
};
```

```jsx
// GOOD
const handleClick = () => {
  console.log('Clicked');
};

const MyComponent = () => (
  <button onClick={handleClick}>Click me</button>
);
```

**However**, if you need dynamic logic or closure over component state, you may define the function inside—but do so cautiously or use useCallback to memoize.

### Memoize Expensive Calculations

Use `useMemo` for expensive computations and `useCallback` for expensive or frequently re-created callbacks.

* `useMemo` returns a memoized value.
* `useCallback` returns a memoized callback function.

Only use them for **truly expensive operations** (e.g., large data transformations, heavy calculations). Overusing `useMemo/useCallback` can harm readability and performance.

```
// Suppose we have an expensive calculation
function calculateBigData(items) {
  // ...some heavy logic...
  return items.reduce(...);
}

const MyExpensiveComponent = ({ items, onSubmit }) => {
  const processedItems = useMemo(() => calculateBigData(items), [items]);

  const handleSubmit = useCallback(() => {
    // use processedItems
    onSubmit(processedItems);
  }, [processedItems, onSubmit]);

  return (
    <div>
      {/* Render processedItems */}
      <button onClick={handleSubmit}>Submit</button>
    </div>
  );
};
```

### Avoid Prop Drilling

When you pass props through multiple levels of components, it's called prop drilling. This can make your code difficult to follow and maintain.

**Strategies to Fix Prop Drilling**

* Context API: Great for global or app-wide data (e.g., user session, theme).
* Custom Hooks: Encapsulate state logic in a hook that can be reused.
* State Management Libraries: If the app is large or has complex state, consider libraries like Redux, Zustand, etc.

**Example of using Context**

```
// themeContext.js
export const ThemeContext = createContext();

export const ThemeProvider = ({ children }) => {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{theme, setTheme}}>
      {children}
    </ThemeContext.Provider>
  );
};

// AnyChildComponent.js
import { useContext } from 'react';
import { ThemeContext } from './themeContext';

const AnyChildComponent = () => {
  const { theme } = useContext(ThemeContext);
  return <div className={theme}>I am themed!</div>;
};
```

### Use Error Boundaries

To gracefully handle JavaScript errors, use **Error Boundaries**. They catch errors in the component tree and can show a fallback UI.

```
// ErrorBoundary.js
export class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    // Log error
  }

  render() {
    if (this.state.hasError) {
      return <h1>Something went wrong!</h1>;
    }
    return this.props.children;
  }
}
```

### Keep Components Self-Contained

When possible, *colocate* files related to a single component:

* Component File (MyComponent.tsx)
* Styles (MyComponent.styles.js or .module.css)
* Tests (MyComponent.test.ts)
* Types (MyComponent.types.ts if using TypeScript)

You can read more about *Organizing components* in the [Project Structure chapter](/frontend/react/project-structure).

### Code Splitting and Lazy Loading

Use `React.lazy` and `Suspense` to split your code into chunks, loading them on demand. This speeds up initial load times.

```
const About = React.lazy(() => import('./About'));

function App() {
  return (
    <Suspense fallback={<div>Loading...</div>}>
      <About />
    </Suspense>
  );
}
```

### Optimize Rendering with `React.memo`

Use `React.memo` for functional components to skip re-render if props don't change.

```
const MyButton = React.memo(({ label, onClick }) => {
  console.log('Render MyButton');
  return <button onClick={onClick}>{label}</button>;
});
```

### Performance Profiling

React DevTools provides Profiler to analyze renders, re-renders, and component updates.

**Using the React Profiler**

1. Install React DevTools Extension
   * Chrome: React DevTools
   * Firefox: React DevTools
2. Enable Profiler
   * Open React DevTools in Chrome/Firefox DevTools (F12 → Components tab).
   * Click on Profiler and Start Recording.
   * Perform interactions (e.g., clicking buttons, navigating).
   * Stop recording and analyze render times.
3. Identify Expensive Renders
   * React highlights slow renders in red.
   * Look for components that render frequently without needing to.
4. Use Profiling API in Production

```
import { Profiler } from 'react';

<Profiler id="MyComponent" onRender={(id, phase, actualDuration) => {
    console.log(`${id} re-rendered during ${phase}, took ${actualDuration}ms`);
}}>
    <MyComponent />
</Profiler>
```

### Use Lazy Initialization in `useState`

Avoid expensive calculations on initial render.

```
const [bigArray] = useState(() => new Array(10000).fill("Item"));
```

### Use `next/image` for Automatic Optimization

The `<Image />` component automatically optimizes images on the server side.

* Converts `image.jpg` to an optimized format (e.g., `WebP`).
* Serves different sizes based on the device screen (DPR-based resizing).
* Uses built-in lazy loading to defer off-screen images.
* Caches and compresses images automatically.
* Gives the `priority` prop for critical images (logos, banners, hero images) that should load immediately.
* Gives the `placeholder` props that automatically generates blurred placeholders for slow-loading images.

| Feature           | <img> (HTML) | <Image /> (Next.js) |
|-------------------|--------------|---------------------|
| Resizing          | No           | ✅ Yes               |
| Lazy Loading      | No           | ✅ Yes (default)     |
| Responsive Sizes  | No           | ✅ Yes               |
| Format Conversion | No           | ✅ Yes (WebP, AVIF)  |
| Caching           | No           | ✅ Yes               |
| Placeholder Blur  | No           | ✅ Yes               |

```
// Mobile (<768px): Image takes 100% of viewport width.
// Tablet (768px - 1200px): Image takes 50% of viewport.
// Larger screens: Fixed at 800px.

<Image 
  src="/large-image.jpg"
  width={800} 
  height={400} 
  alt="Large Image"
  priority // Has priority over other images
  placeholder="blur" // Renders blur placeholder
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 800px"
/>
```

**External images optimization**

By default, `<Image />` only optimizes local images (`/public` folder). To optimize external images (CDNs, APIs), configure `next.config.js`:

```
module.exports = {
  images: {
    remotePatterns: [
      {
        protocol: "https",
        hostname: "cdn.example.com",
        pathname: "/images/**",
      },
    ],
  },
};
```

**When to Use `<img>` Instead of `<Image />`**

| Use `<Image />` for:                         | Use `<img>` for:                                    |
|--------------------------------------------|---------------------------------------------------|
| Local images                               | Unoptimized external images                       |
| CDN-hosted images (with remotePatterns)    | Dynamic user-generated content (e.g., API images) |
| Images that must be responsive & optimized | Icons & simple inline images                      |

### Use Compound Components for Reusability

Instead of relying on props drilling, use *Compound Components* to allow multiple related components to work together seamlessly.

```
const Tabs = ({ children }) => {
  const [activeIndex, setActiveIndex] = useState(0);
  return React.Children.map(children, (child, index) =>
    React.cloneElement(child, { activeIndex, setActiveIndex, index })
  );
};

const TabList = ({ children, activeIndex, setActiveIndex }) => (
  <div>
    {React.Children.map(children, (child, index) =>
      React.cloneElement(child, { isActive: activeIndex === index, onClick: () => setActiveIndex(index) })
    )}
  </div>
);

const Tab = ({ children, isActive, onClick }) => (
  <button style={{ fontWeight: isActive ? "bold" : "normal" }} onClick={onClick}>
    {children}
  </button>
);

const TabPanels = ({ children, activeIndex }) => <div>{children[activeIndex]}</div>;

const TabPanel = ({ children }) => <div>{children}</div>;

// Usage
<Tabs>
  <TabList>
    <Tab>Tab 1</Tab>
    <Tab>Tab 2</Tab>
  </TabList>
  <TabPanels>
    <TabPanel>Content 1</TabPanel>
    <TabPanel>Content 2</TabPanel>
  </TabPanels>
</Tabs>;
```

### Use State Reducers

Instead of using multiple `useState` hooks, use a *state reducer* pattern to consolidate state updates and avoid prop drilling.

```
const initialState = { count: 0 };

const reducer = (state, action) => {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      return state;
  }
};

const Counter = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </div>
  );
};
```

### Optimize List Rendering with Virtualization

Rendering large lists (>1000 items) in React can cause performance issues. Use [TankStack Virtual](https://tanstack.com/virtual/latest) to optimize it.

**Why TanStack Virtual?**

* Efficient Rendering - Renders only visible elements instead of the entire list.
* Improved Performance - Reduces memory usage and re-renders, improving FPS and responsiveness.
* Highly Customizable - Works for lists, grids, infinite scrolling, and dynamic row heights.
* Lightweight & Framework-Agnostic - More flexible and performant than `react-window` or `react-virtualized`.

```
import { useVirtualizer } from "@tanstack/react-virtual";

const items = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);

const VirtualizedList = () => {
  const parentRef = useRef(null);

  const rowVirtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 35,
  });

  return (
    <div ref={parentRef}>
      <div style={{ height: rowVirtualizer.getTotalSize(), position: "relative" }}>
        {rowVirtualizer.getVirtualItems().map((virtualRow) => (
          <div key={virtualRow.key} style={{ position: "absolute", top: virtualRow.start }}>
            {items[virtualRow.index]}
          </div>
        ))}
      </div>
    </div>
  );
};

export default VirtualizedList;
```

### Use `useDeferredValue` for Smooth User Input

To reduce laggy UI updates, use useDeferredValue when dealing with search input or large state updates.

```
const Search = ({ query }) => {
  const deferredQuery = useDeferredValue(query); // Defer rendering heavy UI updates
  return <ExpensiveComponent data={filterData(deferredQuery)} />;
};
```
