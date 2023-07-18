**_try not to rm -rf these ones..._**
👋 See ya later 1000 or so lines of notes...

# React

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

function PostProvider({children}) {
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
  )
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
  if(context === undefined) throw new Error('PostContext was used outside of the PostProvider');
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
        currentCity: action.payload
      };
    case "DELETE_CITY":
      return {
        ...state,
        cities: state.cities.filter((city) => city.id !== action.payload),
        isLoading: false,
        currentCity: {}
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
        cityContextError
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

function ProtectedRoute({children}) {
  const { isSignedIn } = useAuth();
  const navigate = useNavigate();

  useEffect(() => {
    if(!isSignedIn) navigate('/')
  }, [isSignedIn, navigate]);

  return isSignedIn ? children : null;
}

export default ProtectedRoute

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