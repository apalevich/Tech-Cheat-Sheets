Популярная JavaScript-библиотека, позволяющая выполнять [HTTP](HTTP.md)-запросы в браузере или [Node.js](Node.js.md)
# Установка

Axios устанавливается и испортируется так же, как любой другой пакет через `npm install axios` или `yarn add axios`.

Для использования в браузере можно загрузить через тег `<script>`:
```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

# Настройка

При необходимости можно определить глобальные настройки, используемые в каждом вызове:

```js
axios.defaults.baseURL = 'https://api.example.com'; axios.defaults.headers.common['Authorization'] = AUTH_TOKEN; axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
```

Можно также создать несколько экземпляров axios и каждому передать его дефолтные настройки:

```js
const instance = axios.create({ baseURL: 'https://api.example.com' });
```
# Использование API через методы

Параметры запроса могут быть переданы объектом в качестве аргумента функции `axios()`, либо можно использовать методы, указывающие метод запроса:
- `axios.get()`
- `axios.post()`
- `axios.delete()`
- `axios.put()`
- `axios.patch()`
- `axios.options()`

https://axios-http.com/docs/api_intro
# Запросы

Запросы axios являются асинхронными, поэтому есть несколько способов c ними работать:

1.  На промисах с колбэками:
```js
const axios = require('axios')

const getItems = () => {
	try {
		return axios.get(url)
	} catch (er) {
		console.error()
	}
}

getItems()
	.then(function (response) {
		console.log(response);
	})
	.catch(function (error) {
		console.log(error);
	});
```

2. На await/async:
```js
const axios = require('axios')

const getItems = async () => {
	try {
		return await axios.get(url)
	} catch (er) {
		console.error()
	}
}

const countItems = async () {
	const items = await getItems()

	if (items.data) {
		console.log(`Got ${items.data.length} items`)
	}
}
```

# Внешние источники
Документация — https://axios-http.com/docs/intro
Книга Flavio Copes "Node.js" — https://flaviocopes.com/access/