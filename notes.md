***try not to rm -rf these ones...***
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
  }, [])

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

  return { movies, isLoading, error }
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
  step: 1
}

function reducer(state, action) {
  console.log(state);
  console.log(action);
  switch (action.type) {
    case "dec":
      return {...state, count: state.count - state.step};
    case "inc":
      return {...state, count: state.count + state.step};
    case "setCount":
      return {...state, count: action.payload};
    case "setStep":
      return {...state, step: action.payload};
    case "reset":
      return initialState;
    default:
      throw new Error('Unknown action!');
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
    dispatch({ type: "inc"});
  };

  const defineCount = function (e) {
    dispatch({ type: "setCount", payload: Number(e.target.value) });
  };

  const defineStep = function (e) {
    dispatch({  type: 'setStep', payload:  Number(e.target.value)})
  };

  const reset = function () {
    dispatch({ type: "reset"});
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
