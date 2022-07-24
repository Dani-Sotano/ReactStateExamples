# State Management

React offers different approaches to handling state in your application. A core principle of React is the composition of components. Components give us great flexibility and offer better structuring especially when creating big websites with a lot of different parts. But creating a nested structure can also provide a lot of complexity. Especially when considering that the same variables are used and updated in different components. 

There are different ways on how state is travelling through different components. I want to present you the possibilities and advantages and disadvantages of each option.

## Using props to pass state up and down nested components

The first way how the state is passed up and down the component hierarchy is just by passing props down and functions, that will then pop back to the higher level component. Okay what?

```jsx
function Main() {
    let count = 1;
    function addToCount(value) {
        count = count+value;
	      render();
    }
    return (
        <FirstLevelComponent 
              value={count} 
              update={addToCount}>
        </FirstLevelComponent>
    )
}
```

The Main component has the count value and a method to update the count variable. The Main Component accesses the FirstLevelComponent, and passes two props: a value `value={count}` and a function `update={addToCount}`. 

But the function is not called inside of this component, it is passed to the `FirstLevelComponent.` The FirstLevelComponent can access the values via the props variable `props.value` and `props.update`.

The update function is called, whenever the user clicks the button to submit the value. This is handled inside the handleOnClick method.

```jsx
function FirstLevelComponent(props){
  let value = "";

  const handleOnClick = (event) => {
      event.preventDefault();
      props.update(value)
      let element = document.querySelector("input")
      element.value = "";
  }

  const handleChange = (event) => {
      value = Number(event.target.value);
  };

  return (
      <div>
          <div>Your base value is {props.value}</div>
          <input onChange={handleChange} />
          <button onClick={handleOnClick}> Update </button>
      </div>
  )
}
```

Inside Main.update(value) the count value is updated with the input value. After the update of the value, we have to trigger the update of the component, so we call render();

But React also offers other methods to update the DOM automatically after the state has updated: `useState` and `useReduce`.

## useState

UseState is a React Hook. But what is a React hook? 
We use React components to generate a structured hierarchy and reusability of different parts of the user interface. We want components to be stateless so that every input generates the same output without any side effects. But for most web applications we do need to store user input inside a component to work with it. React hooks help us to keep our components stateless, by separating the state concern from the concern of the component. Equal to how I want to reuse my components I can reuse my hook and pass it to other components. A hook can be stateful and can manage the side effects. 

Hooks also have access to certain React capabilities like managing the state. For example, it starts rerendering the UI when the state is updated. 

You can create state by calling `React.useState(initialValue)` and set an `initial value` or `‘’`. The useState method will return an array containing the state variable and a method to update the state. You can use them now every time you want to update or access the state. You can pass them as props and no matter at which level you are, the method will update the state from everywhere and rerender the component where the hook method was defined (Here it renders the Main component and all child components).

```jsx
function Main() {
    const [count, setCount] = React.useState(1);

    function addToCount(value) {
        setCount(count + Number(value))
    }

    return (
        <FirstLevelComponent
            value={count}
            update={addToCount}>
        </FirstLevelComponent>
    )
};
```

Use state is good to use when you have a simple component that does not require a lot of values to be passed to other components. If a component gets more complex, we can use useReducer.

## useReducer

UseReducer is similar to useState, but instead of defining and updating individual elements you can define and update several values, objects, and arrays inside of a state object. This helps especially when values depend on each other and it is easier to update them at once in one object or whenever you have a high number of values that need to be updated you can aggregate the values in one state object.

A reducer looks similar to useState. You can create a reducer by providing it with a reducer function, initial arguments, and initial values. In return like in state you get an array containing the current state object and a derived dispatch function. 

```jsx
const [state, dispatch] = useReducer(reducer, {count: 1})
```

So what do I have to provide? The reducer function is a function that defines what you want to do after a state update. Imagine you want to decrease or increase a number, depending on a button that is clicked. So you would have two actions: INCREASE and DECREASE. So the reducer functions have to handle both events. 

The reducer function expects two arguments, the current state, and an action argument. The action argument is defined by you. If you want to use the increase or decrease functionality you have to define the action and the value inside this action object.

`action = {type: ACTION.INCREASE, value: 5}`

You can now define what should happen inside the reducer function, depending on the action and the value that is submitted. 

```jsx
function reducer(state, action) {
    if (action.type === ACTIONS.INCREASE) {
        return { ...state, count: state.count + Number(action.value) }
    } else if (action.type === ACTIONS.DECREASE) {
        return { ...state, count: state.count - Number(action.value) }
    }
}
```

You return a shallow copy of the new object while updating the copy state object on your behalf.

But the reducer function is not the one that you have to call. You call the dispatch function, pass in the action object and React.useReducer then calls the reduce method, passes the current state and triggers the rerendering.

```jsx
dispatch({ type: ACTIONS.INCREASE, value: value })
```

# Solving props drilling

Sometimes you will have a very deeply nested and complex data structure, where you have to pass values down several layers of components to find yourself sending a value back up by using a function. During the process of passing the element down all the nested components, most of them don't need the data but have to handle it anyway.

So how can you solve this problem: there are two options Context API and Component Composition.

## Context API

Imagine a piece of land where four houses are placed with enclosing fences. There is a beautiful area around it with a lot of flowers, but you cant reach it because, you have a fence around your house. Someone can pass the flowers over the fence to you, but you always need someone else to help you reach the flowers. Imagine now, that the fence is only a line on the ground. You know your area, but you can reach the other area easily by yourself. 

That is context API, it is a Wrapper around your house (component), which is a separate piece of land (component) but you actually have access to the land. 

The wrapper contains the global state for all of the included components. You have to define the state once and then you can export it and each subcomponent inside the wrapper can access it.

`export const CountContext = React.createContext()`

All Components inside the `CountContext.Provider` can access the variables defined in `value={{}}`.

```html
<CountContext.Provider value={{count: count, update: update}}>
    <FirstLevelComponent />
</CountContext.Provider>
```

How to access it? Very easy:

```jsx
function FirstLevelComponent(props) {
    const { count, update  } = React.useContext(CountContext);
....
```

When using Context API we have to be careful when reusing a component. As we only implicitly pass props to the component via the Provider, and define the context to be used inside the component itself, we can not use the component outside of the defined provider. Furthermore, whenever the context is updated, all components nested inside the Provider are updated. This is no problem for smaller apps, but bigger applications can suffer from performance issues. 

## Component Composition

How can we maintain component reusability without falling into props drilling? There is a way to lift components up to the root level and pass the data directly into the component that requires it, without losing nesting functionalities. 

```jsx
<Wrapper >
    <FirstLevelComponent count={count} update={update}/>
</Wrapper>
```

We want our components to be inside the wrapper, but the wrapper does not need our state values. We can pass the values directly to the `FirstLevelComponent.` The FirstLevelComponent is still nested under the Wrapper Component. To enable the nesting, the element has to be passed as children through the Wrapper. 

```jsx
function Wrapper({children}){
    return (
        <div>
            {children}
        </div>
    )
}
```

## Conclusion

In summary, there are several ways to pass variables and functions to different component levels. But only useState and useReducer trigger a rendering of the components. Both re-render the component itself and all subsequent child components. To reduce the number of components rendered after a state change, we always need to set the state to the lowest possible level. Especially if our application is large and complex, contains many state updates, or is deeply nested. 

On another note, a blog post about React State without considering Redux is probably not complete. I've worked with Redux in the past, but that was before hooks were a thing. I've been thinking about when to use Redux, and if you have a large, complex codebase where state is changed very frequently and the changes themselves can be very complex, it's probably worth considering Redux.
