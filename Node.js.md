Node.js — это среда, которая позволяет запускать код на языке JavaScript на сервере или компьютере.

Преимущества:
- Скорость — до 2 раз быстрее C или Java
- JavaScript — позволяет писать бэкенд и серверные приложения на знакомом языке
- Асинхронность — достигается аналогично браузеру, через колбэки и события
- Множество библиотек — официальный реестр NPM содержит более 500 тысяч пакетов + прочие источники

# Инструменты
Node — сама среда выполнения JavaScript-кода

V8 — движок обработки кода внутри Node. Аналогичный используется в Chrome и Electron (но не в других браузерах). Он написан на C++ и имеет открытый исходный код.

> [!INFO] Несмотря на то, что JS изначально был интерпретируемым языком, с 2009 года движки стали его компилировать для лучшей скорости выполнения. V8 использует механизм компиляции Just-In-Time (JIT)

ECMAScript (ES) — это стандарт языка JavaScript. Он имеет различные версии, в которых вводятся новый синтаксис и прочие инструменты

NPM — менеджер пакетов и библиотек для Node

NVM — менеджер версий, который позволяет запускать различные версии Node (полезно для тестирования)

# Отличия от JS в браузере

Главное отличие — в Node нет привычных нам браузерных API: DOM, window, document и прочих Web Platform API. Зато есть свои API, как из внешних модулей, так и из стандартной бибилиотеки (например, дающие доступ к файловой системе).

Другое отличие — контроль над тем, в какой среде будет запускаться ваш код, какие версии Node и библиотек использовать и т.п. Это означает, что мы можем писать на любой версии ES без использования инструментов типа Babel.

Наконец, Node использует систему модулей CommonJS, а браузеры — ES модули. Практически это означает, что в Node мы используем `require()` вместо `import`.

## Интерактивный режим

Интерактивный режим Node называется REPL. Он доступен по команде `node` в терминале. Имеет функцию автодополнения через tab

Глобальные переменные доступны в объекте `global`

Результат последней операции доступен в переменной `_`

Также есть специальные команды, которые начинаются с точки:
- `.help` — показывает справку
- `.editor` — позволяет использовать Enter для переноса строки, запуск по Ctrl-D
- `.break` — сброс многострочной команды
- `.clear` — удаляет контекст
- `.load` — загружает файл по пути, относительно текущей директории
- `.save` <имя> — сохраняет сессию в файл с именем
- `.exit` — выход из интерпретатора

# API и стандартная библиотека

## `process`
Это стандартный модуль, который не требует импортирования и позволяет выполнять основные операции с приложением:

```js
// Доступ к переменным окружения
process.env.NODE_ENV

// Доступ к аргументам вызова из командной строки
process.argv // приходят в виде массива со строками, что может требовать парсинга

// Завершение приложения
process.exit() // Экстренное завершение
process.exitCode = 1 // Установка статуса для завершения
process.on('SIGTERM', () => { ... }) // Корректное завершение
process.kill(process.pid, 'SIGTERM') // Корректное завершение другого приложения
```

> [!INFO] SIGRTERM и SIGKILL являются сигналами низкоуровневой системы POSIX

## `console`
Очень похож на `console` в браузере, но объекты выводятся как строки. В остальном отличий не замечено.

Логирование позволяет указывать тип переменной для интерпретации:
- `%s` как строку
- `%d` или `%i` как целое число
- `%f` как дробное число
- `%O` как объект

```js
// Логирование с интерпретацией

console.log('My %s has %d years', 'cat', 2)
// My cat has 2 years
console.log('My %d has %f years', 'cat', 2.5)
// My NaN has 2.5 years

// Подсчёт одинаковых элементов

['orange', 'orange', 99].forEach(console.count)
// orange: 1;
// orange: 2;
// 99: 1;
console.count('orange')
// orange: 3
})

// Замер времени

console.time('timerName')
doSomething()
console.timeEnd('timerName')
```

## `readline`
Стандартный модуль, который позволяет получать данные ввода из потока данных. Например, вот как он работает с потоком `process.stdin` чтобы получить пользовательский ввод с клавиатуры:

```js
const readline = require('readline')
.createInterface({
	input: process.stdin,
	output: process.stdout
})

readline.question('Who are you?', (answer) => {
	console.log(`Hi ${answer}!`)
	readline.close()
})
```

Существуют альтеративные пакеты для обработки ввода, которые позволяют скрывать пароль, делать чекбоксы и много другое. Например, `Inquirer.js`.

## `http`

Этот модуль позволяет принимать и обрабатывать веб-запросы и таким образом создавать HTTP-сервер.

```js
const http = require('http')

const port = 3000

const server = http.createServer()

server.on('request', requestHandler)
server.on('listening', listeningHandler)

server.listen(port)

function requestHandler(req, res) {
	res.statusCode = 200
	res.setHeader('Content-Type', 'application/json')
	res.end(JSON.stringify({data: 'Hello World!'}))
}

function listeningHandler() {
	console.log(`Server running at port ${port}/`)
}
```

`http.createServer()` возвращает объект класса `http.Server`
У `http.Server` есть события, к которым мы привязывает колбэки: `requestHandler` и `listeningHandler` к событиям 'request' и 'listening' соответственно.

`req` и `res` в колбэке — это два объекта, передаваемые в аргументы на каждый запрос: [http.IncomingMessage](https://nodejs.org/api/http.html#http_class_http_incomingmessage) и [http.ServerResponse](https://nodejs.org/api/http.html#http_class_http_serverresponse) . Мы используем их для обработки тела запроса и задания параметров ответу. Когда мы используем `res.end()`, его аргумент устанавливает тело ответа и ответ отправляется. Этот метод обязательно должен быть установлен для ответа.

Кстати, передать колбэки можно иначе: аргументами в `http.createServer()` и `server.listen()`:

```js
const server = http.createServer(generalHandler)

server.listen(port, listeningHandler)
```

Модуль `http` способен и сам выполнять запросы:

```js
const options = {
	method: 'GET',
	hostname: 'example.com',
	port: 443,
	path: '/about'
}

const req = http.request(options, (res) => {
	console.log(`statusCode: ${res.statusCode}`)

	res.on('data', (d) = > {process.stdout.write(d)})
})

req.on('error', console.error)

req.end()
```

POST-запрос выглядит чуть сложнее:

```js
const payload = JSON.stringify({todo: 'Buy milk'})

const options = {
	method: 'POST',
	hostname: 'example.com',
	port: 443,
	path: '/todos',
	headers: {
		'Content-Type': 'application/json',
		'Content-Length': data.length
	}
}
...
req.write(data)
req.end()
```

Методы PUT и DELETE выполняются аналогично POST, достаточно изменить метод в объекте `options`.

> [!WARNING] Выполнять запросы таким образом не очень удобно, поэтому обычно используют стороннюю библиотеку [Axios](Axios.md). Если всё же необходимо использовать стандартные модули, импортируй `https` вместо `http` для безопасности
# Импорты и экспорты

По умолчанию объекты и переменные приватны, но их можно сделать доступными для импорта в другие файлы двумя способами:

```js
const data = { meaning: 42 }

// 1 — для экспорта объекта
module.exports = data

const data = require('./data')

// 2 — для экспорта свойств
exports.data = data

const items = require('./items').data
```

# Команда NPX

`npx` — это сокращение от "npm exec", то есть выполнение кода через npm. Ей можно передать любые команды и скрипты, которые нужно выполнить — это позволяет найти ей множество применений.

Например, запуск CLI-утилит:
```shell
> npx vue create my-vue-app
> npx create-react-app my-react-app
```

>[!QUESTION] Flavio Copes пишет, что эта команда может выполнять команды пакетов, даже не устанавливая их. Однако, у меня не получилось этого добиться на node 16-18
 
Также можно использовать `npx` для переключения между версиями Node (вместо  `nvm`): 

```shell
> npx node@16
```

Другая особенность `npx` — она позволяет запускать код не только из пакетов, а даже из удалённых источников, например:

```shell
> npx https://gist.github.com/zkat/4bc19503fe9e9309e2bfaa2c58074d32
```

# Асинхронность

JavaScript по умолчанию синхронный и однопоточный. Что это значит?

Количество потоков условно означает количество задействованных процессоров. То есть устройства с одним процессором (ядром?) — однопоточные.

Асинхронность означает, что события могут происходить независимо друг от друга. 

В современных компьютерах у каждой программы есть свой независимый поток выполнения, а компьютер быстро переключается между ними. Таким образом, они однопоточные, но асинхронные.

Научившись управлять потоками выполнения в коде JavaScript, мы делаем его асинхронным.

## Event Loop

ЭЛ — это механизм работы выполнения команда в JS. Он происходит из однопоточности языка — все операции выполняются друг за другом.

Однако, мы можем управлять очередью задач, идущих на выполнение в ЭЛ. В этой части описано, как это сделать.

> [!IMPORTANT] В отличие от Node.js, браузеры обычно имеют несколько ЭЛ — для каждой вкладки и каждого сервис-воркера

Любой код во время выполнения блокирует ЭЛ, что может "повесить" даже пользовательский интерфейс, включая скролл и клики. Чтобы решить эту проблему, JavaScript полагается на коллбэки, промисы и async/await.

## The call stack

Стек вызовов — это место, откуда ЭЛ берёт задачи (вызывает их). Он действует по принципу "Last In, First Out", то есть как стакан: задачи берутся в работу, начиная с последних, попавших в очередь.

Один вызов = один цикл ЭЛ = один тик

Например, для кода
```js
const foo = () => {
  bar()
  baz()
}
```
очередь вызовов будет примерно следующая:
![](node-1.png)
То есть сначала в стек попадает `foo()`, потом вложенная функция `bar()`, которая выполняется, после неё выполняется `baz()`. Когда все вложенные команды выполнены, из очереди уже выходит `foo()` как выполненная.

## Очереди

Задачи попадают в стек из очередей и занима. Они отличаются срочностью. Приоритет выполнения будет следующий:

1. Обычные синхронные операции, каждая из которых выполняется в свой тик
2. `process.nextTick()` — принимает колбек, выполняемый в конце текущего тика, то есть до того, как ЭЛ возьмёт следующую задачу из стека
3. Message Queue — это очередь, в которую попадают колбэки таймеров, событий браузера (как кликов пользователя, так и события типа `onLoad`), а также `fetch`
4. Job Queue — очередь промисов, представленная в ES6. Имеет приоритет выше Message Queue

> [!INTERESTING] Так как нулевой таймер `setTimeout(()=>{}, 0)` стал популярным приёмом для отправки колбека в Message Queue, в Node и некоторых браузерах есть аналог `setImmediate(()=>{})`. Они очень похожи, но порядок их выполнения между друг другом определяется множеством факторов и практически непредсказуем

## Управление очередью

### Колбэки

Колбэк — это фунция, которая передаётся другой функции чтобы быть выполненной после остальной работы

Традиционно, колбэки были основным инструментом управления стэком вызовов в JavaScript. Их использовали в атрибутах html-элементов для вызова по событиям. Например:

```js
const heading = document.createElement('h1');
heading.onclick = () => {
	alert('Clicked on heading')
};
```

Позже появился более привычные нам в колбэках событий браузера:

```js
document
.getElementById('button')
.addEventListenet('click', () => {
	//do something
})
```

Колбэки также используются в таймерах

### Таймеры

Синтаксис таймера позволяет передать колбэку аргументы:
```js
setTimeout(myFunction, 2000, firstArg, secondArg);

setInterval(myFunction, 50, firstArg);
```

Функции setTimeout и setInterval возвращают id таймера, которые можно передать в качестве аргумента clearTimeout и clearInterval, чтобы уничтожить таймер. В том числе это можно использовать внутри таймеров:

```js
const intervalId = setInterval(() => {
	if (App.resultStatus == 'arrived') {
		clearInterval(intervalId)
		return
	}
})
```

Значение времени в setInterval определяет промежуток времени от старта до старта колбэка:
![](node-2.png)

Чтобы выставить время от окончания до старта выполнения колбэка, следует использовать рекурсивно setTimeout:

```js
const callback = (timerValue) => {
	// do something
	setTimeout(callback, timerValue)
}

setTimeout(callback, 1000, 1000)
```

### Промисы (ES6+)

Промис — это прокси (представитель) значения, которое станет доступно позже

У промиса есть несколько состояний:
- Pending — промис вызван функцией, продолжающей выполнение, пока промис работает над получением значения
- Resolved — значение получено
- Rejected — получить значение не удалось

Промисы используются под капотом Fetch, async/await и других интерфейсов, но мы можем создавать их самостоятельно через конструктор:
```js
let done = true

const isItDone = new Promise((resolve, reject) => {
	if (done) {
		resolve('Success!')
	} else {
		reject ('Still working')
	}
})
```

Колбэки resolve и reject принимают строки или объекты для коммуникации с вызывающей функцией.

Однако, чтобы использовать значение из промиса, его нужно обработать в колбэке:

```js
const checkDone = () => {
	isItDone
	.then((res) => {
		console.log(res)
	})
	.catch((err) => {
		console.err(err)
	})
}
```

Колбэки then и catch нужны для того, чтобы получить значение промиса, когда то будет готово.

Промисы могут возвращать другие промисы, создавая цепочку промисов, как бывает у fetch():

```js
// 1. Делаем запрос
fetch('/todos.json')
// 2. Обрабатываем статус промиса
.then((response) => {
	if (response.status === 200) {
		return Promise.resolve(response)
	}
	return Promise.reject(new Error(response.statusText))
})
// 3. Преобразуем в нужный формат
.then((response) => response.json())
// 4. Работаем со значением
.then((data) => console.log('data is here:', data))
// 5. Обрабатываем возможные ошибки
	.catch((err) => console.error('Request failed:', err))
```

### Async/await (ES2017+)

Async/await — это обновлённый механизм работы с асинхронными функциями, построенный на промисах и генераторах.

Как использовать?

1. Добавляем при определении функции слово async — теперь функция вернёт промис:
```js
const func1 = async () => {
	return 'test'
}
func1.then(console.log) // 'test'
```
2. Теперь добавляем await перед теми вызовами функций, которые возвращают промисы, чтобы остановить ход выполнения до тех пор, пока промис не вернёт значение или ошибку:
```js
const checkStatus = async () => {
	console.log(await funk1())
}
```

Сравнение промисов с колбеками и синтаксисом async/await:

| Promise+callbacks | Async/await |
|---|---|
| `const getFirstUserData = () => {return fetch('./users.json').then(response => response.json).then(users => users[0]).then(user => fetch('/users/'+user.name).then(userResponse => userResponse.json())}` | `const getFirstUserData = async () => {const response = await fetch('/users.json');const users = await response.json();const user = users[0];const userResp = await fetch('/user/'+user.name);const userData = await user.json();resurt userData}`|

# События

События знакомы всем, кто имел с ними опыт, используя JavaScript в браузере. Node имеет стандартный модуль `events`, который позволяет использовать ту же технику, создавая, запуская и реагируя на любые события.

`events` — это класс, который нужно импортировать и создать экземпляр для работы с ним

```js
// Пример из документации:
import { EventEmitter } from 'node:events';
class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();
// Пример Flavio Copes
const eventEmitter = require('events').EventEmitter()
```

Основные методы события:
- emit — запускает событие
- on — устанавливает колбэк события

emit поддерживает произвольное количество аргументов, но первый должен быть названием события. Остальные аргументы становятся аргументами функции-колбека:

```js
eventEmitter.on('created', (start, end) => {
	console.log(`started from ${start} to ${end}`)
})

eventEmitter.emit('created');
```

События по умолчанию синхронные (выполняются в порядке регистрации), но с помощью setImmediate(), process.nextTick() и других приёмов могут стать асинхронными.

Полный events API в документации: [https://nodejs.org/api/events.html](https://nodejs.org/api/events.html)
# Подключение БД

Node.js может работать с различными базами данных. Рассмотрим несколько из них:
## MySQL
MySQL — одна из самых популярных и простых в использовании реляционных БД для веб-разработки.

В примерах ниже будем использовать популярный npm-пакет [mysql](https://github.com/mysqljs/mysql):
```bash
npm install mysql
```

Далее с помощью `mysql.createConnection(option)` создаётся объект, через методы которого выполняется управление подключением: `.connect()`, `.query()`, `.end()`.

```js
const mysql = require('mysql')

const options = {
	user: 'foo',
	password: 'bar'
}

const connection = mysql.createConnection(options)

connection.connect(err => {
	if (err) console.error(err)
})

connection.query(
	'SELECT * FROM `todos` where `id` = ?',
	givenId,
	(err, result, fields) => {
		if (err) console.error(err)
		
		handleTodos(result);
	})
```



# Другие статьи
[HTTP](HTTP)
[NPM package](NPM%20package.md)
[Axios](Axios.md)
[WebSocket](WebSocket.md)

# Внешние ресурсы
Официальный сайт — [https://nodejs.org/en/download/](https://nodejs.org/en/download/)
Книга Flavio Copes "The Node.js Handbook" — https://flaviocopes.com/access/

