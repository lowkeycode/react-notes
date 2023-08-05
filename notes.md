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

Because components are just JS functions we can write JS in them. We can also use JS inside JSX in the return. To use JS we need to "Enter JS mode" which is done using curly brackets  `{}`. Then inside the brackets we need to use JS EXPRESSIONS. So this means ternaries, short-circuiting, variables or return an array using array methods. Statements are NOT VALID in the JSX return. Ex.) if/else statements or switch statements. Styles can also be determined with JS mode inside the style attribute with a ternary.

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

  if (!isOpen)
  return (
    <footer>
      We're not open.
    </footer>
  )

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









# ===========================================
# ============== CONTINUE HERE ==============
# ===========================================

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
- Allow us to use non-visual logic
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

After adding a `jsconfig.json` then removing it. I started getting typescript errors everywhere. I implemented a basic jsconfig.json as per VS Code docs after 2 1/2hrs of struggling to figure out what the fuck went wrong.

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

Then we can use another hook called the `getMeMyFuckingQueryStrings` hook, and destructure as needed in any component that is currently rendered.

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

Or we can use it to go back. (I assume its based onthe browser history api)

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

In the app when the `city:id` route is rendered from clicking on a city in the city list the id is updated in the url and the city component mounts. When the id changes the useEffect dependent on that id will run and call `getCity(id)`. We can see in the context that getCity updates the state, causing a rerender of the provider, which will cause a rerender of the city component, which will call `getCity` AGAIN resulting in another state update and an infinite loop.

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
const getCity = useCallback(async function getCity(id) {
    if(Number(id) === currentCity.id) return;
    try {
      dispatch({ type: "LOADING", payload: true });
      const res = await fetch(`${BASE_URL}/cities/${id}`);
      const data = await res.json();
      dispatch({ type: "CURRENT_CITY_LOADED", payload: data });
    } catch (error) {
      console.log(`There was an error getting the city! ${error.message}`);
      dispatch({ type: "ERROR", payload: error.message });
    }
  }, [currentCity.id])
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

To lazy load/code split pages/components we store the result of xalling the react `lazy` function which takes a callback that return the JS dynamic `import` function with the path to the page/component. Then vite automatically takes care of the code splitting for us. We wrap the Routes in the react `Suspense` component giving it a fallback prop which is the desired fallback component (in this case a spinner). Then we can see the differences in bundle sizes (commented out) from one big chunks to a bunch of smaller ones. Additionally after the initial slower load of a page, when the use navigate back the chunk is already loaded and very fast.

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
-  Don't break contexts out into smaller contexts only holding single values to prevent rerenders for subscribed components.
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

