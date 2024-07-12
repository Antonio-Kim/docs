## State Management

There are cases where different states, such as email, an array of items, has to be shared between different components. Like many functionalities in React, there are many different ways to implement statement management: **props drilling**, **React Context**, and **Redux**. Since Redux is the most popular method to implement state management, it will only be discussed here.

### Redux

To install Redux, use the following command:

```bash
npm i @reduxjs/toolkit react-redux
```

Redux uses centralized immutable object called **store** to save states and functions. It's very similar to useReducer where there's some initial state and based on dispatch, it alters the states. Redux often is paired with Redux Toolkit to minimize the configuration of redux. Each area of state in Redux is called **slice**, and they contain some state and reducer to change state. Here's an example of how you can setup a slice in redux:

```tsx
export const someSlice = createSlice({
  name: 'someFeature',
  initialState,
  reducers: {
    someAction: (state) => {
      state.someValue = 'something';
    },
    anotherAction: (state) => {
      state.someOtherValue = 'something else';
    },
  },
});
```

**createSlice** takes in an object with slice's name, initial state, and reducers that modifies the state of the slice. **createSlice** then needs to be declared inside main slice repository called **configureStore** in order to be accessible. Each action often takes in state and actual type of implementation of action to modify the state. The type of implementation often used is PayLoadAction from `@reduxjs/toolkit`.

```tsx
export const store = configureStore({
  reducer: {
    someFeature: someSlice.reducer,
  },
});
```

- **Providing store to components**: In order for store to be accessible to other component, the **Provider** with store parameter has to be wrapped outside the components that will use the states.

```tsx
<Provider store={store}>
  <Header />
  <Content />
</Provider>
```

- **Accessing store slice's state**: Note that this is about accessing the state. Modifying will be discussed next. To access the state, **useSelector** hook from React redux is used:

```tsx
const someValue = useSelectr((state: RootState) => state.someFeature.someValue);
```

- **Modifying the store slice's state**: React Redux also provides functionality to use reducers from the slice.

```tsx
const dispatch = useDispatch();

return (
  <button onClick={() => dispatch(someSlice.actions.someAction())}>
    Some button
  </button>
);
```

Here's the gist of how it works: dispatch create an object called **action** that is sent to the store for information about the state and how much/what should it be updated by. The **action** used in Redux is called **PayloadAction** and has predefined types that gets sent to Redux Store:

```tsx
type PayloadAction<Payload, Type extends string = string, Meta = undefined> = {
  type: Type;
  payload: Payload;
  error?: boolean;
  meta?: Meta;
};
```

The type in PayloadAction is instructing store which slice and which reducers to use based on this information that's being sent. The payload is object that is used by the reducer as parameter for that function. The actual implementat should be defined inside the reducer, but this PayloadAction is merely an information that is being sent to the store for change.
