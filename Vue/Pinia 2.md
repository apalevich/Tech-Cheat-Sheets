Стейт менеджер, который:
- Официальный для Vue 3
- Реактивен по умолчанию
- Модульный — можно использовать несколько сторов вместе или по отдельности
- Разработан автором Vue Router и членом Vue.js Core Team Эдуардо Сан Мартин Мароте

# Основные механики

## Инициация стора

```js
import { defineStore } from 'pinia';

export const useTodoStore = defineStore('TodoStore', { //use... is a conventional prefix
	state: () => ({
		todos: []
	}),
	actions: addTodo(todo) {
		this.todos.push(todo)
	},
	getters: {
		doneTodos: (state) => state.todos.filter(todo => todo.done)
	}
})
```

Структура стора подобна структуре компонентов Vue:
- **state подобен data**, который хранит данные
- **actions подобны methods**, которые изменяют данные
- **getters подобны computed**, и возвращают обработанные свойства без изменения оригинальных данных

## Получение состояния

```html
<script setup>
import { useTodoStore } from '@/stores/TodoListStore';
import { storeToRefs } from 'pinia'; // to make markup reactive

const store = useTodoStore();
const { todoList } = storeToRefs(store);
</script>

<template>
	<ul>
		<li v-for="todo in todoList">{{ todo }}<li>
	</ul>
</template>
```

> [!WARNING] Деструктурированные из стора свойства не будут реактивными, если не применить к ним `storeToRef`

## Изменение состояния

```html
<script setup>
import { ref } from 'vue';
import { useTodoStore } from '@/stores/todoListStore';

const newTodo = ref('');
const store = useTodoListStore();

const checkAndAddTodo = (item) => {
	if (!item) {
		return
	}
	store.AddTodo(item);
	newTodo.value = '';
}
</script>

<template>
	<form @submit.prevent="checkAndAddTodo(newTodo)">
		<input v-model="newTodo" type="text">
		<button>Add</button>
	<form>
<template>
```

# Миграция c Vuex

Pinia по сравнению с Vuex:
- Позволяет использовать actions напрямую, без мутаций (проще API)
- Лучшая поддержка DevTools
- Поддержка TypeScript

## Процесс

Предположим, у нас в проекте уже есть index.js с подключенным `vuex` и инициализированным стором:

```js
import { createService } from 'vuex';

export default createStote({
	state: () => ({
		user: 'John Doe',
		events: []
	}),
	mutations: {
		ADD_EVENT(state, event) {
			state.events.push(event);
		}
	},
	actions: {
		createEvent({ commit }, event) {
			commit('ADD_EVENT', event)
		}
	}
})
```

Сначала нам следует выполнить `npm install pinia` чтобы установить пакет. После этого нам надо использовать его в main.js:
```js
import { createApp } from 'vue';
import { createPinia } from 'pinia';

import App from '@/App.vue';
import store from '@/stores'

createApp(App)
	.use(store)
	.use(createPinia())
	.mount('#app')
```

Далее, мы создаём нужные нам сторы для Pinia:
1. Для каждого стора делаем файл типа`somethingStore.js` в папке `/store`
2. В каждом файле импортируем `defineStore` из `'pinia'`
3. Экспортируем константу с именем `useSomethingStore`, которой присваиваем `defineStore("SomethingStore", {объект_стора})`

> [!WARNING] Не забудь, что свойство store — это функция, возвращающая объект. То есть как data в компонентах, а не как во Vuex

Осталось получить данные в компоненте:

```html
<script>
import { useUserStore } from '@/stores/UserStore';
export default {
	setup() {
		const userStore = useUserStore();
		return {userStore}
	}
}
</script>

<template>
	<p>Logged in as {{ userStore.user }}</p>
</template>
```

## Геттеры
Геттеры во Vuex всегда получают state в качестве аргумента. В Pinia есть два варианта:

1. Обычная функция — имеет state в качестве this:
```js
getters: {
	firstName() {
		return this.user.split(' ')[0]
	}
}
```
2. Стрелочная функция — имеет state в качестве аргумента:
```js
getters: {
	numberOfNames: state => state.user.split(' ').length
}
```

В обоих случаях геттер используется в компоненте так же, как обычное свойство из состояния

## Actions

Изменение состояния не требует коммитов, поэтому можно скопировать `actions` из стора `vuex` и в каждом изменить одну строчку:

Вместо
```js 
actions: {
	setEvents({ commit }) {
		someAsyncCall
		.then(r => commit('ADD_EVENT', r.events))
	}
}
```

Используем
```js
actions: {
	setEvents(events) {
		
		someAsyncCall
		.then(r => this.events = r.events)
	}
}
```

А в компоненте импортируем и создаём нужный стор, после чего просто заменяем `$store.dispatch(name_of_action)` на прямое использование свойства:

```js
import { useEventStore } from '@/stores/EventStore'
export default {
	setup() {
		const eventStore = useEventStore()
		return { eventStore }
	},
	created() {
		this.eventStore
		.setEvents()
	}	
}
```

> [!WARNING] Не забудь заменить $store.dispatch(action) в вёрстке и других местах

## Практическая работа

Выполните миграцию с Vuex на Pinia в ветке https://github.com/Code-Pop/from-vuex-to-pinia/tree/07-Challenge

# Best Practices / Proven Patterns

Встроенная во Vue система реактивности позволяет писать простые js-файлы с объектом состояния, которые прекрасно работают. Зачем же использовать Pinia?

- Для масштабирования и работы в команде
- Чтобы изменять состояние более упорядоченно через actions
- Более безопасно для SSR, так как последние инициализируют состояние на сервере
- Расширенные инструменты DevTools (например, вкладка Timeline)

а также поддержка TS, фокус на опыте разработчиков и т.д. 

## Инициализация через Options и Setup

Примеры, указанные выше, описаны как Options сторы, где в `defineStore` вторым аргументом передаётся объект со state, actions и т.д.

В Setup стор вторым аргументом передаётся функция. Такой способ больше напоминает Composition API во Vue.

Setup имеет расширенную фунциональность, позволяя использовать:

- [Composables](https://vuejs.org/guide/reusability/composables.html#what-is-a-composable) (VueUse) — функции, позволяющие управлять изменяющимся состоянием (например, позицией мыши)
- Вотчеры

Пример с ними — компонент, определяющий локацию пользователя:

```js
import { watch } from 'vue'
import { defineStore } from 'pinia'
import { useGeoLocation } from '@vueuse/core'

export const useGeoLocationStore = defineStore('geolocation', () => {
	const { coords } = useGeoLocation()

	// следим за координатами, чтобы обновлять географическое название 
	watch(
	() => coords.value,
	(newValue) => {
		if (newValue) getLocation(coords.value.latitude, coords.value.longitude)
	})

	async function getLocation(latitude, longitude) {
		// запрос к API, определяющему 
	}
	
	return {
		getLocation,
		coords
	}
})

```

> [!INFO] Ключевые моменты:
> 1. Options-сторы используют this для доступа к свойствам (или аргумент state в геттерах)
> 2. Setup-сторы могут обращаться к свойствам напрямую


## Модульность

> [!INFO] Вот ключевые принципы, по которым стоит выделять отдельные модули:
> 1. Данные в одном сторе связаны логически
> 2. Также они могут быть связаны по фичам приложения
> 3. Несколько сторов могут использовать один и тот же API/библиотеку. Это неважно
> 4. Если нужно заимствовать данные из одного стора в другом, используй nested stores

Если функциональность одного стора полагается на данные из другого стора, их можно использовать друг в друге. Эта техника называет Nested Stores.

Например, стор списка ресторанов полагается на стор геолокации (через Setup фунцию):
```js
import { useGeoLocationStore } from './geolocation'

export const useRestaurantsStore = defineStore('restaurants', () => {
	const locationStore = useGeoLocationStore()

	async function getRestaurants() {
		const lat = locationStore.latitude
		const long = locationStore.longitude
		// Запрос
	}
})
```

# Встроенные методы

## $patch

Альтернативный способ обновить данные в сторе. Удобен когда обновить нужно одновременно — это создаст одну запись в DevTools.

`$patch` принимает объект с новыми свойствами, либо функцию которая принимает state и ничего не возвращает:

```js
import { useRestaurantStore } from './stores/restaurants'
const restaurantsStore = useRestaurantStore()


watch(city, (newVal) => {
	if (newVal) {
		// Через объект
		restaurantsStore.$patch({
			restaurantDetails: [],
			searchChoice: ''
		})

		// Через функцию
		restaurantsStore.$patch((state) => {
			state.restaurantDetails = []
			state.searchChoice = ""
		
		})
	}
})



```

$patch может так

## $reset
Сбрасывает значения в текущем сторе в дефолтным.

```html
<button @click="authStore.$reset()">
	Log Out
</button>
```

> [!INFO] $reset работает только в Options-сторах, так как под капотом использует state()

## $onAction

Способ подписаться на экшны. Принимает функцию, которая выполняется перед экшном. При этом аргументы функции способны значительно расширить её функциональность:

```js
someStore.$onAction({(
	name, // имя текущего экшна
	store, // текущий стор
	args, // параметры, переданные в экшн
	after, // колбек, выполняющийся после экшна, принимает результат
	onError, // колбек, выполняющийся в случае ошибки экшна, принимает ошибку
}, state) => {
	const startTime = Date.now()
	
	after((result) => {
		console.log(`Finished ${name} after ${Date.now() - startTime}ms. Result: ${result}`)
	})

	onError((error) => {
		console.warn(`Failed ${name} after ${Date.now() - startTime}ms. Error: ${error}`)
	})
})
```

# Плагины

Плагины — это функции, котрые дополняют возможности Pinia.

Плагины имеют доступ, который называет `context`. Он содержит:
1. Само приложение
2. Текущий экземпляр pinia
3. Стор, расширяется плагином
4. Объект, переданный в `defineStore`

Что позволяют делать плагины:
1. Добавлять новые свойства в сторы
2. Добавлять новые методы в сторы (или оборачивать существующие)
3. Изменять и отменять экшны
4. Создавать сайд-эффекты

## Пример использования плагина:

1. Пишем плагин—функцию, принимающую в качестве аргумента `context`:
```js
export function myPlugin(context) {
	return {
		logContext: function() {
			console.log(context)
		}
	}
}
```
2. Импортируем функцию из папки плагинов в `main.js`:
```js
import { createApp } from 'vue'
import { createPinia } from 'pinia'
import App from './App.vue'
import { myPlugin } from './stores/plugins'

const app = createApp(App)
const pinia = createPinia()
pinia.use(myPlugin)

app.use(pinia)
```
3. Теперь мы можем использовать свойство плагина внутри стора:
```js
async login (user, pass) {
	this.logContext()
	// some code
}
```

### Более сложный пример плагина

Мы можем использовать внутри стора флаг для включения плагина:

```js
// OPTIONS STORE
export const useAuthStore = defineStore('auth', {
	state: () => ({...}),
	actions: {...},
	getters: {...},
	greeting: {
		enabled: true
	}
})
/// SETUP STORE
export const useAuthStore = defineStore('auth', {...}, { // третий аргумент
	greeting: {
		enable: true
	}
})
```

После этого добавляем проверку в плагин:

```js
export function greetOwnerPlugin({store}) {
	if (options.greeting && options.greeting.enabled) { // проверка
		store.$onAction((action) => {
				if (store.$id === 'auth') { // Identify which store
					switch (action.name) {
						case "login":
							alert('Welcome back!')
							break;
						case "logout":
							alert('Hope you enjoyed your session!')
							break;
						case "register":
							alert('Welcome!')
							break;
					}
				}
			}
		})
	}
}
```

Использование `$onAction` внутри плагина позволяет реагировать на ошибки, например, посылать их во внешнюю систему типа Sentry.

