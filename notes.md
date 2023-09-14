# React

## Components, Props & JSX

### Components

Components:

- React apps are made entirely of components
- Building blocks of the app
- Have their own look, data and logic
- Build UIs by combining them
- Can be reused, nested and pass data between them
- Used in a hierarchical component tree
- Components are just functions

They need to only return one parent JSX element, if need be wrap siblings in a single div like below, or a fragment `<></>` which will not show as an element in the dom.

```jsx
function App() {
  return (
    <div className="container">
      <Header />
      <Menu />
      <Footer />
    </div>
  );
}
```

Separation of concerns:
Components contain data, logic and appearance, so this is a different way of looking at separation of concerns, vs the old style of one file for css, one file for html and one for JS.

### JSX

JSX:

- Declarative syntax to describe how components look and work
- Components MUST return a block of JSX
- Extension of JS that allows us to embed JS, CSS and REact into HTML
- Each JSX element is converted to a `React.createElement()` function call
- We CAN use React w/o JSX

JSX is declarative:

- Describes the UI using current data
- React is an abstraction (we never touch the DOM)
- We think of the UI asa reflection of current data

### Logic Inside Components

Because components are just JS functions we can write JS in them. We can also use JS inside JSX in the return. To use JS we need to "Enter JS mode" which is done using curly brackets `{}`. Then inside the brackets we need to use JS EXPRESSIONS. So this means ternaries, short-circuiting, variables or return an array using array methods. Statements are NOT VALID in the JSX return. Ex.) if/else statements or switch statements. Styles can also be determined with JS mode inside the style attribute with a ternary.

Components can also be used with conditional or early returns

Other nots:

- CSS uses `className` instead of `class`
- `htmlFor` instead of HTML `for` with labels
- Every tag need to be closed
- Event handlers are camelCased `onSubmit`
- `aria-` and `data-` attributes are the same as HTML (lowercase with dash)
- CSS property names are camelCased `backgroundColor`
- Comments are inside `{}` because they're JS

```jsx
function Footer() {
  const hour = new Date().getHours();
  const openHour = 12;
  const closeHour = 22;
  const isOpen = hour >= openHour && hour <= closeHour;
  console.log(isOpen);

  if (!isOpen) return <footer>We're not open.</footer>;

  return (
    <footer className="footer">
      {isOpen ? (
        <Order closeHour={closeHour} openHour={openHour} />
      ) : (
        <p>
          We're happy to welcome you between {openHour}:00 and {closeHour}:00.
        </p>
      )}
    </footer>
  );
}
```

Conditional styles:

```jsx
function Pizza({ pizzaObj }) {
  return (
    <li className={`pizza ${pizzaObj.soldOut ? "sold-out" : ""}`}>
      <img src={pizzaObj.photoName} alt={pizzaObj.name} />
      <div>
        <h3>{pizzaObj.name}</h3>
        <p>{pizzaObj.ingredients}</p>
        <span>{pizzaObj.soldOut ? "SOLD OUT" : pizzaObj.price}</span>
      </div>
    </li>
  );
}
```

### Styles

Styles inside the index.css are global, whereas any styles added inside the component are inline or styles added from css modules are scoped to that component.

### Props

Props:

- Used to pass data from parent to child
- Parents can control how children look and work
- Anything can be passed as props even other components
- Props are READ ONLY and COME FROM THE PARENT
- Props can only be updated by the parent which means at that point its just a parent updating its own state
- Props are immutable so children don't touch their parents state
- One-way data flow from parent to child

## State, Events & Forms

### Events

In React for event handlers we need to pass a function definition to be called when the event occurs NOT a function call


State:
- Data that a component holds over time
- Updating state causes a component to rerender

useState hook:
- Returns an array of a state variable and a state updater function
- Hooks can only be called at the top level of a component & not inside statements (if/else)
- State should only be updated using the updater function
- If we want to overwrite state, pass the value directly to the state updater
- If we want to update state based on the previous state use the anonymous callback that allows access to the previous state
- Updating state rerenders the component
 

```jsx
function Steps() {
  const [step, setStep] = useState(1);
  const [isOpen, setIsOpen] = useState(true);

  function handlePrevious() {
    if (step > 1) setStep((s) => s - 1);
  }

  function handleNext() {
    if (step < 3) {
      setStep((s) => s + 1);
      // setStep((s) => s + 1);
    }

    // BAD PRACTICE
    // test.name = "Fred";
    // setTest({ name: "Fred" });
  }

  return (
    <div>
      <button className="close" onClick={() => setIsOpen((is) => !is)}>
        &times;
      </button>

      {isOpen && (
        <div className="steps">
          <div className="numbers">
            <div className={step >= 1 ? "active" : ""}>1</div>
            <div className={step >= 2 ? "active" : ""}>2</div>
            <div className={step >= 3 ? "active" : ""}>3</div>
          </div>

          <StepMessage step={step}>
            {messages[step - 1]}
            <div className="buttons">
              <Button
                bgColor="#e7e7e7"
                textColor="#333"
                onClick={() => alert(`Learn how to ${messages[step - 1]}`)}
              >
                Learn how
              </Button>
            </div>
          </StepMessage>

          <div className="buttons">
            <Button bgColor="#7950f2" textColor="#fff" onClick={handlePrevious}>
              <span>👈</span> Previous
            </Button>

            <Button bgColor="#7950f2" textColor="#fff" onClick={handleNext}>
              Next <span>👉</span>
              <span>🤓</span>
            </Button>
          </div>
        </div>
      )}
    </div>
  );
}
```

### Forms

#### Controlled Elements

By default input fields in the DOM maintain their own state, so react needs a way to overcome this. This is why we need to "Control" input elements.

Controlling state is done in 3 steps:
1. Create a piece of state
2. We specify the value attribute on the input we want to control and set it to that state
3. Then we need to connect the two by passing a callback to the onChange event that uses the state updating function

Keep in mind, on form submission, we still need to call `e.preventDefault`. Then we can create objects/ data structure from the state, and do whatever we want with it (lift it up etc.).

```jsx
import { useState } from "react";

export default function Form({ onAddItems }) {
  const [description, setDescription] = useState("");
  const [quantity, setQuantity] = useState(1);

  function handleSubmit(e) {
    e.preventDefault();

    if (!description) return;

    const newItem = { description, quantity, packed: false, id: Date.now() };

    onAddItems(newItem);

    setDescription("");
    setQuantity(1);
  }

  return (
    <form className="add-form" onSubmit={handleSubmit}>
      <h3>What do you need for your 😍 trip?</h3>
      <select
        value={quantity}
        onChange={(e) => setQuantity(Number(e.target.value))}
      >
        {Array.from({ length: 20 }, (_, i) => i + 1).map((num) => (
          <option value={num} key={num}>
            {num}
          </option>
        ))}
      </select>
      <input
        type="text"
        placeholder="Item..."
        value={description}
        onChange={(e) => setDescription(e.target.value)}
      />
      <button>Add</button>
    </form>
  );
}

```
## State MGMT Intro

- Always start with local state, then as needed move to global state (Context, Redux)
- We can lift up state by passing a defined function to children as props so we can share state with the parent and amongst siblings
- We can also derive state, which is preferred if possible to prevent using too much state & causing unnecessary rerenders

We can implement features like SORTING from derived state
```jsx
import { useState } from "react";
import Item from "./Item";

export default function PackingList({
  items,
  onDeleteItem,
  onToggleItem,
  onClearList,
}) {
  const [sortBy, setSortBy] = useState("input");

  let sortedItems;

  if (sortBy === "input") sortedItems = items;

  if (sortBy === "description")
    sortedItems = items
      .slice()
      .sort((a, b) => a.description.localeCompare(b.description));

  if (sortBy === "packed")
    sortedItems = items
      .slice()
      .sort((a, b) => Number(a.packed) - Number(b.packed));

  return (
    <div className="list">
      <ul>
        {sortedItems.map((item) => (
          <Item
            item={item}
            onDeleteItem={onDeleteItem}
            onToggleItem={onToggleItem}
            key={item.id}
          />
        ))}
      </ul>

      <div className="actions">
        <select value={sortBy} onChange={(e) => setSortBy(e.target.value)}>
          <option value="input">Sort by input order</option>
          <option value="description">Sort by description</option>
          <option value="packed">Sort by packed status</option>
        </select>
        <button onClick={onClearList}>Clear list</button>
      </div>
    </div>
  );
}

```

## Components, Composition & Reusability

- Components like functions should be abstracted when they get too big, but not too often with too many small components. This is more intuitive but don't worry about big components, moreso worry about if it's doing too many things.

We can use component composition by using the children prop to prevent prop drilling.

```jsx
export default function App() {
  const [movies, setMovies] = useState(tempMovieData);
  const [watched, setWatched] = useState(tempWatchedData);

  return (
    <>
      <NavBar>
        <Search />
        <NumResults movies={movies} />
      </NavBar>

      <Main>
        <Box>
          <MovieList movies={movies} />
        </Box>

        <Box>
          <WatchedSummary watched={watched} />
          <WatchedMoviesList watched={watched} />
        </Box>
      </Main>
    </>
  );
}
```

## React Behind The Scenes

### Components, Instances & Elements

Component:
- Function that re turns a react elements tree (JSX)
- Blueprints

Instance:
- When we use the component
- Each instance gets its own separate state, props and lifecycle

Element:
- All elements from the JSX are converted to `React.createElement` calls
- These come together in a big object called the VDOM(Virtual DOM or more accurately "React Element Tree")


### Rendering

1. Render is triggered
- Either on initial render
- Or rerender by a state update

2. Render Phase
- Component functions are called & react figures out HOW to update the DOM
- This only happens internally inside react (Not the traditional term of render)

3. Commit Phase
- React writes to the DOM, inserting, updating and deleting elements

4. Browser Paint
- This is outside of React at this point but this is the final step that produces the visual change that users see

#### Render Triggers

Only 2 ways:
- Initial render
- State update in on or more components (rerender)

*Render process is always triggered for the whole application* (React looks at the entire tree)
- Renders are scheduled when the JS engine has time

#### Render Phase

- React goes through the ENTIRE component tree to see which components triggered a render and call those component functions which results in a new VDOM (React Element Tree)

Initial Render:
- React calls all components to create a VDOM (React Element Tree)
- Actually just a POJO (Plain old javascript object)

Rerenders:
- State update happens
- React goes through tree and calls components that triggered a render (state update)
- ALL CHILDREN OF THAT COMPONENT ARE RERENDERED no matter what
- This only means that the VDOM is recreated NOT the actual DOM
- The new VDOM is then reconciled & diffed against the FIBER TREE as it existed before the state update inside Reacts reconciler called "Fiber" hence the name "Fiber Tree", resulting in a new updated "Fiber Tree" and is then used to write to the DOM

Reconciliation:
- Writing to the DOM is slow and thats why React does this reconciliation against the VDOM because its cheaper and usually only certain parts need updates/creation/deletion
- Result of this is a list of operations to perform on the DOM with the new state
- "Fiber" is the engine of React
- Fiber creates a fiber tree from the React Element Tree which has a "fiber" for each component instance and DOM element
- Fibers are NOT created on every render, but rather mutated over and over again
- Each fiber contains current state, props, side effects, used hooks and queue of work to be performed
- Fibers are arranged in a linked list, which makes it easier for react to process work
- Work can be performed asynchronously which enables new features like `Suspense` and transitions
- Long renders are non-blocking
- New "Work in progress" fiber tree created
- List of DOM updates is created

Commit Phase:
- React writes to the DOM ("Flushes" to the DOM)
- Committing is synchronous (DOM updated in one go and cant be interrupted to ensure no impartial results)
- Work in progress fiber tree becomes the current fiber tree for the next cycle
- Done by the web host `react-dom` 
- This is what makes react cross platform as the commit phase is done by different "hosts"

Browser Paint:
- Done by whatever browser the user is using

### Diffing

2 Assumptions:
1. 2 elements of different types will produce different trees
2. Elements with a stable key will stay the same across renders

Only 2 different scenarios:
- Same position DIFFERENT element
- Same position SAME element

1. Same position DIFFERENT element (DOM or React element)
- Entire subtree including state considered invalid and removed
- This means child state might be completely reset

2. Same position SAME element (DOM or React element)
- Element & children and state is kept
- New props/attributes are passed if the changed between renders and again element and children are kept
- If we want to keep state, we can use the key prop to differentiate unique elements of the same type
- If we want to clear out state, we can use the key prop and change it

### Render Logic
There is:
- Render logic (State and things that produce JSX)
- Event handlers

Components are pure functions, so given the same input (props) will produce the same output (JSX)

Rules for render logic:
- No side effects
- Dont perform network requests
- Dont start timers
- Dont use DOM APIs
- Dont mutate objects or variables outside function scope
- Dont update state or refs (This causes infinite loops)


Side effects should be done with:
- Event handlers (default strategy)
- Effects

*Side Note*
State in event handler functions are batched and asynchronous. This is where using set state with the callback accessing the previous state is helpful.
We can see this happen if we console log a state we just updated, we get the old value.

### React Events

React uses event delegation automatically for us on the root element. 


## Effects & Data Fetching

When are effects Executed?
Effects run after the component mounts, because effects may contain long running logic like fetching data

- Mount (Initial Render)
- Commit
- Browser Paint
- EFFECT

NOTE:
Setting state inside an effect will cause additional rerender and they should NOT BE OVERUSED

Use effect also has cleanup function that can be returned from it's callback to perform cleanup work as needed. Ex.) Clearing a timer, using an abort controller

## Custom Hooks, Refs & More State

### useRef

With React we don't manually touch the DOM, we let React do it. React provides us with a "ref" which is a reference.

- Essentially a box we can put any data to be preserved between renders in.
- Has a `.current` property that we use to read and write to.

2 Use cases:

1. - Persist state across renders
2. - Select & store DOM elements

- Refs are for data that is NOT RENDERED. Usually only appear in event handlers or effects, not in the JSX (Otherwise use state).
- DONT READ or WRITE TO `.current` in RENDER LOGIC (like state) - use a useEffect hook instead

#### State vs Refs

State:

- Persist state
- Updating causes re-renders
- Immutable
- Asynchronous

Refs:

- Persist state
- Updating DOESNT cause a re-render
- Mutable
- Synchronous

Here we set the initial useRef state to null which is common. Then we add the `ref` property to bind to the desired element and then in the useEffect we have access to the `.current` property after the element has been mounted in the DOM. In this case the `inputEl.current` gives us back the html input element which we can then call the focus method on.

```jsx
function Search({ query, setQuery }) {
  const inputEl = useRef(null);

  useEffect(() => {
    inputEl.current.focus();
  }, []);

  return (
    <input
      className="search"
      type="text"
      placeholder="Search movies..."
      value={query}
      onChange={(e) => setQuery(e.target.value)}
      ref={inputEl}
    />
  );
}
```

### Custom Hooks

2 pieces we can reuse:

1. UI - Components
2. Logic - Functions or custom hooks

If the logic you want to reuse doesn't contain any hooks, just create a reusable helper function. If it DOES CONTAIN HOOKS, then implement a custom hook.

- Need to be prefixed with `use`
- Allow us to reuse non-visual logic
- A custom hook should have one purpose to make it reusable and portable
- Rules of hooks still apply

In our app.js we use our custom hook called useMovies and pass in the required arguments. Below the app.js we see the useMovies hook is composed of 4 hooks. 3 useStates and a useEffect.

```jsx
// App.js

import { useEffect, useRef, useState } from "react";
import StarRating from "./StarRating";
import { useMovies } from "./useMovies";

const average = (arr) =>
  arr.reduce((acc, cur, i, arr) => acc + cur / arr.length, 0);

  const KEY = "e3e85fc8";

export default function App() {
  // const [watched, setWatched] = useState([]);
  const [watched, setWatched] = useState(() => {
    const storedValue = localStorage.getItem('watched');
    return JSON.parse(storedValue);
  });

  const [query, setQuery] = useState("");
  const [selectedId, setSelectedId] = useState(null);

  const { movies, isLoading, error } = useMovies(query, handleCloseMovie);

  function handleSelectMovie(movieId) {
    setSelectedId((curId) => (movieId === curId ? null : movieId));
  }

  function handleCloseMovie() {
    setSelectedId(null);
  }

  ...etc
}
```

```jsx
// useMovie.js
import { useEffect, useState } from "react";

const KEY = "e3e85fc8";

export function useMovies(query, callback) {
  const [movies, setMovies] = useState([]);

  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState("");

  useEffect(() => {
    callback?.();

    const controller = new AbortController();

    const getMovies = async () => {
      try {
        setError(() => "");
        setIsLoading(() => true);
        const res = await fetch(
          `http://www.omdbapi.com/?apikey=${KEY}&s=${query}`,
          { signal: controller.signal }
        );

        if (!res.ok) {
          throw new Error("Something went wrong fetching movies!");
        }

        const data = await res.json();

        if (data.Response === "False") {
          throw new Error("Movie not found!");
        }

        setMovies(() => data.Search);
        setError("");
      } catch (error) {
        console.log(error.message);
        if (error.name !== "AbortError") setError(() => error.message);
      } finally {
        setIsLoading(() => false);
      }
    };

    if (query.length < 3) {
      setMovies(() => []);
      setError("");
      return;
    }

    // handleCloseMovie();
    getMovies();

    return () => {
      controller.abort();
    };
  }, [query, callback]);

  return { movies, isLoading, error };
}
```

## useReducer

**Note**
It has been mentioned in articles that this hook is NOT a replacement for redux, but when use with the context API CAN replace redux in SOME cases

### First Look

The useReducer hook takes from the redux pattern and its concepts. The hook itself takes in a reducer function which you need to define to handle the different actions. The second argument is the initial state. The function returns an array of the state and the dispatch function to dispatch actions. Actions by convention have a type and payload property. Then we can switch over the `action.type` to control the flow of state updates based on what action has been dispatched. We always return the updated state, so using the spread operator is a normal convention then just update the property that needs updating.

```jsx
import { useReducer, useState } from "react";

const initialState = {
  count: 0,
  step: 1,
};

function reducer(state, action) {
  console.log(state);
  console.log(action);
  switch (action.type) {
    case "dec":
      return { ...state, count: state.count - state.step };
    case "inc":
      return { ...state, count: state.count + state.step };
    case "setCount":
      return { ...state, count: action.payload };
    case "setStep":
      return { ...state, step: action.payload };
    case "reset":
      return initialState;
    default:
      throw new Error("Unknown action!");
  }
}

function DateCounter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  const { count, step } = state;

  const date = new Date("june 21 2027");
  date.setDate(date.getDate() + count);

  const dec = function () {
    dispatch({ type: "dec" });
  };

  const inc = function () {
    dispatch({ type: "inc" });
  };

  const defineCount = function (e) {
    dispatch({ type: "setCount", payload: Number(e.target.value) });
  };

  const defineStep = function (e) {
    dispatch({ type: "setStep", payload: Number(e.target.value) });
  };

  const reset = function () {
    dispatch({ type: "reset" });
  };

  return (
    <div className="counter">
      <div>
        <input
          type="range"
          min="0"
          max="10"
          value={step}
          onChange={defineStep}
        />
        <span>{step}</span>
      </div>

      <div>
        <button onClick={dec}>-</button>
        <input value={count} onChange={defineCount} />
        <button onClick={inc}>+</button>
      </div>

      <p>{date.toDateString()}</p>

      <div>
        <button onClick={reset}>Reset</button>
      </div>
    </div>
  );
}
export default DateCounter;
```

### How it works

Why useReducer:

1. When components have lots of state variables & state updates across many event handlers all over the component
2. Multiple state updates need to happen at the same time
3. When updating one piece of state depends on one or more pieces of state

- Alternative way of setting state ideal for complex state and related pieces of state
- Related state is stored in a state object
- Reducer function contains all logic to update state & decouples state from the component
- Reducer is a pure function with no side effects. Doesn't mutate the current state, rather returns a new updated state.
- Actions are objects that describe how to update state with a type and payload
- Dispatch functions are used to trigger state updates by sending actions from event handles to the reducer
- Updating the state with useReducer also causes re-renders

### useState vs useReducer

useState:

- Single independent pieces of state
- Simple
- Should be the default go to

useReducer:

- Related or complex state
- State updates need to happen together
- Too many event handlers? Use useReducers
- Little more complex (Easy once redux clicks)

## Routing & SPAs

### Vite

Using vite as a build tool run `npm create vite` and you can select a start from different frameworks...pretty cool.

We used create-react-app prior to this as it comes setup with eslint and many other things and iw fine for learning. Vite however is the reommended build tool from the react team.

To setup eslint we run `npm i eslint vite-plugin-eslint eslint-config-react-app --save-dev`
Then we needs to create an `eslintrc.json` to extends the default rules of eslint with the react rules we just installed

```json
// eslintrc.json
{
  "extends": "react-app"
}
```

Then we need to configure our vite project with the `vite.config.js`. We import eslint from vite-plugin-eslint and add it to the plugins array

```js
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

import eslint from "vite-plugin-eslint ";

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), eslint],
});
```

Now eslint is running in our project

### Routes

React needs 3rd party libraries to get additional functionality. Routing isn't built in. S owe add `npm i react-router-dom`. You know routing so its sorta self explanatory. But we import the browser router, routes and route. This is the traditional routing strategy. Then a route will get a path and an element to render when the url has that path. `*` will catch wildcard routes so we can render a 404.

```jsx
import { BrowserRouter, Routes, Route } from "react-router-dom";

function App() {
  return (
    <BrowserRouter>
      <h1>Yeah buddy</h1>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="product" element={<Product />} />
        <Route path="pricing" element={<Pricing />} />
        <Route path="*" element={<PageNotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

export default App;
```

### CSS Modules

These are baked into react and will scope styles to the component only. All you need to do is create a css file that follows the naming convention matching the component. Ex.) `Nav.module.css`. Then in the styles make sure you are not using element tags first as it will break out of the component scoping. Ex.) Don't use `ul {display: flex}` as it will apply to all ul's. Instead use `.list {display: flex}` or `.nav ul {display: flex}`.

**UMD Global & Other random errors**

After adding a `jsconfig.json` then removing it. I started getting typescript errors everywhere. I implemented a basic jsconfig.json as per VS Code docs after 2 1/2hrs of struggling to figure out what the fuck went wrong. Sometimes adding an empty jsconfig.json fixes these errors

```json
// jsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es6",
    "jsx": "preserve"
  },
  "exclude": ["node_modules", "**/node_modules/*"]
}
```

### Nested Routes

Nested routes can be wrapped in a parent `Route` tag and will be relative to the parent url. Then we use it as a regular Route tag to render a child route component inside it. To tell React where to render it we need to also use the `<Outlet>` component inside the parent component. It behaves similar to the `{props.children}` I suppose.

Additionally we can specify the index attribute to make a component the default component to be shown. With nested routes provide the duplicate with the index attribute. In this instance when `localhost/app` is visited the cities component will be rendered by default. Note: The url WILL be `localhost/app` in that default instance not `localhost/app/cities`.

```jsx
function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route index element={<Homepage />} />
        <Route path="product" element={<Product />} />
        <Route path="pricing" element={<Pricing />} />
        <Route path="login" element={<Login />} />
        <Route path="app" element={<AppLayout />}>
          <Route index element={<p>Cities</p>}></Route>
          <Route path="cities" element={<p>Cities</p>}></Route>
          <Route path="countries" element={<p>Countries</p>}></Route>
          <Route path="form" element={<p>Form</p>}></Route>
        </Route>
        <Route path="*" element={<PageNotFound />} />
      </Routes>
    </BrowserRouter>
  );
}
```

```jsx
function AppLayout() {
  return (
    <div className={styles.app}>
      <Sidebar />
    </div>
  );
}
```

```jsx
function Sidebar() {
  return (
    <div className={styles.sidebar}>
      <Logo />
      <AppNav />

      // OUTLET HERE
      <Outlet />

      <footer className={styles.footer}>
        <p className={styles.copyright}>
          &copy; Copyright {new Date().getFullYear()} by WorldWise Inc.
        </p>
      </footer>
    </div>
  );
}
```

Notice we can use the outlet a child further down the tree and it still works.

### URL For State Management

The url is actually an easy global place to store simple forms of state like filters, ids, sorting order, open/closed panels etc.

- It allows us to sidestep lifting state to a parent component
- Allows state passing from one page to another
- Can bookmark and share a "saved state" from a specific state/time

This is done through params & query strings:

Params:

- Pass data between pages

Query String:

- Can store global data

### Dynamic Routes

We create a new route with a dynamic :id

`<Route path="cities/:id" element={<City />}></Route>`

Then we link to that route with the `to={}` being the param we want. (Keep in mind relative vs root relative urls)

```jsx
function CityItem({ city }) {
  const { cityName, emoji, date, id } = city;
  return (
    <li>
      <Link to={`${id}`} className={styles.cityItem}>
        <span className={styles.cityEmoji}>{emoji}</span>
        <h3 className={styles.name}>{cityName}</h3>
        <time className={styles.date}>({formatDate(date)})</time>
        <button className={styles.deleteBtn}>&times;</button>
      </Link>
    </li>
  );
}
```

Then in the component where we want to use the param, we use the useParam hook and destructure the variable we called it in the route.

```jsx
function City() {

  const { id } = useParams();
  console.log(id);

  ...
}
```

### Query Strings

Same same, but different.

In the component where we want to add query strings we add them with and initial `?` then each addditional prepended with an `&`.

```jsx
function CityItem({ city }) {
  const { cityName, emoji, date, id, position } = city;
  const { lat, lng } = position;
  return (
    <li>
      <Link to={`${id}?lat=${lat}&lng=${lng}`} className={styles.cityItem}>
        <span className={styles.cityEmoji}>{emoji}</span>
        <h3 className={styles.name}>{cityName}</h3>
        <time className={styles.date}>({formatDate(date)})</time>
        <button className={styles.deleteBtn}>&times;</button>
      </Link>
    </li>
  );
}
```

Then we can use another hook called the `useSearchParams` hook, and destructure as needed in any component that is currently rendered.

Search params are accessed using `searchParams.get('paramName')` which is a convention in web dev

The hook is very similar to `useState()` and also comes with an updater function to update the search params as needed, even adding new additional ones as shown below.

```jsx
function Map() {
  const [searchParams, setSearchParams] = useSearchParams();

  const lat = searchParams.get("lat");
  const lng = searchParams.get("lng");

  return (
    <div className={styles.mapContainer}>
      <h1>Map</h1>
      <h1>
        Position: Lat {lat} | Lng {lng}
      </h1>

      <button
        onClick={() => setSearchParams({ lat: 23, lng: 50, fuckyFuck: "Yes" })}
      >
        Change Position
      </button>
    </div>
  );
}
```

```jsx
function City() {
  const { id } = useParams();
  console.log(id);

  const [searchParams, setSearchParams] = useSearchParams();

  const lat = searchParams.get("lat");
  const lng = searchParams.get("lng");
}
```

### Programmatic Navigation - useNavigate

Useful when save has completed. Or just need to control navigation manually.

We can navigate by url (Keep in mind relative and root relative)

```jsx
return (
  <div className={styles.mapContainer} onClick={() => navigate("form")}>
    <h1>Map</h1>
    <h1>
      Position: Lat {lat} | Lng {lng}
    </h1>

    <button
      onClick={() => setSearchParams({ lat: 23, lng: 50, fuckyFuck: "Yes" })}
    >
      Change Position
    </button>
  </div>
);
```

Or we can use it to go back. (I assume its based on the browser history api)

```jsx
<Button
  type="back"
  onClick={(e) => {
    e.preventDefault();
    navigate(-1);
  }}
>
  &larr; Back
</Button>
```

### Navigate Component

**Not used as much since there's hooks now**

**BUT**
_Still useful in nested routes as REDIRECTS_

The Navigate component MUST have the REPLACE attribute if you want TO BE ABLE TO "GO BACK"

```jsx
<Route index element={<Navigate replace to="cities" />}></Route>
<Route
  path="cities"
  element={<CityList cities={cities} isLoading={isLoading} />}
></Route>
```

## Context API

### Intro

Context API

- A solution to prop drilling
- System to pass data throughout the application without needing to prop drill (GLOBAL STATE)

Parts:

1. Provider

- Special react component that gives children access to a value (commonly placed at the APP level)

2. Value

- Value -> Data we want to make available (Usually state and functions)

3. Consumers

- Components that read the provided context value

When the context value changes, ALL CONSUMERS who are SUBSCRIBED, RE-RENDER.

### Basic Usage

We create a context outside the component we want to provide to (Usually done in its own file and exported I think). The context is capitalized because it is considered a special type of component.

```jsx
const PostContext = createContext();
```

Then inside the component we want to provide values to all its children, we wrap the elements in the `Context.Provider` tag and pass any desired value to the value attribute

```jsx
return (
  <PostContext.Provider
    value={{
      posts: searchedPosts,
      onAddPost: handleAddPost,
      onClearPosts: handleClearPosts,
      searchQuery,
      setSearchQuery,
    }}
  >
    <section>
      <button
        onClick={() => setIsFakeDark((isFakeDark) => !isFakeDark)}
        className="btn-fake-dark-mode"
      >
        {isFakeDark ? "☀️" : "🌙"}
      </button>

      <Header />
      <Main />
      <Archive />
      <Footer />
    </section>
  </PostContext.Provider>
);
```

Then inside ANY child component we use the `useContext` hook to destructure and values from the context we desire

```jsx
const { searchQuery, setSearchQuery } = useContext(PostContext);
```

### Custom Context/Provider

We create our context outside the provider named PostContext. Then the PostProvider is our component that we add all our state, derived state and handler functions to. Then we return the `PostContext.Provider` witth all our values passed in that we want child components to have access to. Then we wrap it around the children props and export both the component (provider) and context.

```jsx
const { createContext, useState } = require("react");
const { createRandomPost } = require("./App");

const PostContext = createContext();

function PostProvider({ children }) {
  const [posts, setPosts] = useState(() =>
    Array.from({ length: 30 }, () => createRandomPost())
  );
  const [searchQuery, setSearchQuery] = useState("");

  // Derived state. These are the posts that will actually be displayed
  const searchedPosts =
    searchQuery.length > 0
      ? posts.filter((post) =>
          `${post.title} ${post.body}`
            .toLowerCase()
            .includes(searchQuery.toLowerCase())
        )
      : posts;

  function handleAddPost(post) {
    setPosts((posts) => [post, ...posts]);
  }

  function handleClearPosts() {
    setPosts([]);
  }

  return (
    <PostContext.Provider
      value={{
        posts: searchedPosts,
        onAddPost: handleAddPost,
        onClearPosts: handleClearPosts,
        searchQuery,
        setSearchQuery,
      }}
    >
      {children}
    </PostContext.Provider>
  );
}

export { PostContext, PostProvider };
```

Then to use it we just wrap any children we want to have access to the values in the component (provider) tag. And in the child components we still use the useContext hook the reference the context and destructure our values.

```jsx
return (
  <section>
    <button
      onClick={() => setIsFakeDark((isFakeDark) => !isFakeDark)}
      className="btn-fake-dark-mode"
    >
      {isFakeDark ? "☀️" : "🌙"}
    </button>

    <PostProvider>
      <Header />
      <Main />
      <Archive />
    </PostProvider>
    <Footer />
  </section>
);
```

```jsx
function Header() {
  const { onClearPosts } = useContext(PostContext);

  return (
    <header>
      <h1>
        <span>⚛️</span>The Atomic Blog
      </h1>
      <div>
        <Results />
        <SearchPosts />
        <button onClick={onClearPosts}>Clear posts</button>
      </div>
    </header>
  );
}
```

A common practice is to export a custom hook in the same file as the context that returns the context to write a little less code.

We also add a check, so if any dev tries to use the context "Outside the provider" (Not in the children of the provider) we can throw an error for them and let them know. Ex.) Trying to destructure the post context at the app level

```jsx
// PostContext.js
function usePosts() {
  const context = useContext(PostContext);
  if (context === undefined)
    throw new Error("PostContext was used outside of the PostProvider");
  return context;
}

export { PostProvider, usePosts };
```

```jsx
const { onClearPosts } = usePosts();
```

### State MGMT With Context API

#### Types of state

1. State Accessibility
   Local State:

- Only needed by one or a few components
- Only accessible in component and children

Global State:

- Needed by many components
- Accessible by every component in the app

2. StateDomain
   Remote State:

- Loaded from a remote server(API)
- Asynchronous
- Needs re-fetching, caching, updating

UI State:

- Everything else
- Theme, list filters, form data etc.
- Synchronous
- Stored in the app

#### Placement

Local Component

- useState, useReducer, useRef
- Local State

Parent Component

- useState, useReducer, useRef
- Lifting up state

Context

- Context API + useState or useReducer
- Global State (preferably UI state)

3rd Party

- Redux, React Query, SWR, Zustand
- Global State (Remote or UI)

URL

- React Router
- Global State, passing between pages

Browser

- Local Storage, Session Storage etc.
- Storing data in users browser

#### Tool Options

Local/UI State

- useState, useReducer, useRef

Local/Remote State

- fetch + useEffect + useState/useReducer
- Good for small apps

Global/Ui State

- Context API + useState/useReducer
- Redux, Zustand, Recoil etc.
- React Router

Global/Remote State

- Context API + useState/useReducer
- Redux, Zustand, Recoil etc.
- React Router
- React Query, SWR, RTK Query (All highly specialized tools)

### Advanced State MGMT (Context API/useReducer)

A common pattern for complex global state is to use the use
Reducer hook alongside the context api.

```jsx
import { createContext, useContext, useEffect, useReducer } from "react";

const CitiesContext = createContext();

const BASE_URL = "http://localhost:8000";

const initialState = {
  cities: [],
  isLoading: false,
  currentCity: {},
  cityContextError: "",
};

function reducer(state, action) {
  switch (action.type) {
    case "LOADING":
      return { ...state, isLoading: action.payload };
    case "CITIES_LOADED":
      return { ...state, cities: [...action.payload], isLoading: false };
    case "CURRENT_CITY_LOADED":
      return { ...state, currentCity: action.payload, isLoading: false };
    case "CREATE_CITY":
      return {
        ...state,
        cities: [...state.cities, action.payload],
        isLoading: false,
        currentCity: action.payload,
      };
    case "DELETE_CITY":
      return {
        ...state,
        cities: state.cities.filter((city) => city.id !== action.payload),
        isLoading: false,
        currentCity: {},
      };
    case "ERROR":
      return { ...state, error: action.payload, isLoading: false };
  }
}

function CitiesProvider({ children }) {
  const [{ cities, isLoading, currentCity, cityContextError }, dispatch] =
    useReducer(reducer, initialState);

  useEffect(() => {
    const fetchCities = async () => {
      try {
        dispatch({ type: "LOADING", payload: true });
        const res = await fetch(`${BASE_URL}/cities`);
        const data = await res.json();
        dispatch({ type: "CITIES_LOADED", payload: data });
      } catch (error) {
        console.log(`There was an error! ${error.message}`);
        dispatch({ type: "ERROR", payload: error.message });
      }
    };
    fetchCities();
  }, []);

  async function getCity(id) {
    if (Number(id) === currentCity.id) return;
    try {
      dispatch({ type: "LOADING", payload: true });
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      const data = await res.json();
      dispatch({ type: "CURRENT_CITY_LOADED", payload: data });
    } catch (error) {
      console.log(`There was an error getting the city! ${error.message}`);
      dispatch({ type: "ERROR", payload: error.message });
    }
  }

  async function createCity(newCity) {
    try {
      dispatch({ type: "LOADING", payload: true });
      const res = await fetch(`${BASE_URL}/cities`, {
        method: "POST",
        body: JSON.stringify(newCity),
        headers: { "Content-Type": "application/json" },
      });
      const data = await res.json();
      dispatch({ type: "CREATE_CITY", payload: data });
    } catch (error) {
      console.log(`There was an error creating the city! ${error.message}`);
      dispatch({ type: "ERROR", payload: error.message });
    }
  }

  async function deleteCity(id) {
    try {
      dispatch({ type: "LOADING", payload: true });
      await fetch(`${BASE_URL}/cities/${id}`, { method: "Delete" });
      dispatch({ type: "DELETE_CITY", payload: id });
    } catch (error) {
      console.log(`There was an error deleting the city! ${error.message}`);
      dispatch({ type: "ERROR", payload: error.message });
    }
  }

  return (
    <CitiesContext.Provider
      value={{
        cities,
        isLoading,
        currentCity,
        getCity,
        createCity,
        deleteCity,
        cityContextError,
      }}
    >
      {children}
    </CitiesContext.Provider>
  );
}

function useCities() {
  const context = useContext(CitiesContext);
  if (context === undefined)
    throw new Error("CitiesContext used outside of CitiesProvider.");
  if (context.error) throw new Error(context.error);
  return context;
}

export { CitiesProvider, useCities };
```

### Protected Routes

To protect routes we can access a global auth context to determine if the user is logged in and if not then redirect them back to home.

```jsx
import { useNavigate } from "react-router-dom";
import { useAuth } from "../contexts/AuthContext";
import { useEffect } from "react";

function ProtectedRoute({ children }) {
  const { isSignedIn } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isSignedIn) navigate("/");
  }, [isSignedIn, navigate]);

  return isSignedIn ? children : null;
}

export default ProtectedRoute;
```

We wrap the components we want to protect in the protected route component and check the auth state. If the user is authenticated then we can render its children.

```jsx
// App.jsx

return (
  <AuthProvider>
    <CitiesProvider>
      <BrowserRouter>
        <Routes>
          <Route index element={<Homepage />} />
          <Route path="product" element={<Product />} />
          <Route path="pricing" element={<Pricing />} />
          <Route path="login" element={<Login />} />
          <Route
            path="app"
            element={
              <ProtectedRoute>
                <AppLayout />
              </ProtectedRoute>
            }
          >
            <Route index element={<Navigate replace to="cities" />}></Route>
            <Route path="cities" element={<CityList />}></Route>
            <Route path="cities/:id" element={<City />}></Route>
            <Route path="countries" element={<CountryList />}></Route>
            <Route path="form" element={<Form />}></Route>
          </Route>
          <Route path="*" element={<PageNotFound />} />
        </Routes>
      </BrowserRouter>
    </CitiesProvider>
  </AuthProvider>
);
```

## Performance Optimizations/Advanced useEffect

### Basics

Perf Op Tools:

1. Prevent wasted renders

- memo
- useMemo
- useCallback
- Passing elements as children or as props

2. Improve App Speed/Responsiveness

- use Memo
- useCallback
- useTransition

3. Reduce Bundle Size

- Use fewer 3rd party packages
- Code splitting/Lazy loading

Rerenders happen in 3 instances:

1. State changes
2. Context changes that a component is subscribed to

- Important to note that if ANYTHING changes in the context the subscribed component rerenders. Ex.) The component is only subscribed to the context for a function, but another component causes a state update in the context, the component subscribed for just some function usage rerenders too.

3. Parent rerenders

- Parents rerenders give the common misconception that changing props rerenders a component. This is untrue. Props only change when the parent rerenders which in turn causes all children to rerender anyway.

Remember:
Render does not mean that the DOM rerenders, it means that React is creating a new VDOM and running its diffing and reconcilliation phases with the component functions being called. But this can be expensive.

Wasted render:
A render where the diffing/reconcile phases ran but no changes were made.

> Only a problem when they happen too much or the component is very slow

### Components As Props

We can use component composition to access a component as children or pass an element as a defined prop to render in a component. If this component is very slow to render and the parent changes, we can do this to prevent rerenders of this slow child component. This is because React is smart enough to know that when the state changes in the parent, but the slow child DOESNT DEPEND ON PARENT STATE that the child already exists and should not rerender. (This is because components are functions and if the input doesn't change it wont rerender the inputs)

Here we see that every increase to count will rerender a list of 100,000 elements, causing substantial lagginess.

```jsx
import { useState } from "react";

function SlowComponent() {
  const words = Array.from({ length: 100_000 }, () => "WORD");
  return (
    <ul>
      {words.map((word, i) => (
        <li key={i}>
          {i}: {word}
        </li>
      ))}
    </ul>
  );
}

export default function Test() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      <SlowComponent />
    </div>
  );
}
```

When we refactor to pass the slow component using composition and it doesnt depend on its parents state, then React prevents rerenders of the slow component and the updates to state are instant and smooth.

```jsx
import { useState } from "react";

function SlowComponent() {
  // If this is too slow on your maching, reduce the `length`
  const words = Array.from({ length: 100_000 }, () => "WORD");
  return (
    <ul>
      {words.map((word, i) => (
        <li key={i}>
          {i}: {word}
        </li>
      ))}
    </ul>
  );
}

function Counter({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button>
      {children}
    </div>
  );
}

export default function Test() {
  // const [count, setCount] = useState(0);
  return (
    <div>
      {/* <h1>Slow counter?!?</h1>
      <button onClick={() => setCount((c) => c + 1)}>Increase: {count}</button> */}
      <Counter>
        <SlowComponent />
      </Counter>
    </div>
  );
}
```

We can see this effect in the profiler with providers as well. Even though the providers are parent components when state changes happen only children who are subscribed to the context are rerendered.

```jsx
function PostProvider({children}) {

  ...etc

  return (
    <PostContext.Provider
      value={{
        posts: searchedPosts,
        onAddPost: handleAddPost,
        onClearPosts: handleClearPosts,
        searchQuery,
        setSearchQuery,
      }}
    >
      {children}
    </PostContext.Provider>
  )
}
```

### Memoization

Memoization:

- An optimization technique that executes a pure function once, then stores the results in memory. If the function is called again with the exact same inputs then the cached value is used and the function is not executed again. Otherwise if the inputs change, then it will execute again.

In React we can a few memoization techniques

1. Component memoization with `memo`
2. Object memoization with `useMemo`
3. Function memoization with `useCallback`

These:

1. Prevent wasted renders
2. Improve app speed/responsiveness

#### Memo

- Used to create a component that WILL NOT RERENDER WHEN ITS PARENT RERENDERS as long as the PROPS HAVE STAYED THE SAME
- ONLY AFFECTS PROPS. Will still rerender if its own state changes, or when a context it is subscribed to changes
- Only use with a **heavy** component or one that rerenders often and in either case props stay the same

We can simplify `memo` by wrapping the default export of a component instead of redeclaring a variable.

```jsx
import { memo } from "react";

function ToggleSounds({ allowSound, setAllowSound }) {
  return (
    <button
      className="btn-sound"
      onClick={() => setAllowSound((allow) => !allow)}
    >
      {allowSound ? "🔈" : "🔇"}
    </button>
  );
}

export default memo(ToggleSounds);
```

#### useMemo & useCallback

If we have an object or function created inside a component and that component rerenders, even if the object/function looks exactly the same it is actually a brand new one.

- useMemo is used to cache values (Objects)
- useCallback is used to cache functions
- Cached values/functions will be returned as long as dependencies (in the dependency array) stay the same, otherwise the value will be recreated

3 main use cases:

1. Memoizing props to prevent wasted rerenders
2. Memoize values to prevent expensive recalculations
3. Memoizing values that are use in the dependency array of another hook

useCallback Example:

In the app when the `city:id` route is rendered from clicking on a city in the city list the id is updated in the url and the city component mounts. When the id changes the useEffect dependent on that id will run and call `getCity(id)`. We can see in the context that getCity updates the state, causing a rerender of the provider, this causes the getCity function to be recreated. Then because the effect has the getCity function in it's dependency array it will be called again, which will update the state causing a rerender of the provider recreating the getCity function again, resulting in an infinite loop.

the solver as shown below is to use `useCallback` to memoize the function to prevent the value from being recalculated if the input has not changed, solving our problem and preventing the infinite loop.

```jsx
// App.jsx
function App() {
  return (
    <AuthProvider>
      <CitiesProvider>
        <BrowserRouter>
          <Routes>
            <Route index element={<Homepage />} />
            <Route path="product" element={<Product />} />
            <Route path="pricing" element={<Pricing />} />
            <Route path="login" element={<Login />} />
            <Route
              path="app"
              element={
                <ProtectedRoute>
                  <AppLayout />
                </ProtectedRoute>
              }
            >
              <Route index element={<Navigate replace to="cities" />}></Route>
              <Route path="cities" element={<CityList />}></Route>
              <Route path="cities/:id" element={<City />}></Route>
              <Route path="countries" element={<CountryList />}></Route>
              <Route path="form" element={<Form />}></Route>
            </Route>
            <Route path="*" element={<PageNotFound />} />
          </Routes>
        </BrowserRouter>
      </CitiesProvider>
    </AuthProvider>
  );
}
```

```jsx
// CitiesContext.jsx
const getCity = useCallback(
  async function getCity(id) {
    if (Number(id) === currentCity.id) return;
    try {
      dispatch({ type: "LOADING", payload: true });
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      const data = await res.json();
      dispatch({ type: "CURRENT_CITY_LOADED", payload: data });
    } catch (error) {
      console.log(`There was an error getting the city! ${error.message}`);
      dispatch({ type: "ERROR", payload: error.message });
    }
  },
  [currentCity.id]
);
```

```jsx
// City.jsx
function City() {

  const { id } = useParams();
  const {getCity, currentCity, isLoading} = useCities();

  useEffect(() => {
    getCity(id)
  }, [id, getCity])

  const { cityName, emoji, date, notes } = currentCity;

  if(isLoading) return <Spinner/ >


  return (
    etc...
  )
```

### Code Splitting (Optimizing Bundle Size) - Suspense Intro

Bundle:

- JS file containing the entire apps code. Downloading it will load the entire app at once. This gives us an SPA

Bundle Size:

- Amount of JS the user needs to download

Code Splitting:

- Splitting the bundle into multiple smaller parts so it can be downloaded over tim as parts are needed "lazy loading"

A common practice is to split apps at the page level, but can ALSO be done with COMPONENTS.

To lazy load/code split pages/components we store the result of calling the react `lazy` function which takes a callback that return the JS dynamic `import` function with the path to the page/component. Then vite automatically takes care of the code splitting for us. We wrap the Routes in the react `Suspense` component giving it a fallback prop which is the desired fallback component (in this case a spinner). Then we can see the differences in bundle sizes (commented out) from one big chunks to a bunch of smaller ones. Additionally after the initial slower load of a page, when the use navigate back the chunk is already loaded and very fast.

```jsx
// App.jsx
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import { lazy, Suspense } from "react";

// import Homepage from "./pages/Homepage";
// import Product from "./pages/Product";
// import Pricing from "./pages/Pricing";
// import Login from "./pages/Login";
// import AppLayout from "./pages/AppLayout";
// import PageNotFound from "./pages/PageNotFound";

// dist/assets/index-35c0d3a4.css   30.30 kB │ gzip:   5.06 kB
// dist/assets/index-b39bb755.js   524.79 kB │

const Homepage = lazy(() => import("./pages/Homepage"));
const Product = lazy(() => import("./pages/Product"));
const Pricing = lazy(() => import("./pages/Pricing"));
const Login = lazy(() => import("./pages/Login"));
const AppLayout = lazy(() => import("./pages/AppLayout"));
const PageNotFound = lazy(() => import("./pages/PageNotFound"));

// dist/assets/Logo-515b84ce.css             0.03 kB │ gzip:   0.05 kB
// dist/assets/Product-cf1be470.css          0.47 kB │ gzip:   0.27 kB
// dist/assets/Homepage-b9276e6f.css         0.51 kB │ gzip:   0.30 kB
// dist/assets/PageNav-d3c5d403.css          0.51 kB │ gzip:   0.28 kB
// dist/assets/Login-c4b57744.css            0.57 kB │ gzip:   0.30 kB
// dist/assets/AppLayout-43ea7ec8.css        1.91 kB │ gzip:   0.70 kB
// dist/assets/index-300e441a.css           26.42 kB │ gzip:   4.33 kB
// dist/assets/Product.module-02d70b80.js    0.06 kB │ gzip:   0.07 kB
// dist/assets/Logo-f77965b9.js              0.21 kB │ gzip:   0.19 kB
// dist/assets/PageNotFound-ca165fda.js      0.23 kB │ gzip:   0.20 kB
// dist/assets/PageNav-1aded4e1.js           0.49 kB │ gzip:   0.27 kB
// dist/assets/Pricing-fd88bf0a.js           0.64 kB │ gzip:   0.41 kB
// dist/assets/Homepage-d05efb60.js          0.67 kB │ gzip:   0.41 kB
// dist/assets/Product-2430de33.js           0.85 kB │ gzip:   0.48 kB
// dist/assets/Login-a197655a.js             1.03 kB │ gzip:   0.54 kB
// dist/assets/AppLayout-8ad6e6e9.js       156.92 kB │ gzip:  46.14 kB
// dist/assets/index-458ae3a8.js           366.39 kB │ gzip: 102.23 kB

import CityList from "./components/CityList";
import CountryList from "./components/CountryList";
import City from "./components/City";
import Form from "./components/Form";
import ProtectedRoute from "./pages/ProtectedRoute";
import SpinnerFullPage from "./components/SpinnerFullPage";

import { CitiesProvider } from "./contexts/CitiesContext";
import { AuthProvider } from "./contexts/AuthContext";

function App() {
  return (
    <AuthProvider>
      <CitiesProvider>
        <BrowserRouter>
          <Suspense fallback={<SpinnerFullPage />}>
            <Routes>
              <Route index element={<Homepage />} />
              <Route path="product" element={<Product />} />
              <Route path="pricing" element={<Pricing />} />
              <Route path="login" element={<Login />} />
              <Route
                path="app"
                element={
                  <ProtectedRoute>
                    <AppLayout />
                  </ProtectedRoute>
                }
              >
                <Route index element={<Navigate replace to="cities" />}></Route>
                <Route path="cities" element={<CityList />}></Route>
                <Route path="cities/:id" element={<City />}></Route>
                <Route path="countries" element={<CountryList />}></Route>
                <Route path="form" element={<Form />}></Route>
              </Route>
              <Route path="*" element={<PageNotFound />} />
            </Routes>
          </Suspense>
        </BrowserRouter>
      </CitiesProvider>
    </AuthProvider>
  );
}

export default App;
```

### Don't Optimize Prematurely

DONT:

- Dont wrap everything in memo, useMemo or useCallback.
- Don't break contexts out into smaller contexts only holding single values to prevent rerenders for subscribed components.
- Premature optimization will cause a performance hit and make code unreadable

DO:

- Use profiler to identify bottleneck
- Watch for laggy behavior
- Memoize expensive rerenders & calculations
- Optimize contexts that change often with many subscribers
- Memoize ctx value and children
- Implement code splitting/lazy loading

### useEffect Rules & Best Practices

Dependency Array:

- Every state variable, prop AND context value used inside the effect must be added to the dep array
- All "reactive" values also must be included. This means for example if a helper function utilizes some state variables and is called inside the useEffect, then we must add that function as a dep because it uses "reactive" values
- NEVER ignore the `exhaustive-deps` ESLint rule
- DONT use objects or arrays as dependencies because a new object will not equal the old object even though they look the same

**Removing unnecessary dependencies**

Removing helper function dependencies:

- Move the function inside the effect
- If you need the function in multiple places, memoize it
- If the function doesn't reference any reactive values, move it outside the component

Removing object dependencies:

- Include only the properties you need (Have to be primitives)
- Use same strategy for functions (Move or memoize)

Other strategies:

- If you have multiple related reactive values, consider using a `useReducer`
- You dont need to include `setState` or `dispatch` functions in the array. React guarantess they are stable across renders

**When NOT to use effects**

Effects should be used as a last resort, when no other solution makes sense. The core team calls them an "escape hatch" and have been over used in many scenarios

3 Overuse Cases:

1. User Events

- Should be handled with event handlers even if they create side effects

2. Fetching data on component mount (Not good for production apps)

- Use React Query instead or another library

3. Synchronizing state changes with one another

- Creates multiple rerenders which is problematic (Use derived state/event handlers instead)


### State Updates Based On Other State

This is usually not desirable but the alternative is putting the repeated calculations across multiple setState functions. The issue here is that when one of the dependencies is updated it causes a rerender, which also causes the effect to run, which ALSO causes a second render. So this usually is avoided when possible.

```jsx
useEffect(() => {
    setDuration((number * sets * speed) / 60 + (sets - 1) * durationBreak)
  }, [number, sets, speed, durationBreak])
```

### Helper Functions In Effects

When we use helper functions inside a useEffect, we need to add it to the dependency array. But it causes issues. When we use the `handleInc/Dec` functions we are causing an update to the duration state which causes a rerender of the component and recreates the `playSound` function and because its a dependency of the useEffect, the effect gets runs again. this causes the duration to be set again, but using the current value which actually hasn't changed. This is why we see it flicker to the increased value, then immediately flash back to the original value.

If we walk through the possible solves...

We could move it outside the component:
- Wouldn't work because its a reactive value and depends on other state from props

We could move in inside the effect: 
- Wouldn't work because then we would lose scope of it and couldn't use it inside our event handlers

We could memoize it:
- This also breaks down quickly as described below

```jsx
import { memo, useEffect, useState } from 'react';
import clickSound from './ClickSound.m4a';

function Calculator({ workouts, allowSound }) {
  const [number, setNumber] = useState(workouts.at(0).numExercises);
  const [sets, setSets] = useState(3);
  const [speed, setSpeed] = useState(90);
  const [durationBreak, setDurationBreak] = useState(5);

  const [duration, setDuration] = useState(0)
  // const duration = (number * sets * speed) / 60 + (sets - 1) * durationBreak;

  const playSound = function () {
    if (!allowSound) return;
    const sound = new Audio(clickSound);
    sound.play();
  };

  useEffect(() => {
    setDuration((number * sets * speed) / 60 + (sets - 1) * durationBreak);
    playSound();
  }, [number, sets, speed, durationBreak, playSound])

  function handleInc() {
    setDuration(dur => Math.floor(dur) + 1);
    playSound();
  }

  function handleDec() {
    setDuration(dur => dur > 1 ? Math.floor(dur) - 1 : 0)
    playSound();
  }


  const mins = Math.floor(duration);
  const seconds = (duration - mins) * 60;

  return (
    <>
      <form>
        <div>
          <label>Type of workout</label>
          <select value={number} onChange={(e) => setNumber(+e.target.value)}>
            {workouts.map((workout) => (
              <option value={workout.numExercises} key={workout.name}>
                {workout.name} ({workout.numExercises} exercises)
              </option>
            ))}
          </select>
        </div>
        <div>
          <label>How many sets?</label>
          <input
            type='range'
            min='1'
            max='5'
            value={sets}
            onChange={(e) => setSets(e.target.value)}
          />
          <span>{sets}</span>
        </div>
        <div>
          <label>How fast are you?</label>
          <input
            type='range'
            min='30'
            max='180'
            step='30'
            value={speed}
            onChange={(e) => setSpeed(e.target.value)}
          />
          <span>{speed} sec/exercise</span>
        </div>
        <div>
          <label>Break length</label>
          <input
            type='range'
            min='1'
            max='10'
            value={durationBreak}
            onChange={(e) => setDurationBreak(e.target.value)}
          />
          <span>{durationBreak} minutes/break</span>
        </div>
      </form>
      <section>
        <button onClick={handleDec}>–</button>
        <p>
          {mins < 10 && '0'}
          {mins}:{seconds < 10 && '0'}
          {seconds}
        </p>
        <button onClick={handleInc}>+</button>
      </section>
    </>
  );
}

export default memo(Calculator);

```

Memoized:

This however creates a weird bug where when the `allowedSound` value changes. Because its part of the dependency array of useCallback the function gets recreated, and because the function itself is a dependency of the useEffect, then the playSound function is called and we hear a sound when turning sound on. Additionally if we inc/dec a bunch of times then turn the sound off/on then our duration gets reset because of the same reasons above
```jsx
const playSound = useCallback(
    function () {
      if (!allowSound) return;
      const sound = new Audio(clickSound);
      sound.play();
    },
    [allowSound]
  );

  useEffect(() => {
    setDuration((number * sets * speed) / 60 + (sets - 1) * durationBreak);
    playSound();
  }, [number, sets, speed, durationBreak, playSound])
```


The proper solve is to move separate logic between effects. Give them single responsibility. So we let the one effect deal with setting the duration and the other with playing the sound,


```jsx
unction Calculator({ workouts, allowSound }) {
  const [number, setNumber] = useState(workouts.at(0).numExercises);
  const [sets, setSets] = useState(3);
  const [speed, setSpeed] = useState(90);
  const [durationBreak, setDurationBreak] = useState(5);

  const [duration, setDuration] = useState(0);
  // const duration = (number * sets * speed) / 60 + (sets - 1) * durationBreak;

  // const playSound = useCallback(
  //   function () {
  //     if (!allowSound) return;
  //     const sound = new Audio(clickSound);
  //     sound.play();
  //   },
  //   [allowSound]
  // );

  useEffect(() => {
    const playSound = function () {
      if (!allowSound) return;
      const sound = new Audio(clickSound);
      sound.play();
    };
    playSound();
  }, [duration, allowSound]);

  useEffect(() => {
    setDuration((number * sets * speed) / 60 + (sets - 1) * durationBreak);
  }, [number, sets, speed, durationBreak]);

  function handleInc() {
    setDuration((dur) => Math.floor(dur) + 1);
  }

  function handleDec() {
    setDuration((dur) => (dur > 1 ? Math.floor(dur) - 1 : 0));
  }

  return (
    etc...
  )
```

## Redux & Redux Toolkit (Thunks)

Redux:

- 3rd party library to manage state
- Global state stored in a "store" using "actions" (similar to useReducer)
- Global store updated -> All consuming components rerender 
- Conceptually similar to combining Context with usereducer
- 2 ways to write Redux: 1) Classic Redux 2) Redux Tool Kit
- Goal is to make state update logic separate from the rest of the app

Do you need Redux?
- Historically it was used in most React apps for global state, but now there are many alternatives. Many apps dont need it unless they have a lot of global state

1. It can be hard to learn so its good to know how it works
2. You WILL encounter it so you should understand it
3. Some apps require it or something similar

With useReducer we have and event handler which -> dispatches an action with a payload -> to the reducer which -> takes the current state along with the action payload and returns -> a new state which -> causes a rerender.

Redux is similar but:
Event handler which -> Action creator function (An optional convention which makes writing actions easier and keeps them in a central place) which -> goes through a dispatcher to -> the store where one or more reducer live which are pure functions and take the current state along with the action payload (usually one reducer per app feature) and returns -> a new state which -> causes a rerender.


### Action Creators Intro

`npm i redux`

Using `createStore` is deprecated but Redux team recommends still for learning

The following code is pretty self-explanatory:

We can create multiple reducer for our store. We do this by, for each one defining their initial state to be passed in as defaults and switch over the action.type as usual performing necessary state updates as needed. Then we use the `combineReducers` function to combine them into a rootReducer that we pass to `createStore` to create our store with our different reducers.

In our actionCreators we dispatch our desired actions.

**Note**
Our action types that we define as per Redux team convention Ex.) `customer/createCustomer` have nothihng to do with Redux. They could be called anything and Redux under the hood figures out which actions are related to what reducer. Our naming has nothing to do with it.

```js
// store.js
const { createStore, combineReducers } = require("redux");

const initialStateAccount = {
  balance: 0,
  loan: 0,
  loanPurpose: "",
};

const initialStateCustomer = {
  fullName: "",
  nationalID: "",
  createAt: "",
};

function accountReducer(state = initialStateAccount, action) {
  switch (action.type) {
    case "account/deposit":
      return { ...state, balance: state.balance + action.payload };
    case "account/withdraw":
      return { ...state, balance: state.balance - action.payload };
    case "account/requestLoan":
      if (state.loan > 0) return state;
      return {
        ...state,
        loan: action.payload.amount,
        loanPurpose: action.payload.purpose,
        balance: state.balance + action.payload.amount,
      };
    case "account/payLoan":
      return {
        ...state,
        loan: 0,
        loanPurpose: "",
        balance: state.balance - state.loan,
      };
    default:
      return state;
  }
}

function customerReducer(state = initialStateCustomer, action) {
  switch (action.type) {
    case "customer/createCustomer":
      return {
        ...state,
        fullName: action.payload.fullName,
        nationalID: action.payload.nationalID,
        createdAt: action.payload.createdAt,
      };
    case "customer/updateName":
      return { ...state, fullName: action.payload };
    default:
      return state;
  }
}

const rootReducer = combineReducers({
  account: accountReducer,
  customer: customerReducer
})

const store = createStore(rootReducer);

function deposit(amount) {
  return { type: "account/deposit", payload: amount };
}

function withdraw(amount) {
  return { type: "account/withdraw", payload: amount };
}

function requestLoan(amount, purpose) {
  return {
    type: "account/requestLoan",
    payload: { amount, purpose },
  };
}

function payLoan() {
  return { type: "account/payLoan" };
}

store.dispatch(deposit(1000))
store.dispatch(withdraw(200))
console.log(store.getState());

store.dispatch(requestLoan(200, "Small loan"))
console.log(store.getState());

store.dispatch(payLoan())
console.log(store.getState());

function createCustomer(fullName, nationalID) {
  return {
    type: "customer/createCustomer",
    payload: { fullName, nationalID, createdAt: new Date().toISOString() },
  };
}

function updateName(fullName) {
  return { type: "customer/updateName", payload: fullName };
}

store.dispatch(createCustomer('Cam Remesz', 666));
console.log(store.getState());
store.dispatch(updateName('Cameron Remesz'));
console.log(store.getState());

```

### File Structure: State Slices

For a more organized file structure we create `feature` FOLDERS ex.) `customers` and store all customer related files in there including components. There we would add our `customerSlice.js`. Then do that for all our other pieces of state related to redux. Then at the top level of our `src` folder we have our `store.js` file and as we can see is a default export. We import it into our `index.js` and wrap the entire app in the `Provider`

```js
// accountSlice.js
const initialStateAccount = {
  balance: 0,
  loan: 0,
  loanPurpose: "",
};

export default function accountReducer(state = initialStateAccount, action) {
  switch (action.type) {
    case "account/deposit":
      return { ...state, balance: state.balance + action.payload };
    case "account/withdraw":
      return { ...state, balance: state.balance - action.payload };
    case "account/requestLoan":
      if (state.loan > 0) return state;
      return {
        ...state,
        loan: action.payload.amount,
        loanPurpose: action.payload.purpose,
        balance: state.balance + action.payload.amount,
      };
    case "account/payLoan":
      return {
        ...state,
        loan: 0,
        loanPurpose: "",
        balance: state.balance - state.loan,
      };
    default:
      return state;
  }
}

export function deposit(amount) {
  return { type: "account/deposit", payload: amount };
}

export function withdraw(amount) {
  return { type: "account/withdraw", payload: amount };
}

export function requestLoan(amount, purpose) {
  return {
    type: "account/requestLoan",
    payload: { amount, purpose },
  };
}

export function payLoan() {
  return { type: "account/payLoan" };
}
```

```js
// customerSlice.js
const initialStateCustomer = {
  fullName: "",
  nationalID: "",
  createAt: "",
};

export default function customerReducer(state = initialStateCustomer, action) {
  switch (action.type) {
    case "customer/createCustomer":
      return {
        ...state,
        fullName: action.payload.fullName,
        nationalID: action.payload.nationalID,
        createdAt: action.payload.createdAt,
      };
    case "customer/updateName":
      return { ...state, fullName: action.payload };
    default:
      return state;
  }
}

export function createCustomer(fullName, nationalID) {
  return {
    type: "customer/createCustomer",
    payload: { fullName, nationalID, createdAt: new Date().toISOString() },
  };
}

export function updateName(fullName) {
  return { type: "customer/updateName", payload: fullName };
}

```
We run:
`npm i react-redux`

Then our whole application has access to the store. We will also now access the dispatch method through the `react-redux` library which will give us the `store.dispatch` as a hook as seen further on.

```js
// store.js
import React from "react";
import ReactDOM from "react-dom/client";
import "./index.css";
import App from "./App";
import { Provider } from "react-redux";

import store from "./store";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);

```

Then in any component we can use `useSelector` to access our desired part of the store.
Here obviously the name is blank because we haven't set it but this creates a subscription to the store and when the state in the store changes then this component will rerender.

```jsx
import { useSelector } from "react-redux";

function Customer() {
  const fullName = useSelector((store) => store.customer.fullName)

  return <h2>👋 Welcome, {fullName}</h2>;
}

export default Customer;

```

Here we use the `useDispatch` hook to get access to the dispatch method .

```jsx
import { useState } from "react";
import { useDispatch } from "react-redux";

import { createCustomer } from "./customerSlice";

function CreateCustomer() {
  const [fullName, setFullName] = useState("");
  const [nationalID, setNationalID] = useState("");

  const dispatch = useDispatch();

  function handleClick() {
    if(!fullName || !nationalID) return;
    dispatch(createCustomer(fullName, nationalID))
  }

  return (
    <div>
      <h2>Create new customer</h2>
      <div className="inputs">
        <div>
          <label>Customer full name</label>
          <input
            value={fullName}
            onChange={(e) => setFullName(e.target.value)}
          />
        </div>
        <div>
          <label>National ID</label>
          <input
            value={nationalID}
            onChange={(e) => setNationalID(e.target.value)}
          />
        </div>
        <button onClick={handleClick}>Create new customer</button>
      </div>
    </div>
  );
}

export default CreateCustomer;

```

```jsx
import { useState } from "react";
import { useDispatch, useSelector } from "react-redux";

import { deposit, withdraw, requestLoan, payLoan } from "./accountSlice";
import store from "../../store";

function AccountOperations() {
  const [depositAmount, setDepositAmount] = useState("");
  const [withdrawalAmount, setWithdrawalAmount] = useState("");
  const [loanAmount, setLoanAmount] = useState("");
  const [loanPurpose, setLoanPurpose] = useState("");
  const [currency, setCurrency] = useState("USD");

  const dispatch = useDispatch();
  const hasLoan = useSelector((state) => !!state.account.loan);

  function handleDeposit() {
    if (!depositAmount) return;
    dispatch(deposit(depositAmount));
    setDepositAmount("");
    console.log(store.getState());
  }

  function handleWithdrawal() {
    if (!withdrawalAmount) return;
    dispatch(withdraw(withdrawalAmount));
    setWithdrawalAmount("");
    console.log(store.getState());
  }

  function handleRequestLoan() {
    if (!loanAmount || !loanPurpose) return;
    dispatch(requestLoan(loanAmount, loanPurpose));
    setLoanAmount("");
    setLoanPurpose("");
    console.log(store.getState());
  }

  function handlePayLoan() {
    if (!hasLoan) return;
    dispatch(payLoan());
    console.log(store.getState());
  }

  return (
    <div>
      <h2>Your account operations</h2>
      <div className="inputs">
        <div>
          <label>Deposit</label>
          <input
            type="number"
            value={depositAmount}
            onChange={(e) => setDepositAmount(+e.target.value)}
          />
          <select
            value={currency}
            onChange={(e) => setCurrency(e.target.value)}
          >
            <option value="USD">US Dollar</option>
            <option value="EUR">Euro</option>
            <option value="GBP">British Pound</option>
          </select>

          <button onClick={handleDeposit}>Deposit {depositAmount}</button>
        </div>

        <div>
          <label>Withdraw</label>
          <input
            type="number"
            value={withdrawalAmount}
            onChange={(e) => setWithdrawalAmount(+e.target.value)}
          />
          <button onClick={handleWithdrawal}>
            Withdraw {withdrawalAmount}
          </button>
        </div>

        <div>
          <label>Request loan</label>
          <input
            type="number"
            value={loanAmount}
            onChange={(e) => setLoanAmount(+e.target.value)}
            placeholder="Loan amount"
          />
          <input
            value={loanPurpose}
            onChange={(e) => setLoanPurpose(e.target.value)}
            placeholder="Loan purpose"
          />
          <button onClick={handleRequestLoan}>Request loan</button>
        </div>

        <div>
          <span>Pay back $X</span>
          <button onClick={handlePayLoan}>Pay loan</button>
        </div>
      </div>
    </div>
  );
}

export default AccountOperations;

```

### Redux Middleware & Thunks

Where do we make async calls with Redux?
- The store holds reducers which are pure functions without side effects, so not in the store
- Fetching data in components and then dispatching would work but this is not ideal. We want to keep them free from data fetching
- We want data fetching all in one place...
- MIDDLEWARE is the answer. It will sit between the dispatched action and the store
- This is perfect for async code (API calls, timers, logging, any side effects)
- We CAN write ourselves, but its easier to use a 3rd party library
- Most common one is called Thunk


3 Steps to "Effect" middleware:
- Install middleware
- Apply to store
- Use in action creator functions

1. 
`npm i redux-thunk`

2. Apply to store by passing a second argument to the `createStore` function which is the `applyMiddleware` function, passing in our imported middleware

```jsx
import accountReducer from "./features/accounts/accountSlice";
import customerReducer from "./features/customers/customerSlice";

import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from "redux-thunk";

const rootReducer = combineReducers({
  account: accountReducer,
  customer: customerReducer
})

const store = createStore(rootReducer, applyMiddleware(thunk));

export default store;
```

Then we are going to get the currency and use it in the middleware to make an API call to convert it

```jsx
// AccountOperations.js
function handleDeposit() {
    if (!depositAmount) return;
    dispatch(deposit(depositAmount, currency));
    setDepositAmount("");
    setCurrency("USD")
    console.log(store.getState());
  }
```

3. In our action creator function we make allow a regular synchronous dispatch to the store if no conversion is needed, otherewise we make the API call.

We do this by RETURNING a function that gets the `dispatch` and `getState` functions from the store as arguments. We perform our API call as usual then call the `dispatch` function with our desired value to put in the store.

```jsx
export function deposit(amount, currency) {
  if(currency === "USD") return { type: "account/deposit", payload: amount };

  return async (dispatch, getState) => {
    // API Call
    const res = await fetch(`https://api.frankfurter.app/latest?amount=${amount}&from=${currency}&to=USD`);
    const data = await res.json();
    console.log(data);
    const convertedAmount = data.rates.USD

    // Dispatch action
     dispatch({ type: "account/deposit", payload: convertedAmount })
  }
}
```


### Redux Dev Tools

If not installed as an extension in chrome install them

Then add to your project with:

`npm i redux-devtool-extension`

Then we wrap the `applyMiddleware` function with the `composeWithDevTools` function

```js
import accountReducer from "./features/accounts/accountSlice";
import customerReducer from "./features/customers/customerSlice";

import { createStore, combineReducers, applyMiddleware } from 'redux';
import thunk from "redux-thunk";
import { composeWithDevTools } from "redux-devtools-extension";

const rootReducer = combineReducers({
  account: accountReducer,
  customer: customerReducer
})

const store = createStore(rootReducer, composeWithDevTools(applyMiddleware(thunk)));

export default store;
```

Then we can see our actions dispatched in the Redux tab in devtools as well as the states and their updates.

The best feature is that you can jump between states and see them visually, or they have a slider that can be played, or slide it between them.

Additionally you can manually dispatch actions in the devtools as well

### Modern Redux (Redux Tool Kit - RTK)

Redux Tool Kit
- Recommended by the Redux team
- Forces us to use best practices
- Can use classic redux and RTK together "IF needed"
- Write a lot less code to achieve the same results

3 Hightlights:
- Write code that "Looks like" it mutates state inside reducers (Using a library called Immer which converts our code back to non-mutating) This allows for complex state to be easily managed
- Automatically creates action creators from our reducers, which CAN create extra work for us in some instances
- Automatic setup of Thunk middleware & Redux devtools


`npm i @reduxjs/toolkit`

The store with RTK (much simpler):
```js
// store.js
import accountReducer from "./features/accounts/accountSlice";
import customerReducer from "./features/customers/customerSlice";

import { configureStore } from "@reduxjs/toolkit";

const store = configureStore({
  reducer: { account: accountReducer, customer: customerReducer },
});

export default store;

```

The concept of slices is baked into RTK. So now we get a `createSlice` function that we pass a config object to build our slice. We give it:
- A name
- The initial state
- Our reducers
  - The reducers are now seemingly mutating state directly so we dont need to copy the old state or return a new state, we directly assign new values to the state

The reducers in RTK by default only accept one value as the payload. If we want more then we need to use prepare function before the reducer to define a multi-value payload as seen in `requestLoan`.


```js
import { createSlice } from "@reduxjs/toolkit";

const initialState = {
  balance: 0,
  loan: 0,
  loanPurpose: "",
  isLoading: false,
};

const accountSlice = createSlice({
  name: "account",
  initialState,
  reducers: {
    deposit(state, action) {
      state.balance += action.payload;
    },
    withdraw(state, action) {
      state.balance -= action.payload;
    },
    requestLoan: {
      prepare(amount, purpose) {
        return {
          payload: { amount, purpose },
        };
      },
      reducer(state, action) {
        if (state.loan > 0) return;
        state.loan = action.payload.amount;
        state.loanPurpose = action.payload.purpose;
        state.balance += action.payload.amount;
      },
    },
    payLoan(state, action) {
      state.balance -= state.loan;
      state.loan = 0;
      state.loanPurpose = "";
    },
  },
});

export const { deposit, withdraw, requestLoan, payLoan } = accountSlice.actions;

export default accountSlice.reducer;

```

Another footgun from being so opinionated is that updates to state now have to happen IN THE ORDER THEY SHOULD OCCUR. Notice below the example if the loan is set to 0 first, then the second line `state.balance -= state.loan;` would result in `0` being subtracted from the `state.balance`, so the balance wouldn't even change.
```js
payLoan(state, action) {
  state.loan = 0;
  state.balance -= state.loan;
  state.loanPurpose = "";
},
```

### Redux vs Context API

Context + useReducer:
- Built into React
- Easy to setup single context
- Additional state "slice" requires a new context for each (leads to provider hell in App.js)
- No mechanism for async ops (Should be done here anyway)
- Performance optimization is a pain
- Only simple React devtools

Redux:
- 3rd party library (larger bundle size)
- More work to set up initially
- Easy to create new slices
- Supports middleware for async ops (Thunks built in)
- Performance optimization out of the box
- Excellent dev tools

Usage:

Generic advice - Use context for small apps, redux for large ones 

Context + useReducer:
- Good with values that don't change often
- Solve prop drilling
- Manage state in a smaller sub-tree of the app (compound component pattern)

Redux:
- Global UI state that changes often (Shopping cart, current tabs, filts, search) because it is automatically optimized for this
- Complex state with nested objects/arrays (RTK allows you to mutate state)
- Not super common for UI state (Redux has started to fall out of favor)

Good advice - Use whats best for your app based on your needs and make an educated decision

## Professional React 

## Professional Setup

### Eslint Config

`npm i eslint vite-plugin-eslint eslint-config-react-app ---save-dev`

We need to the eslint config for react we just installed extend eslint

```json
//  .eslintrc.json
{
  "extends": "react-app"
}
```

And also add the eslint plugin to the plugin array

```js
// vite.config.js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import eslint from 'vite-plugin-eslint'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), eslint],
})

```

AND... for whatever reason, I need to add an empty `jsconfig.json` at the top level to make a shit ton of react errors go away??????

### Application Planning

#### For smaller apps
1. Break desired UI in components
2. Build the static version (no state)
3. Think about state MGMT & data flow

#### For larger real world apps
1. Gather application requirements & features
2. Divide the app into pages
  - Think about overall and page level UI
  - Break UI into components
  - Design and build static version
3. Divide the app into feature categories
  - For each feature category think about state MGMT & data flow
4. Tech decisions (libraries)

#### App Requirements
- Simple app to order one or more pizzas from a menu
- No login required
- Pizza menu can change (should be loaded from an API)
- Users can add pizzas to a cart
- Ordering requires name, phone number and address
- GPS should be provided to make delivery easier
- Users can mark orders as priority which adds 20% to the total cart price
- Orders made by sending a POST to the API
- No payment processing, paid upon delivery
- Each placed order gets a unique ID so users can look up order status
- Users can mark order as priority even AFTER an order has been placed

#### Features & Pages

Feature Categories
1. User
2. Menu
3. Cart
4. Order

Pages
1. Home Page `/` 
2. Pizza Menu `/menu`
3. Pizza Cart `/cart`
4. Placing New Order `/order/new`
5. Looking Up An Order `/order/:id`

#### State MGMT
Feature categories usually map well to state slices

1. User -> Global UI (No accounts so stays in app)
2. Menu -> Global Remote (Menu fetched from API)
3. Cart -> Global UI (No need for an API, stored in app)
4. Order -> Global Remote (fetched & submitted to API)

Technology Decisions (Stack):
Routing -> React Router
Styling -> Tailwind
Remote State -> React Router
UI State -> Redux

### File Structure

Feature based structures are nice for bigger projects.

- features
  - cart
  - menu
  - order
  - services
  - ui
  - user
  - utils

We could also add other folders with things like contexts, hooks or even pages. The desired outcome of this folder structure is to prevent a structure that is broken up into way topo many small sections that it becomes really confusing to navigate the code base

## React Router With Data Loading

### React Router v6.4

With the new version of React Router, if we want to access the new powerful APIs like loaders, data fetching etc. we need to use a bit different syntax(Sort of looks like angular). We use the `createBrowserRouter` function to define an array of route objects. Then we pass the returned value of that function (`router`) as a prop to the `RouterProvider`

Create a sort of shell, we can create a parent route that takes no path property (So it gets rendered no matter what, in react router these are called 'Layout Routes') Then we add the children property and any child path will render the corresponding component. Keep in mind you have to use the `Outlet` component from `react-router-dom` to tell react where to render the children

```jsx
import { RouterProvider, createBrowserRouter } from "react-router-dom"
import Home from "./features/ui/Home"
import Menu from "./features/menu/Menu"
import Cart from "./features/cart/Cart"
import CreateOrder from "./features/order/CreateOrder"
import Order from "./features/order/Order"

const router = createBrowserRouter([
  {
    element: <AppLayout />,
    children:  [{
      path: '/',
      element: <Home />
    },
    {
      path: '/menu',
      element: <Menu />
    },
    {
      path: '/cart',
      element: <Cart/>
    },
    {
      path: '/order/new',
      element: <CreateOrder />
    },
    {
      path: '/order/:id',
      element: <Order />
    }]
  },
  
])


function App() {
  return (
    <RouterProvider router={router}>

    </RouterProvider>
  )
}

export default App

```

```jsx
// AppLayout.jsx
import { Outlet } from "react-router-dom"

import Header from "./Header"
import CartOverview from "../cart/CartOverview"

function AppLayout() {
  return (
    <div>
      <Header />

      <main>
        <Outlet />
      </main>

      <CartOverview />
    </div>
  )
}

export default AppLayout

```

### Fetching Data - Loaders

1. Create Loader
2. Provide Loader
3. Provide data to the page

Can be placed anywhere but convention is to place it in the file of that page.


We have a file in our services related to all our data fetching functions for a part of the app
```js
// apiRestaurant.js
const API_URL = 'https://react-fast-pizza-api.onrender.com/api';

export async function getMenu() {
  const res = await fetch(`${API_URL}/menu`);

  // fetch won't throw error on 400 errors (e.g. when URL is wrong), so we need to do it manually. This will then go into the catch block, where the message is set
  if (!res.ok) throw Error('Failed getting menu');

  const { data } = await res.json();
  return data;
}
```

Then we create our loader inside the file where we need it then export it. Then we import the loader and provide it for the component we want in `App.jsx` and then we can get the value from `useLoaderData` and react router then knows what loader we passed in. This way instead of using an effect to wait for the component to mount and have an initial render, the data is being fetched at the same time the data is being rendered.

```jsx
// Menu.jsx
import { useLoaderData } from "react-router-dom";
import { getMenu } from "../services/apiRestaurant";
import MenuItem from "./MenuItem";

function Menu() {
  const menu = useLoaderData();

  console.log(menu);

  return (
    <ul>
      {
        menu.map(zah => <MenuItem key={zah.id} pizza={zah}/>)
      }
    </ul>
  );
}

export async function loader() {
  return await getMenu();
}

export default Menu;

```

```jsx
// App.js
import { loader as menuLoader } from "./features/menu/Menu"

const router = createBrowserRouter([
  {
    element: <AppLayout />,
    children:  [
      {
      path: '/',
      element: <Home />
      },
      {
        path: '/menu',
        element: <Menu />,
        loader: menuLoader,
      },
      etc...
    ]
}
])
```

### Loading States

React Router now provides a `useNavigation` hook that will update a global state if any of the components are loading, idle or submitting. So we can use this in our top level layout component to display a loading animation.

```jsx
import { Outlet, useNavigation } from "react-router-dom";

import Header from "./Header";
import CartOverview from "../cart/CartOverview";
import Loading from "./Loading";

function AppLayout() {
  const navigation = useNavigation();
  const isLoading = navigation.state === 'loading'
  console.log(navigation);
  // formAction: undefined
  // formData: undefined
  // formEncType: undefined
  // formMethod: undefined
  // json: undefined
  // location: {pathname: '/menu', search: '', hash: '', state: null, key: 'mgcjhsqx'}
  // state: "loading"
  // text: undefined
  return (
    <div className="layout">
      {isLoading && <Loading />}

      <Header />

      <main>
        <Outlet />
      </main>

      <CartOverview />
    </div>
  );
}

export default AppLayout;

```

### Errors

React router now provides an `errorElement` property where we can pass an element to be shown if an error occurs. Any error that occurs, whether it be due to a component rendering, or due to failed http request, the error will bubble up until it hits a route where the `errorElement` is defined to be handled. In the error component we can then use the `useRouteError` hook to get the error info and display it. Here we handle the instance of a fetch request going wrong in the menu component. as well as at the  `AppLayout` level if navigating to an unknown route for ex.) `/sdfgdsfghfg`. The router automatically displays a nicely formatted message based on the `error.data` or `error.message`.

```jsx
const router = createBrowserRouter([
  {
    element: <AppLayout />,
    errorElement: <Error/>,
    children:  [{
      path: '/',
      element: <Home />
    },
    {
      path: '/menu',
      errorElement: <Error/>,
      element: <Menu />,
      loader: menuLoader,
    }]
  }
])  
```

```jsx
// Error.jsx
import { useNavigate, useRouteError } from 'react-router-dom';

function Error() {
  const navigate = useNavigate();
  const error = useRouteError();
  console.log(error);

  return (
    <div>
      <h1>Something went wrong 😢</h1>
      <p>{error.data || error.message}</p>
      <button onClick={() => navigate(-1)}>&larr; Go back</button>
    </div>
  );
}

export default Error;
```

If we need our http functions to access parameters we can't just use the `useParams` hook because it would be used outside a component. React Router has thought of this so the loader function has a `params` property which you can destructure and access the params as defined in the router.

```jsx
// Orders.jsx
export async function loader({params}) {
  return await getOrder(params.orderId)
}
```

### React Router Actions

Where loaders are used to read data, actions are used to write data.

Actions follow the same pattern as loaders, where you defined your action in the same component file where you want it. Then you add it to the action property of that route. Then any form submission from that route will be handled by the action. You use the `Form` component from react router and specify the method `POST, UPDATE, PATCH` but `GET` is not allowed here. The `Form` component handles a bunch of stuff under the hood whereas before we would have had to create state variables and control all the inputs as well as having a handler function that called preventDefault.

The action function intercepts the request from the form submission and we get access to it. So we call the `request.formData()` to get the formData then we call the `Object.fromEntries()` function from JS to pull out our input data. At this point we can format the data as needed and perform any http requests as desired like in our `createOrder` function. This returns the new order from the api with its created id so the we can call the `redirect` method provided by react router and navigate to the newly created order.

Additionally we can perform some custom validation/error handling with help of the action. We can check the phone number to make sure its formatted properly and if not we can return early with an errors object. Anything we return from the action we can access through the `useActionData` hook and then render some validation feedback as needed. 

```jsx
// App.jsx
{
  path: '/order/new',
  element: <CreateOrder />,
  action: createOrderAction
},
```

```jsx
// CreateOrder.jsx
import { Form, redirect, useActionData, useNavigation } from "react-router-dom";
import { createOrder } from "../services/apiRestaurant";

// https://uibakery.io/regex-library/phone-number
const isValidPhone = (str) =>
  /^\+?\d{1,4}?[-.\s]?\(?\d{1,3}?\)?[-.\s]?\d{1,4}[-.\s]?\d{1,4}[-.\s]?\d{1,9}$/.test(
    str
  );

function CreateOrder() {
  const cart = fakeCart;

  const navigation = useNavigation();
  const isSubmitting = navigation.state === "submitting";
  const formErrors = useActionData();

  return (
    <div>
      <h2>Ready to order? Lets go!</h2>

      <Form method="POST">
        <div>
          <label>First Name</label>
          <input type="text" name="customer" required />
        </div>

        <div>
          <label>Phone number</label>
          <div>
            <input type="tel" name="phone" required />
          </div>
          {formErrors?.phone && <p>{formErrors.phone}</p> }
        </div>

        <div>
          <label>Address</label>
          <div>
            <input type="text" name="address" required />
          </div>
        </div>

        <div>
          <input
            type="checkbox"
            name="priority"
            id="priority"
          />
          <label htmlFor="priority">Want to yo give your order priority?</label>
        </div>

        <div>
          <input type="hidden" name="cart" value={JSON.stringify(cart)} />
          <button disabled={isSubmitting}>
            {isSubmitting ? "Placing Order..." : "Order Now"}
          </button>
        </div>
      </Form>
    </div>
  );
}

export async function action({ request }) {
  const formData = await request.formData();
  const data = Object.fromEntries(formData);

  const order = {
    ...data,
    cart: JSON.parse(data.cart),
    priority: data.priority === "on",
  };

  const errors = {};

  if (!isValidPhone(order.phone))
    errors.phone =
      "Please give us your correct phone number. We might new to contact you.";

  if (Object.keys(errors).length > 0) return errors;

  const newOrder = await createOrder(order);

  return redirect(`/order/${newOrder.id}`);
}

export default CreateOrder;

```

## Redux w/ Advanced React Router

Redux Selectors:

A common pattern for accessing DERIVED state from the store instead of calculating it again and again at the component level, is to make reusable selectors that are in the slice. This can come at a performance cost in larger app, so there are patterns, libraries to look into that help with this if need be.

**NOTE**
Selectors that do not require any argument to be passed in will only need the function definition passed to the `useSelector` hook when being called. If a selector does require arguments then, when defining the function you will NEED TO USE CURRYING Ex.) `getQuantityById`. Then you can call the selector with the parameter. See below for examples.

```jsx
// cartSlice.js
import { createSlice } from '@reduxjs/toolkit';

const initialState = {
  cart: [],
};

const cartSlice = createSlice({
  name: 'cart',
  initialState,
  reducers: {
    addItem(state, action) {
      state.cart = [...state.cart, action.payload];
    },
    deleteItem(state, action) {
      state.cart = state.cart.filter(
        (pizza) => pizza.id !== action.payload
      );
    },
    increaseItemQty(state, action) {
      const pizza = state.cart.find(
        (pizza) => pizza.id === action.payload
      );
      pizza.quantity++;
      pizza.totalPrice = pizza.quantity * pizza.unitPrice;
    },
    decreaseItemQty(state, action) {
      const pizza = state.cart.find(
        (pizza) => pizza.id === action.payload
      );
      pizza.quantity--;
      pizza.totalPrice = pizza.quantity * pizza.unitPrice;
    },
    clearCart(state) {
      state.cart = [];
    },
  },
});

export const {
  addItem,
  deleteItem,
  increaseItemQty,
  decreaseItemQty,
  clearCart,
} = cartSlice.actions;

export default cartSlice.reducer;

export const getCart = (state) => state.cart.cart;

export const getTotalCartQuantity = (state) =>
  state.cart.cart.reduce((acc, cur) => acc + cur.quantity, 0);


//  ======== NOTE FROM ABOVE ========


export const getTotalCartPrice = (state) =>
  state.cart.cart.reduce((acc, cur) => acc + cur.unitPrice, 0);

export const getQuantityById = (id) => (state) =>
  state.cart.cart.find((item) => item.id === id)?.quantity ?? 0;

```

```jsx
// MenuItem.jsc
function MenuItem({ pizza }) {
  const { id, name, unitPrice, ingredients, soldOut, imageUrl } = pizza;

  const dispatch = useDispatch();
  const currentQuantity = useSelector(getQuantityById(id));

  const isInCart = currentQuantity > 0;

  etc...
}
```

## Styled Components

### Crash Course/Basics

Styled components actually just leverage an uncommon feature of ES6 called [tagged template literals](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals). This allows for the template literal to be passed to a function which is the `styled.{htmlElement}` function. To use them we declare a const variable uppercased because they are going to be components. Then we use the `styled.` function with the type of html element appended. Then immediately following we use template literals and define our css as per usual. Under the hood styled components will ensure these styles are specifically scoped to that exact component with a unique class name. The components can then be used in our render logic and have the ability of regular components with all expected event handlers built in. To styled the top level functional component Ex.) `App` We create a styled component with convention being `Styled` + `ComponentName` and use a `styled.div` at the top level. 


```jsx
import styled from "styled-components"

const H1 = styled.h1`
  font-size: 30px;
  font-weight: 600;
  background-color: yellow;
`

const Button = styled.button`
  font-size: 1.4rem;
  padding: 1.2rem 1.6rem;
  font-weight: 500;
  border: none;
  border-radius: 7px;
  background-color: purple;
  color: white;
  cursor: pointer;
  margin: 20px;
`

const Input = styled.input`
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: .8rem 1.2rem; 
`

const StyledApp = styled.div`
  background-color: orangered;
  padding: 20px;

`

function App() {
  return (
    <StyledApp>
      <H1>Ohhhhhh buddy</H1>

      <Button onClick={() => alert(`Feckin hell`)}>Check In</Button>
      <Button onClick={() => alert(`Feckin hell`)}>Check Out</Button>

      <Input type='number' placeholder='Number of guests'></Input>
    </StyledApp>
  )
}

export default App

```


### Global Styles

For global styles we can put a `globalstyles.js` in our styles folder and export a styled component just like any other it with all our css inside the template literals, including all valid css such as css variables defined with `:root`. 

Then to use it we need to make it a self closing sibling of the app.

```jsx
function App() {
  return (
    <>
      <GlobalStyles />
      <StyledApp>
        <H1>Ohhhhhh buddy</H1>

        <Button onClick={() => alert(`Feckin hell`)}>Check In</Button>
        <Button onClick={() => alert(`Feckin hell`)}>Check Out</Button>

        <Input type="number" placeholder="Number of guests"></Input>
      </StyledApp>
    </>
  );
}
```

Additionally its convention to extract the styled components into their own folders. You can if you'd like keep some local styled components. Ex.) `StyledApp` as seen above could just stay instead of extracting to their own files, because they aren't really going to be reused.

```jsx
// ./ui/Input.jsx
import styled from "styled-components";

const Input = styled.input`
  border: 1px solid var(--color-grey-300);
  background-color: var(--color-grey-0);
  border-radius: var(--border-radius-sm);
  box-shadow: var(--shadow-sm);
  padding: 0.8rem 1.2rem;
`;

export default Input;
```

### Styled Component Props & css function

We can pass custom defined props to our styled components. Then in our components since we are just technically using template literals, we can also use JS `${}` in them. This allows us to create reusable components with styling logic. Below the commented out `type` prop we defined could be used to determine styling, but all components when rendered would ALL be `h1`'s. This is bad for SEO and accessibility, so the library has a solve foe this using the `as` prop that we pass in with the element type and then will render the correct html element in the DOM.

Additionally, the library supplies a `css` helper for template literals. It is designed to be used for when calling functions in template literals....but here we use it to just get access to css syntax highlighting.

```jsx
function App() {
  return (
    <>
      <GlobalStyles />
      <StyledApp>
        <Heading as='h1'>The Wild Oasis</Heading>
        {/* <Heading type='h1'>The Wild Oasis</Heading> */}
        <Heading as='h2'>Check in and out</Heading>

        <Button onClick={() => alert(`Feckin hell`)}>Check In</Button>
        <Button onClick={() => alert(`Feckin hell`)}>Check Out</Button>

        <Heading as='h3'>Form</Heading>
        <Input type="number" placeholder="Number of guests"></Input>
      </StyledApp>
    </>
  );
}
```


```jsx
import styled, { css } from "styled-components";

const Heading = styled.h1`
  ${props => props.as === 'h1' && css`
  font-size: 3rem;
  font-weight: 600;
  `}

  // ${props => props.type === 'h2' && css`
  // font-size: 2rem;
  // font-weight: 600;
  // `}

  ${props => props.as === 'h2' && css`
  font-size: 2rem;
  font-weight: 600;
  `}

  ${props => props.as === 'h3' && css`
  font-size: 1rem;
  font-weight: 500;
  `}
  
  line-height: 1.4;
`;

export default Heading;

```