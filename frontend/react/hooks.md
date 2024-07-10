## Using hooks in React

- **useEffect**: creates a side effect when component is loaded, and when state inside the second parameter changes. The second parameter of useEffect is called Dependency Array
- **useReducer**: Another type of state management than useState. Takes two parameters: function that changes when it is invoked, and initial state. It then is assigned to a tuple where the first parameter is the state name that will be used within a function, and dispatch function that uses the reducer function when declared. Not really used for simple state assignments but useful for large and complex state managet. (Very similar to Redux?)
- **useRef**: useful for variables that requires its value to persist for lifetime of a component. If the component re-renders, the value remains constant. Commonly used to access HTML elements in React imperatively.
- **useMemo**: often used to pass expensive computational cost to another component.
- **useCallback**: memoizes function so that it doesn't recreate on each render. Often used to prevent unnecessary re-renders of child components. Just like memo and effect, useCallback has dependency array. React has a function called memo which prevents unnecessary re-render. It wraps the component and memoizes the result for given set of props

useMemo is for values, and useCallback is for values and functions, respectively, and are often used for performance optimization.

#### When does components re-render?

Usually when state changes, or useEffect is invoked. It renders both the component and child components. This might sound expensive but it does not: the DOM will only be updated after a re-render if the virtual DOM changes.
