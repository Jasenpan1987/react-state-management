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

# 3. Flux

Flux is the pattern to extract our state data into stores.

```js
// Action creators
export const updateNumberOfPeople = value => {
  AppDispatcher.dispatch({
    type: "UPDATE_NUMBER_OF_PEOPLE",
    value
  });
};

// Store
class ItemStore extends EventEmitter {
  constructor() {
    super();

    AppDispatcher.register(action => {
      if (action.type === "UPDATE_NUMBER_OF_PEOPLE") {
        ...
      }

      if (action.type === "UPDATE_SLICE_PER_PERSON") {
        ...
      }
    })
  }
}

// Dispatcher
const AppDispatcher = new Dispatcher
```

# 4. Redux

Redux is similar to flux pattern, but with one, immutable state tree. Redux has multiple reducers which takes an action and the current state returns the new state.

Redux is small, it only exposes the following 5 methods:

```js
import {
  applyMiddleware,
  createStore,
  combineReducers,
  bindActionCreators,
  compose
} from "redux";
```

Implement bindActionCreator

```js
const bindActionCreator = (action, dispatch) => (...args) =>
  dispatch(action(...args));

const addValue = bindActionCreator(add, store.dispatch);
```

Implement bindActionCreators

```js
const bindActionCreators = (actions, dispatch) =>
  Object.keys(actions).reduce((boundActions, key) => {
    boundActions[key] = bindActionCreator(actions[key], dispatch);
    return prev;
  }, {});

const errors = bindActionCreators(
  {
    set: setError,
    clear: clearError
  },
  store.dispatch
);
```

# 5. MobX

## 5.1 Computed Properties

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

var john = new Person("John", "Doe");
john.firstName; // John
john.lastName; // Doe
john.fullName; // John Doe
```

## 5.2 Decorator

Decorators are simply syntactic suggers for higher order functions. Decorators are functions.

```js
function readOnly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}
```

To use decorators

```js
class Person {
  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @readOnly
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

Decorator is not in the js yet, we need to use babel to transpile.

## 5.3 MobX

### 5.3.1 some basics

example 1

```js
const { computed, action, observable, autorun } = mobx;

class Person {
  @observable
  firstName;
  @observable
  lastName;

  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  @computed
  get fullName() {
    return this.firstName + " " + this.lastName;
  }
}

const person = new Person("Steve", "Kinney");
const quote = observable("Only JavaScript can prevent forest fires.");

const render = () => {
  document.body.innerText = person.fullName + " : " + quote;
};

autorun(render);

setTimeout(() => {
  person.firstName = "bar";
}, 2000);
```

- Obervable is a wrapper function, when the value inside Observable changes, the `autorun` gets triggered.
- Computed value will change when any of the value changes.
- `quote` won't trigger the `autorun` if we re-assign it, because inside of the Observable, it's a string, and string is immutable, but we can call `quote.set("fooo")`, that will triggers the `autorun`.

### 5.3.2 a simplified observable

```js
function onChange(oldValue, newValue) {
  console.log("Value Changed", { oldValue, newValue});
}

function Observable(value) {
  return {
    get() {return value}
    set(newValue) {
      onChange(this.get(), newValue);
      value = newValue;
    }
  }
}
```

Use Observable

```js
class Person {
  @observable
  firstName;
  @observable
  lastName;

  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }
}
```

is equivalent to

```js
function Person(firstName, lastName) {
  this.firstName;
  this.lastName;

  extendObservable(this, {
    firstName: firstName,
    lastName: lastName
  });
}
```

Where extendObservable can be implemented as

```js
function extendObservable(target, source) {
  source.keys().forEach(function(key) {
    const wrappedObservable = observable(source[key]);

    Object.defineProperty(target, key, {
      set: value.set,
      get: value.get
    });
  });
}
```

and the decorator can be defined as

```js
const observable = object => extendObservable(object, object);
```

### 5.3.3 Three component of mobx

1. Observable state
2. Actions
3. Derivations

   3.1. computed property

   3.2. Reactions (side effects)

## 5.4 MobX with arrays, objects and maps

- Object: observable({})
- Array: observable([])
- Maps: observable(new Map())

For Arrays, we can simply call the methods such as `push` and `splice`, the observable will pickup the changes and re-render the page just like calling other immutable methods. And Object is the same, we can do `obj.name = "foo"` and trigger the re-render.

Map is like object, except the key of maps can be anything, we can have function as a key, we can have object as a key, not just strings. We should use Map if we are going to be adding keys later on.

## 5.5 MobX with react

```js
@observer
class Counter extends Component {
  render() {
    const { counter } = this.props;
    return (
      <div>
        <h3>Counter: {counter.count}</h3>
        <button onClick={counter.increment}>Add</button>
        <button onClick={counter.decrement}>Reduce</button>
        <button onClick={counter.reset}>Reset</button>
      </div>
    );
  }
}

const Counter = observer(({ counter }) => {
  return (
    <div>
      <h3>Counter: {counter.count}</h3>
      <button onClick={counter.increment}>Add</button>
      <button onClick={counter.decrement}>Reduce</button>
      <button onClick={counter.reset}>Reset</button>
    </div>
  );
});
```

MobX also has store, not store, but storeS

```js
import { Provider } from "mobx-react";
import { ItemStore } from "./stores/ItemStore";
import { UserStore } from "./stores/UserStore";

const itemStore = new ItemStore();
const userStore = new UserStore();

ReactDOM.render(
  <Provider itemStore={itemStore} userStore={userStore}>
    <App />
  </Provider>,
  document.querySelector("#root")
);
```

When we need to pull out some properties, we can do this

```js
// for class component
@inject("itemStore")
class NewItem extends Component {
  state = {...}
  handleChange = e => {...}
  render() {...}
}

// for function component
const UnpackedItems = inject("itemStore")(observer(({ itemStore }) => {
  return (
    <Items
      title="Unpacked Items"
      items={itemStore.filteredUnpackedItems}
      total={itemStore.unpackedItemsLength}
    >
      <Filter
        value={itemStore.unpackedItemsFilter}
        onChange={itemStore.updateUnpackedFilter}
      />
    </Items>
  )
}))
```

and all the data inside the `itemStore` will be available from `this.props.itemStore`.
