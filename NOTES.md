# 1. Application State

React is just a view library, not an MVC framework, it takes your state into DOM elements. The react state management follows the rule of **figure out the minimal state representation of your app and compute everything else on demand**. For example, `fullName` can be calculated from `firstName` and `lastName`, so we don't need to put the `fullName` into the state.

State has many different pieces, such as model data, view/ui state, session state and communications state can all be one slice of the global state. Also, we can categorise the state into **long-lasting state** and **ephemeral state**.

# 2. React Component State

Quiz 1: what is the output

```js
state = { count: 0 };

this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });
this.setState({ count: this.state.count + 1 });

console.log(state); // ?
```

The answer is **0** because setState is **asynchronous**, and this is because react is trying to avoid unnecessary re-renders.

Quiz 2: what will be the next rendered count after we press the button

```js
class Counter extends Component {
  state = { count: 0 }

  increment3 = () => {
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
    this.setState({ count: this.state.count + 1 });
  }

  render() {
    <div>
      <p>{this.state.count}</p>
      <button onClick={this.increment3}>X 3<button>
    </div>
  }
}
```

The answer is **1**, because all the `setState` statement will be queued up the state changes, and during the execution of the three `setState` expression, the state will always reference to 0, so it is like running the 0 + 1 by 3 times.

Quiz 3: Same implementation except the setState part this time, we use the function

```js
class Counter extends Component {
  state = { count: 0 }

  increment3 = () => {
    this.setState(s => ({ count: s.count + 1 }));
    this.setState(s => ({ count: s.count + 1 }));
    this.setState(s => ({ count: s.count + 1 }));
  }

  render() {
    ...
  }
}
```

This time, it's **3**, because it takes the function each time and execute each of them.

## 2.1 State Patterns and anti-patterns

State should be considered as private data, and generally we don't want it to be modified by the outside components.

An alternative way of creating computed properties is using a getter

```js
class User extends Component {
  get fullName() {
    const { firstName, lastName } = this.props;
    return `${firstName} ${lastName}`;
  }

  render() {
    return <h1>{this.fullName}</h1>;
  }
}
```

## 2.2 State Architecture Pattern

### 2.2.1 Container Pattern

It split the components into two categories, the presentational components and the container components. Most of the components are class components because it's eaiser to leverage them to become container components.

The benefit we get from this split is the components are now easier to test.

### 2.2.2 HOC Pattern

```js
const withCount = WrappedComponent => class extends Component {
  state = {count: 0}
  increment = () => {
    this.setState({
      ...
    })
  }

  render() {
    return (
      <WrappedComponent
        count={this.state.count}
        increment={this.increment}
        {...this.props}
      />
    )
  }
}
```

The wrapped component will receive the props from outside, as well as the props from the wrapped class state.

If we have a state that will be comsumed in multiple places, we can extract it out and create a higher order component.

Notice that we can configure the wrapped component name by

```js
return class extends Component {
  ...
  static displayName = `WithPizzaCalculations(${WrappedComponent.displayName ||
      WrappedComponent.name})`;

  render() {
    ...
  }
}
```

so that the output will be `<WithPizzaCalculations(PizzaCalculator)>`.

### 2.2.3 Render Properties Pattern

```js
export default class WithCount extends Component {
  state = { count: 0 };

  increment = () => {
    this.setState({
      count: this.state.count + 1
    });
  };

  render() {
    return (
      <div className="WithCount">
        {this.props.render(this.state.count, this.increment)}
      </div>
    );
  }
}

// when we using WithCount
const Foo = () => (
  <WithCount
    render={(count, increment) => (
      <div>{count}</div>
      <button onClick={increment}>Add</button>
    )}
  />
)
```
