Todo App from the Redux documentation [[1]](https://redux.js.org/basics/exampletodolist).

## The Todo App Features

- Create a new todo.
- Display a list of todos, can click on each todo to toggle its completed status.
- Filter the list to display all/completed/incompleted todos.

## React

We define some components for the app:
- `AddTodo`: includes **input** field for entering the todo name and a **button** to add it into the list.
- `Todo`: displays a todo, has its **onClick** callback to self-toggle.
- `TodoList`: displays a todo list by iterating through all todos.
- `Link`: displays filter options, has its **onClick** callback to perform the filtering.

Normally, each aforementioned component is responsible for both manipulating and displaying the data. For example, the TodoList component may have to both filter and list of todos and display them, probably something like this:

```javascript
class TodoList extends React.Component {
  render() {
    /* Business logic: Filtering the list */
    const todos = []
    this.props.todos.forEach(todo => {
      if (this.props.filter === todo.status)
        todos.push(<Todo />)
      // and so on...
    })
    
    /* Display logic: Display the list */
    return ({todos})
  }
}

/* Usage */
<TodoList todos={todos} filter={filter} />
```

However this causes the tight coupling between the "display logic" and "business" logic, it's better if we can separate them and divide React components into: **Presentational and Container Components** [[2]](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0). Redux documentation writes a nice summary about them [[3]](https://redux.js.org/basics/usagewithreact#presentational-and-container-components). We rewrite the above code to something like this:

```javascript
/* Presentational Component */
class TodoList extends React.Component {
  render() {
    return ({this.props.todos})
  }
}

/* Container Component */
class VisibleTodoList extends React.Component {
   // `todos` and `filter` can be props, state,
   // fetched from API or gotten from Redux state
   // as we will see shortly.
  
  render() {
    return <TodoList todos={todos} filter={filter} />
  }
}

/* Usage */
<VisibleTodoList />
```

By doing this, we can utilize Presentation Component easier, keep the codebase manageable and flexible.

Naming:
- Presentational Components are responsible for displaying, thus they should be named as **Noun** (we display "something").
- Container Components handle the logic, meaning that they "apply" something onto Presentational Components, we should name them as **Adj/Verb + Noun**.

Accordingly, our Container Components are:
- `VisibleTodoList`: responsible for filtering the todo list before displaying.
- `FilterLink`: responsible for specifying the state of the link before displaying (which filter option is being selected).
- `AddTodo`: [read more](https://redux.js.org/basics/usagewithreact#designing-other-components).

## Redux and React-Redux

Using React, our App needs to be aware of all the state such as the todo list and the filter status, we could have all them in the App component and pass as props for its children like this:

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      'visibilityFilter': SHOW_ALL  // Initial value
    }
  }
  
  render() {
    return (
      <div>
         <AddTodo />
         <VisibleTodoList todos={this.props.todos} filter={this.state.visiblityFilter} />
         <Footer />  // Contains 3 FilterLink (all/completed/incompleted)
      </div>
    )
  }
}

// Usage
const todos;  // Fetch from API
<App todos={todos} />
```

When the app is large, the whole App's state can be complex and complicated to manage. This is where Redux comes to play: (container) components doesn't own the status anylonger, all state will be kept in a single **store** of the project.

_Insert Redux docs here_ :wink:

First we write Action Creators in `actions/index.js`. You can read about why we use them in [4].

And the reducers too in `reducers/index.js`.

Now comes the interesting part: the data is now in the store, we need to make (container) components access them in order to manipulate and display them. There are 2 things we need to do:
- How to make the store _visible_ to components?
- How to make components read and write to the store?

**How to make the store _visible_ to components?**

We can pass the store as the property of the App, then the App continues passing down to its children. And then all components in the tree can read from the store using `store.getState()` and write to it using `store.dispatch(actionCreator)`:

```javascript
const store = createStore(rootReducer)

<App store={store} />
```

However this is inconvenient since we need to explictly pass the store down to every component.

This is where **React-Redux** comes into play, it uses a special component called `<Provider>` to make the store available to all container components in the application without passing it explicitly:

```javascript
import { Provider } from 'react-redux'

// You only need to use it once when renderring the root
<Provider store={store}>
  <App />  // All components in the tree can now "see" the store
</Provider>
```

**How to make components read and write to the store?**

So now we need a way so that our container components can interract with the store. As we saw earlier, container components are mainly responsible for business logic, we can write them by hand using `store.subscribe()` to know when the store change so that they can tell their corresponding presentational components to display accordingly.

However, we should generate our container components with the React Redux library's `connect()` function, which provides many useful optimizations to prevent unnecessary re-renders [5].

The `connect()` methods involves two means to interract with the store:
- `mapStateToProps`: this function is subscribed to the store, meaning that it will be called when the state changes. The container component can READ via this.
- `mapDispatchToProps`: this function contains the `dispatch` method of the store, allow us to define some callback functions that dispatch actions to the store. Thus, the container component can WRITE via this. Notice that if this function is not passed to `connect()`, the container component's props will by default receive the `dispatch` function (used to dispatch actions to the store).

Hopefully everything now makes sense when you read the source of `containers/`.

## References

[[1] Example: Todo List](https://redux.js.org/basics/exampletodolist)

[[2] Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

[[3] Redux docs - Presentational and Container Components](https://redux.js.org/basics/usagewithreact#presentational-and-container-components)

[[4] Idiomatic Redux: Why use action creators?](https://blog.isquaredsoftware.com/2016/10/idiomatic-redux-why-use-action-creators/)

[[5] Implementing Container Components](https://redux.js.org/basics/usagewithreact#implementing-container-components)

