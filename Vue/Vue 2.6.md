---
tags:
  - JavaScript
  - Framework
---


> - Incrementally adoptable
> - Focused on view layer only
> - Easy to integrate with libraries
> - perfectly capable of poweringsophisticated SPA
> 
> *— from Vue's documentation*

# Инсталляция

Есть различные сборки Vue:

- Compiler — транслирует шаблоны (template strings) и в рендер-функции
- Runtime — отвечает за создание экземпляров, рендеринг, взаимодействие с виртуальным DOM, буквально **всё минуc компилятор**
```javascript
// this requires the compiler  
new Vue({  
template: '<div>{{ hi }}</div>'  
})  
  
// this does not  
new Vue({  
render (h) {  
return h('div', this.hi)  
}  
})
```

- Full — включает компилятор и рантайм
- UMD — для использования в браузере внутри тега `<script>`

> [!INFO] UMD (Universal Module Definition) — modules which are capable of working everywhere, be it in the client, on the server or elsewhere. [source](https://github.com/umdjs/umd)

- CommonJS — для использования со старыми сборщиками (browserfy, webpack1, etc.)
- ES Module — начиная с версии Vue 2.6 есть две сборки
	- ESM для современных сборщиков (webpack2, Rollup)
	- ESM для браузеров через `<script type="module">`

## Установка через webpack

Решения вроде `vue-loader` или `vueify` предварительно компилируют шаблоны из файлов `*.vue`. Такой способ предпочтительнее, так как сборка будет легче на ~30%

Если используется webpack, то нужно установить `vue-loader` из npm и указать Vue в качестве модуля лоадера в конфигурации:

```js
module: {
    loaders: [
      {
        test: /\.vue$/,
        loader: 'vue',
      },
      // ... your other loaders ...
    ]
  }
```

Если webpack не используется, то можно использовать `vue-cli`, который создаст его с нужными настройками:

```
npm i --global vue-cli
vue init webpack
```

# Пример использования

Приложение состоит из корневого экземпляра Vue:

```html
<div
	id="app"                  // Entry point
	v-bind:title="message"    // Attribute binding (reactive)
	some-attr=42 >
	{{ message }}             // Text interpolation
</div>

<script>
new Vue({                     // Vue instance
	el: '#app',
	data: {
		message: "Hello world!"
	},
});
```

> Подход, когда бизнес-логика описана в JavaScript и логика отображения в шаблонах, позволяет с первого взгляда понять, что будет отображено на странице. Это сравнительно лучше, чем описывать логику отображения элемента отдельно от него самого.
> 
> *Callum Macrae "Vue.js" (O'Reilly)* 

В данном примере app доступен как глобальная переменная. Содержимое data, computed и methods доступно напрямую как ключи.
```bash
> app.message
< "Hello world!"
```

При этом данные реактивны — при их изменении в JavaScript, они меняются в DOM.

Аттрибуты, DOM-элемент и другое доступно в общей области видимости через специальные свойства:
```js
console.log(app.#el)
// HTMLElement div.#app
console.log(app.#attrs)
// { "some-attr": 42 }
```

> [!WARNING] Указанный метод называется DOM-template и он имеет некоторые ограничения. Например, внутри тегов `<ul>`, `<ol>`, `<table>` и `<select>` вместо `<component>` нужно использовать `<li>`, `<tr>` и `<option>` с аттрибутом  `:is="component"` соответственно
> 

Другие сточники компонентов не имеют таких ограничений:
- Свойство template (`template: '...'`)
- Файлы компонентов  с расширением `.vue`
- Использование тега `<script type="text/x-template">`

# Экземпляр Vue

> Каждый компонент — это экземпляр Vue
 *-документация*

Содержимое data добавляется в **Reactivity System** Vue, кроме случаев:
- Свойство добавлено после создания экземпляра
- На свойстве (data?) использован `Object.freeze()`

## Хуки жизненного цикла

У каждого этапа жизненного цикла есть свой метод (хук) в экземпляре:
- created () {...}
- mounted () {...}
- updated () {...}
- destroyed () {...}

>[!INFO] Не используй стрелочные функции на хуках, так как они используют Window в качестве this

# Template syntax

Vue использует синтаксис HTML для создания темплейтов. Благодаря этому Vue-темплейты могут быть понятны почти любым браузерам и парсерам.

Под капотом Vue компилирует темплейты в рендер-функцию виртуального DOM. И когда состояние меняется, Vue применяет систему реактивности и таким образом определяет минимальный список необходимый изменений DOM.

## Контент
```html
Реактивное значение в темплейте:
<span>{{ message }}</span>

Нереактивное значение:
<span v-once>{{ message }}</span> 

Можно использовать выражения:
<span>{{ 'Score: ' + message.toUpperCase() }}
```

## Аттрибуты
```html
Инъекция html-кода из переменной:
<span v-html="rawHtml">

Назначение аттрибута из переменной:
<img :id="componentId">

Назначение булевых аттрибутов:
<buttod :disabled="buttonDisability">
```

Аттрибут и директива будут включены при рендеринге только если его значение приводится логически к `true`

# Computed, Metrods и Watchers

Главное отличие Computed от Methods в том, что первый — кэшируется.

В Computed можно назначать отдельно сеттеры и геттеры:
```js
computed: {
	// short way:
	fullName() {
		return `${this.firstName} ${this.lastName}`
	}
	// explicit way:
	fullName: {
		get() {
			return `${this.firstName} ${this.lastName}`
		},
		set(newValue) {
			var names = newValue.split(' ')
			this.firstName = names[0]
			this.lastName = names[names.length - 1]
		}
	}
}
```

Watch даёт дополнительные возможности, например:
- Выполнять асинхронные операции
- Ограничивать частоту операций

# Классы и стили

## Классы
могут передаваться в `v-bind:class` разными способами:
```html
Нативно и в виде строки:
<img class="paddings-12 align-center">
В виде ключей объекта, включающимися по условию:
<img :class="{'align-center': isCentered}">
В виде массива:
<img :class="[centeredPos, paddingClass]">
Массив поддерживает тернарный оператор:
<img :class="[isCentered ? centeredPos : leftPos]">
Массивы могут содержать другие формы:
<img :class="[{'align-center': isCentered}, paddingClass, 'red-background']">
```

Если при использовании компонента указать ему классы, они объединятся с классами тега верхнего уровня в компоненте

## Стили

Стили имеют отличия:
```html
Нативно и в виде строки
<img style="padding: 0 12px; border: 1px solid black">
В форме объекта:
<div :style="{color: activeColor, fontSize: fontSize + 'px'}">
В форме массива с объектами:
<div :style="[styleObject, errorStyles]">
```

В объекте ключи должны писаться в кавычках, либо camelCase:
```js
data: {
	styleObject: {
		fontSize: '16px',
		color: 'red',
		'background-color': aquamarine,
	},
}
```

В стандартных сборках Vue можно не писать префиксеры, так как PostCSS с плагином autoprefixer включены по умолчанию

Тем не менее, префиксы можно указать в значениях. Использоваться будет только подходящий браузеру префикс с фолбэком в последнее значение ([?](https://v2.vuejs.org/v2/guide/class-and-style.html#Multiple-Values))
```html
<div v-bind:style="{ display: ['-webkit-box', '-ms-flexbox', 'flex'] }"></div>
```

# Директивы

```html
Обычная директива:
<img v-if="check">

Директива с аргументом:
<a v-bind:href="url">Link</a>

Динамический аргумент:
<a v-bind:[attr]="url">Link</a>
```

В переменной динамического аргумента должна быть либо строка, либо `null` — тогда аттрибут удаляется.

> [!INFO] Лучше избегать заглавных букв в названиях динамических аргументов, так как в темплейтах внутри HTML-файлов (in-DOM templates) они будут автоматически преобразованы в строчные


## Условия

`v-if` — это директива, которая при ложном значении уничтожает листенеры и вложенные компоненты/элементы. Её можно назначать элементу `<template>` и завернуть в него несколько элементов.

Если при использовании `v-if` состояние сохраняется, то можно добавить аттрибут `key`, чтобы указать Vue, что элементы нужно заменить полностью.

`v-show` стоит использовать когда нужно, чтобы элемент остался в DOM-дереве, но был скрыт через `display: none;`

> [!INFO] Основное отличие: `v-if` требует больше ресурсов для включения/выключения, а `v-show` — для инициализации

Не рекомендуется использовать в одном элементе `v-if` и `v-for`. Если это происходит, последний имеет больший приоритет, поэтому лучше использовать `v-for` на родителе. 
Согласно [Style Guide](https://v2.vuejs.org/v2/style-guide/#Avoid-v-if-with-v-for-essential), есть только один случай, где это допустимо — для фильтрации значений:

```html
<li v-for="todo in todos" v-if="!todo.isDone">{{ todo }}</li>
```

Элемент с `v-else` должен идти сразу за элементом с `v-if` или `v-show`

## Циклы

### Для последовательности чисел:
```html
Генерация последовательности чисел (c 1!):
<span v-for="n in 10">
```

### Для массива:
```html
Через in:
<li v-for="(item, index) in items">
Так же работает через of:
<li v-for="(item, index) of items">
```

### Для объекта:
```html
Обход объекта (под капотом Object.keys):
<li v-for="(value, key, index) in items">
```

### Производительность

Можно повысить производительно, назначив атрибут `key` — тогда при изменении данных Vue определит, где какой элемент и пропатчит только нужные.

> [!INFO] Изменяющие(`push`, `pop`, ...) и неизменяющие (`map`, `filter`, ...) методы имеют примерно одинаковую производительность, так как Vue стремится максимально переиспользовать DOM-элементы, если данные в новом массиве совпадают с данными из старого.

### Фильтрация элементов

Фильтры указываются в директиве `v-for` через вертикальную черту с командой `filterBy`:

```html
<li v-for="item in items | filterBy 'true' in 'done'"
```

Можно сделать поиск, используя указав переменную, заданную через модель инпуту:

```html
<input v-model="query">
<h3>Search results<h3>
<ul>
	<li v-for="item in items | filterBy query in 'name'">{{item.name}}</li>
</ul>
```

### Ранжирование элементов

Ранжирование указывается через встроенный фильтр `orderBy`:

```html
<li v-for="item in items | orderBy 'id'"
```

# Реактивность

Реактивность — самая мощная часть Vue, которая встречается повсюду. Как это работает?

Она работает через изменение объектов в data. Каждое свойство каждого объекта Vue заменяет на геттеры и сеттеры.

>[!INFO] Dirty checking — это способ поиска изменений, при котором сохранённые копии объекта сравниваются с текущими. Использовался в Angular 1

Реактивность во Vue больше похожа на [Object.defineProperty](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty) bo:
```js
Object.defineProperty(data, 'userId', {
	get() {
		return 
	}
})
```


> [!WARNING] Следующая информация рекомендуется для добавления:
> - Валидация пропсов https://medium.com/js-dojo/validating-your-vue-props-like-a-pro-5a2d0ed2b2d6
> - Реактивность во Vue https://habr.com/en/companies/nordclan/articles/706536/
# Другие статьи по теме
# Внешние ссылки

