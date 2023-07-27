- State manager pattern & library
- Similar to Redux, Flux, etc. but easier since it's mantained speccialy for Vue
- Centralised store for all components

# The Pattern

We are going to solve these problems:

1. Prop drills
2. When we have multiple components that share a common state, so actions from different views may need to mutate the same piece of state.

State manager patter is when component tree becomes a big "view", and any component can access the state or trigger actions, no matter where they are in the tree. So that **we extract the shared state out of the components, and manage it in a global singleton**

![[vuex.png]]

Store — container of the application state. It's **reactive** and changes **only by commiting mutations**.

## Example of code

```javascript
import { createStore } from 'vuex';

const store = createStore({ 
	// REACTIVE DATA
	state: {
		loading: true,
		todos: [],
	},
	// TO TRACK CHANGES
	mutations: {
		SET_LOADING (state, status) {
			state.loading = status
		},
		SET_TODOS (state, todos) {
			state.todos = todos
		},
	},
	// TO UPDATE DATA IN STATE
	actions: {
		fetchTodos(context) {  // — contains store object
			context.commit('SET_LOADING', true);
			axios.get('/someapi/todos')
				.then(r => {
					context.commit('SET_LOADING', false);
					context.commit('SET_TODOS', r.data.todos);
				})
		}
	},
	// TO GET PROCESSED DATA FROM STATE
	getters: {
		doneTodos (state) => {
			return state.todos.filter(todo => todo.done)
		}
	}
})
```

>[!INFO] In a Vue component, you can access the store as `this.$store`


# State

*Single state tree* - when a single object contains all the application level state and serves as the "single source of truth."

> [!INFO] The data you store in Vuex follows the same rules as the `data` in a Vue instance, ie the state object must be plain. **See also:** [Vue#data](https://v3.vuejs.org/api/options-data.html#data-2).

We can access the store from components using `this.$store` inside the `computed` property. Anyway, it's conventionally a bad pattern. Import mapState from vuex and use it:

1. Arrow functions with state as argument
``` javascript
computed: mapState({
    todos: state => state.todos,
}),
```

2. Name of the prop as a string ('x' is same as `state => state.x`)
```javascript
computed: mapState({
	items: 'todos'
}),
```

3. Array of props name as string — every x will became `this.x`
```javascript
computed: mapState([
	'todos', 'loading',
]),
```

4. Processing with another property from `this`: 
```javascript
data: {
	currentItem: 'foo',
},
computed: mapState({
	itemsPlusCurrent (state) { // better to use getter with argument?
		return state.items + this.currentItem
	}
})
```

> *If a piece of state strictly belongs to a single component, it could be just fine leaving it as local state. You should weigh the trade-offs and make decisions that fit the development needs of your app.*

# Getters

Getters are used when we have to process values from the store state. We may think of getters as computed properties of the store itself.

```js
const store = createStore({
	...
	getters: {
		doneTodos (state) {
			return state.todos.filter(todo => todo.done)
		},
		countDoneTodos (state, getters) {
			return getters.doneTodos.length
		}
	}
})
```

Mind the second argument — it's getters itself!

Computed values accesible from components as `this.$store.getters.doneTodos`

## Getter as a method

Make a getter to return a function, that you could pass an argument there:

```js
getters: {
	getTodoData: (state) => (id) => {
		return state.todos.find(todo => todo.id === id)
	},
},
```

Use `this.$store.getters.getTodoData(2)` inside components.

## Map Getters

`MapGetters()` is available to get multiple store getters:

```js
computed: {
	someLocalComputedValues,
	...mapGetters([
		'doneTodos',
		'countDoneTodos',
		'getTodoData',
	]),
},
```

It's possible to rename getter while mapping:

```js
computed: {
	...mapGetters({
		countDone: 'countDoneTodos',
	}),
},
```


# Mutations

Mutations — is the only way to change the state. A process of applying mutations called *commiting*.

Mutations are like event: every mutation has a **handler** and a string **type**.

A handler function is where we perform a state change:
```js
mutations: {
	eraseList (state) {
		state.todos = [];
	},
},
```

> [!warning] Mutation hanlrers always should be synchronous. To handle async operations an one shall use [[#Actions]]

Type is handler name as a string used to commit a mutation: `store.commit('eraseList')`.

## Payload

Mutation handlers have a second argument as a payload:
```js
mutations: {
	addTodo (state, newTodo) {
		state.todos.push(newTodo)
	},
},
```

Now we can commit a mutation in a short form:
```js
store.mutations.commit('addTodo', {value: 'foo', id: 6});
```

Or exposing a handler type:
```js
store.mutations.commit({
	type: 'addTodo',
	value: 'foo',
	id: 6,
});
```

For mapping in components there is `mapMutations` available.

# Actions

Differencies between Actions and Mutations:

1.  Instead of mutating the state, Actions commit Mutations
2. Actions can contain arbitrary asynchronous operations
3. Actions always returns promise [source](https://github.com/vuejs/vuex/blob/3.x/src/store.js#L152)

In a Store, Actions get the context as an argument, that we can acess `context.state`, `context.gettes`, `context.commit` and other actions via `context.dispatch`

So basically actions are async mutations-like wrappers for standart mutations!

```js
actions: {
	incrementAsync (context, payload) {
		setTimeout(() => {
			context.commit('increment', payload.amount)
		}, 1000)
	},
},
```

Then we access that action via `store.dispatch`:
```js
store.dispath('incrementAsync', {amount: 10})
// OR
store.dispacth({
	type: 'incrementAsync',
	amount: 10
})
```

For mapping in components there is `mapActions` available.

## Async tricks

We can control the flow of commits within async actions using Promises:
```js
actions: {
	actionA (context) {
		return new Promise((resolve, reject) => {
			setTimeout(() => {
				context.commit('someMutation')
			}, 1000);
			resolve();
		})
	},
	actionB (context) {
		return context.dispatch('actionA')
		.then(() => {
			context.commit('otherMutation')
		})
	},
}
```

Another example using async/await approach:
```js
actions: {
	async actionA (context) {
		context.commit('gotData', await getData()) // getData return a Promise
	},
	async actionB (context) {
		await context.dispatch('actionA')
		comtext.commit('gotDataOther', await get)
	}
}
```

# Modules
A store could have modules — each of them may have it's own state, actions, getters and even nested modules
```js
import { moduleA } from './moduleA';

const moduleB = {
	state: {...},
	actions: {...},
	mutations: {...},
};

const store = createStore({
	modules: {
		a: moduleA,
		b: moduleB,
	}
});

console.log(store.state.a, store.state.b)

```

The `context` argument inside module Actions will be enhanced with `rootState` property, and module Getters will have third argument containing rootState as well.



