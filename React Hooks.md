* [Introducing Hooks](#Introducing_Hooks)
* [State Hook](#State_Hook)
* [Effect Hook](#Effect_Hook)
* [Rules of Hooks](#Rules_of_Hooks)
* [Building Your Own Hooks](#Building_Your_Own_Hooks)
* [Other Hooks](#Other_Hooks)


# Introducing_Hooks

*Hooks* are a new addition in React 16.8. They let you use state and other React features without writing a class.

```js
import React, { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**Note** *React 16.8.0 is the first release to support Hooks. When upgrading, don’t forget to update all packages, including React DOM. React Native supports Hooks since the 0.59 release of React Native.*

## No Breaking Changes

**Note that Hooks are:**

* **Completely opt-in.** You can try Hooks in a few components without rewriting any existing code. But you don’t have to learn or use Hooks right now if you don’t want to.
* **100% backwards-compatible.** Hooks don’t contain any breaking changes.
* **Available now.** Hooks are now available with the release of v16.8.0.

**There are no plans to remove classes from React.**

# State_Hook

`useState` returns a pair: the current state value and a function that lets you update it. You can call this function from an event handler or somewhere else. It’s similar to this.setState in a class, except it doesn’t merge the old and new state together.


**Note** that unlike `this.state`, the state here doesn’t have to be an object — although it can be if you want. The initial state argument is only used during the first render.

## Declaring multiple state variables
**You can use the State Hook more than once in a single component:**

```js
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```

##  But what is a Hook?

Hooks are functions that let you “hook into” React state and lifecycle features from function components. ***Hooks don’t work inside classes — they let you use React without classes.***

**[hook into something]** to become connected to something such as a computer network or phone system.


**When would I use a Hook?** If you write a function component and realize you need to add some state to it, previously you had to convert it to a class. Now you can use a **Hook** inside the existing function component.

**What do we pass to useState as an argument?** The only argument to the useState() Hook is the initial state. Unlike with classes, the state doesn’t have to be an object. We can keep a number or a string if that’s all we need.

*If we wanted to store two different values in state, we would call **useState( )** twice.*

## Reading State

When we want to display the current count in a class, we read this.state.count:
```html
  <p>You clicked {this.state.count} times</p>
  ```
In a function, we can use count directly:

```html
  <p>You clicked {count} times</p>
  ```

## Updating State

In a class, we need to call this.setState() to update the count state:

```html
  <button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
  </button>
  ```

In a function, we already have setCount and count as variables so we don’t need this:

```html
  <button onClick={() => setCount(count + 1)}>
    Click me
  </button>
  ```

# Effect_Hook

You’ve likely performed data fetching, subscriptions, or manually changing the DOM from React components before. We call these operations “side effects” (or “effects” for short) because they can affect other components and can’t be done during rendering.

The Effect Hook, `useEffect`, adds the ability to perform side effects from a function component. It serves the same purpose as `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount` in React **classes**, but unified into a **single** API. 

For example, this component sets the document title after React updates the DOM:
```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

Let’s compare how **classes** and **Hooks** used in side effects.

## Example Using Classes

In React class components, the render method itself shouldn’t cause side effects. It would be too early — we typically want to perform our effects after React has updated the DOM.

This is why in React classes, we put side effects into componentDidMount and componentDidUpdate. . Coming back to our example, here is a React counter class component that updates the document title right after React makes changes to the DOM:
```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

Note how **we have to duplicate the code between these two lifecycle methods in class.**

This is because in many cases we want to perform the same side effect regardless of whether the component just mounted, or if it has been updated.

## Example Using Hooks

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

**What does useEffect do?** By using this Hook, you tell React that your component needs to do something after render. 

**Why is useEffect called inside a component?** Placing useEffect inside the component lets us access the count state variable (or any props) right from the effect

**Does useEffect run after every render?** Yes! By default, it runs both after the first render and after every update.

## Tips for Using Effects

We’ll continue this page with an in-depth look at some aspects of useEffect that experienced React users will likely be curious about. ***If you are not experienced React user you can skip this and come back when you need.***

**Tip: Use Multiple Effects to Separate Concerns**

One of the Motivations for Hooks is that **class lifecycle** methods often contain unrelated logic, but related logic gets broken up into several methods. Look at next **example**:

```js
class FriendStatusWithCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0, isOnline: null };
    this.handleStatusChange = this.handleStatusChange.bind(this);
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentWillUnmount() {
    ChatAPI.unsubscribeFromFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }

  handleStatusChange(status) {
    this.setState({
      isOnline: status.isOnline
    });
  }
  // ...
  ```

  See, how can Hooks solve this problem?! Just like you can **use the State Hook** more than once, you can also use several **effects**.

  ```js
  function FriendStatusWithCounter(props) {
  const [count, setCount] = useState(0);
  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  const [isOnline, setIsOnline] = useState(null);
  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }

    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });
  // ...
}
```

Hooks let us split the code based on what it is doing rather than a lifecycle method name. ***React will apply every effect used by the component, in the order they were specified.***

## Tip: Optimizing Performance by Skipping Effects

In some cases, applying the **effect** after every render might create a performance problem. In **class components**, we can solve this by writing an extra comparison with `prevProps` or `prevState` inside `componentDidUpdate`:

```js
componentDidUpdate(prevProps, prevState) {
  if (prevState.count !== this.state.count) {
    document.title = `You clicked ${this.state.count} times`;
  }
}
```

This requirement is built into the `useEffect` Hook API. You can tell React to *skip* applying an effect if certain values haven’t changed between re-renders. To do so, pass an array as an **optional** second argument to `useEffect`:

```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

In the example above, we pass [count] as the second argument. If the count is 5, and then our component re-renders with count still equal to 5, React will compare [5] from the previous render and [5] from the next render so React would skip the effect.

When we render with count updated to 6, React will compare the items in the [5] array from the previous render to items in the [6] array from the next render. This time, React will re-apply the effect because 5 !== 6. ***If there are multiple items in the array, React will re-run the effect even if just one of them is different.***

# Rules_of_Hooks

Hooks are JavaScript functions, but they impose two additional rules:

* Only call Hooks at the top level. Don’t call Hooks inside loops, conditions, or nested functions.
* Only call Hooks from React function components. Don’t call Hooks from regular JavaScript functions. (There is just one other valid place to call Hooks — your own custom Hooks. We’ll learn about them in a moment.)

# Building_Your_Own_Hooks
Sometimes, we want to reuse some stateful logic between components. Traditionally, there were two popular solutions to this problem: [higher-order components](https://reactjs.org/docs/higher-order-components.htm) and [render props](https://reactjs.org/docs/render-props.html). Custom Hooks let you do this, but without adding more components to your tree.

***We can use the same logic in more components***

First, we’ll extract this logic into a custom Hook called useFriendStatus:

```js
import React, { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```

It takes friendID as an argument, and returns whether our friend is online.

Now we can use it from both components:

```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```

```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```

***Each call to a Hook has a completely isolated state — so you can even use the same custom Hook twice in one component.***

If a function’s name starts with ”use” and it calls other Hooks, we say it is a custom Hook, like call `useSomething`

# Other_Hooks

There are a few less commonly used built-in Hooks that you might find useful. For example, useContext lets you subscribe to React context without introducing nesting:

```js
function Example() {
  const locale = useContext(LocaleContext);
  const theme = useContext(ThemeContext);
  // ...
}
```

And useReducer lets you manage local state of complex components with a reducer:

```js
function Todos() {
  const [todos, dispatch] = useReducer(todosReducer);
  // ...

  ```


**Reference:** [React Hooks](https://reactjs.org/docs/hooks-intro.html)