# State Management in Pure React

## State Overview

- Job of React is to take your application state and turn it into DOM nodes
    - It takes your data, applies rules, and determines what is shown on your page. And takes care of updating when your state changes.
    - This is different from legacy web development, when the state of the application was often included as part of the markup

- There are many kinds of state.
    - Model data
        - The nouns in your application
    - View/UI state
        - Are those nouns sorted in ascending/descending order?
    - Session state
        - Is the user logged in?
    - Communication
        - Are we fetching those nouns from the server?
    - Location
        - Where are we in the application? Which nouns are we looking at?

## Class Based State

- `setState`
    - React will batch multiple `setState` calls, figure out the result and then efficiently make the change.
    - It will essentially merge all the objects passed to `setState` and create a new state object
    - You can also pass a function to `setState` which will behave a bit differently:
        - In this case, you're not really batching the `setState` calls - the functions will run in sequence. So the state parameter passed to this function will hold the most recent state.
        - Also allows you to include more robust logic within the function (you can pass in props as the second argument if you need to use props in your logic)
        - This function can be written outside the component and just passed to the `setState` call, which is very useful for both reusability and unit testing
        - You can return undefined from this function and it won't update state
    - `setState` also accepts a second argument, which is a callback that will run after state updates
        - Note: this callback function does not take accept any arguments

- Dos and Don'ts
    - DON'T use state for prop derivations
        - Do this in the render method or break into helper methods/functions
    - DON'T use state for things that aren't included in render method
    - DO use sensible defaults (i.e. keep object types consistent)

## Hooks State
- `useState`
    - When using state updater function, it will batch updates unless you pass in a function
    - When you pass in function argument to the state updater , it only accepts argument for state (no props argument)
    - This function argument behaves similarly as in class component (i.e. state updates aren't batched), with one notable exception:
        - You have to return a value - if you return undefined, state will be updated with that undefined value

- `useEffect`
    - Mainly used for side effects (i.e. AJAX, console.log, etc.)
    - When you want something to happen in response to something else
    - Should always include dependency array as second argument with variables that are included in the effect
    - You can return a function from `useEffect` if you need to clean up anything before component unmounts 

- `useRef`
    - Good way to persist values across renders
    - i.e. if you need to compare the previous state and current state

## Reducers
- Reducer function
    - Function that takes two arguments (current state and action) and returns a brand new state
    - Allows us to separate state management from component implementation
    ```javascript
        // simplest form of reducer
        function reducer(state, action) {
            return state
        }
    ```

- `useReducer`
    ```javascript
        const [state, dispatch] = useReducer(reducer, initialState)
    ```
    - Dispatching actions works in a similar way to redux - i.e. you dispatch an object with `type` and `payload` properties (or whatever you want to name them, that's just the preferred convention)

- How can we stop a component from re-rendering if its props are still the same?
    - Wrap functional component with `React.memo`
        - Don't just throw everything in `React.memo` because (in some cases) you're doing unnecessary work - but if it's saving a lot of rerenders, go for it
    - `useCallback`
        - If you're declaring a function in the body of a functional component, it will create a new function on every render. But if you wrap the declaration with `useCallback`, it will maintain the same reference to the function across multiple renders, which can be handy when passing it as a prop to a child component
    - `useMemo`
        - Similar use case as `useCallback` but with object references

## Context
- Provides a way to pass props through the component tree without having to pass them manually through every level.
- Is it a substitute for redux? Maybe - but it won't provide many of the build in features that redux provides, such as middleware, performance enhancements, etc.


- `React.createContext()`
    - Returns object with `Provider` and `Consumer` components as properties
    - Pass a `value` prop to `Provider`, and children components can access that data by using the render prop pattern (child as a function) with the `Consumer` component.
        ```javascript
            <ExampleContext.Provider value={0}>
                <ExampleContext.Consumer>
                    {value => <p>{value}</p>}
                </ExampleContext.Consumer>
            </ExampleContext.Provider>
        ```
    - In a functional component, you can just use the `useContext` hook instead of the consumer component to access the context data.
        ```javascript
            const contextData = React.useContext(ExampleContext)
        ```
- One downside to context is that you lose performance optimizations from `React.memo` since you're reading from context instead of passing in props 

- Also makes testing more challenging, although there are workarounds (test context provider, HOC)

## Data Fetching
- In class based components we would use `componentDidMount` - for functional components, `useEffect` is the simplest way to fetch data
- It's pretty easy to pull out the fetching logic into a custom hook that also handles setting error/loading states and returning your response
    - `useReducer` could be handy in this case to handle switching between the different fetching/response/error states (or in any situation where you find your state getting more complicated)
- Hooks are great ways to separate reuseable logic (whether that be data fetching, parsing, side effects, etc.) from components. This allows you to handle your logic in a separate place while allowing your components to focus on rendering the UI.

## Thunks
- Reducers themselves don't have an idea of asynchrony
- redux-thunk is a common middleware to handle this issue
    - Major idea behind a thunk is that it's code to execute later
    - A thunk is just a function returned from another function 
    - In the action creators, instead of dispatching the action directly, we can use the thunk to dispatch the action once the async code finishes executing.

Example:
```javascript
    const useThunkReducer = (reducer, initialState) => {
        const [state, dispatch] = useReducer(reducer, initialState);

        const enhancedDispatch = action => {
            // using typeof for simplicity, could also use more robust isFunction check from lodash
            if (typeof action === 'function') {
                action(dispatch);
            } else {
                dispatch(action);
            }
        }

        return [state, enhancedDispatch];
    }
```

- We now have a way to write our async code that is completely separate from the reducer or component it's used in