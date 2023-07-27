Пирамида тестирования:
1. Unit/Модульные тесты — проверяют компонент, модуль или функцию изолированно. Их должно быть как можно больше
2. Интеграционные тесты — проверяют взаимдействие функций/классов/компоненов
3. E2E/Сквозные тесты — проверяют систему полностью

Зачем нужны тесты:
- Гарантии корректной работы системы
- Снижение рабочей нагрузки на разработчиков и тестеров
- Ускорение разработки
- Ускорение рефакторинга

# Особенности

## Преимущества:
- Работает внутри браузера (в отличие от Selenium, например), задействуя его движок
- Поддержка браузерных DevTools
- Интерактивный визуальный UI
- Простой синтаксис

## Недостатки
- Не подходит для кроссбраузерного тестирования — поддерживает только движок Chromium

# Установка
  1. Загрузка пакета
```sh
yarn add cypress -D
```

2. Интеграция в проект
```json
"scripts": {
	"cy:open": "cypress open", // Запуск GUI для использования в разработке
	"cy:run": "cypress run" // Запуск headless для CI/CD
}
```

3. Написание тестов:
	- Тесты создаются в виде отдельных файлов с расширением *.spec.js* (также поддерживается *.test.js*)
	- Тесты размещаются в папке `/integration`
	- Можно создать подпапки по страницам или бизнес-фичам
	- Для констант домена и других опций можно использовать файл [/cypress.json](https://docs.cypress.io/guides/references/legacy-configuration#cypressjson)

# Синтаксис

Cypress использует синтаксис Mocha, который называется BDD

```js
describe('My first test', () => { // Название для группы тестов
	
	before(() => {...}) // хук для действий до начала тестов
	
	it('Visits the example', () => { // Название теста
		cy.visit('/');               // Последовательность действий через API
		cy.get.('[data-cy="section"]').contains('Home page')
	})
	
	after(() => {...}) // хук для действий после тестов
})
```

Шпаргалка по синтаксису: https://gist.github.com/samwize/8877226

# Использование

## Взаимодействие с DOM-элементами

### Проверка адреса после нажатия на ссылку
```js
it('Clicks the menu link', () => {
	cy.visit('/')
	cy.get('href="/contacts"').click()

	cy.location('pathname').should('eq', '/contacts') // получает путь и смотрит соответствие
})
```
> [!INFO] Как и человек, Cypress может кликнуть только на кнопку, находящуются в поле видимости (но это можно обойти с помощью параметра `force`)

### Отправка формы
```js
it('Input data', () => {
	cy.visit('/contacts')
	cy.get('[data-cy="email"]').type('asd@qwe.rty')
	cy.get('[data-cy="comment"]').type('Lorem Ipsum')
	cy.get('[data-cy="submit-button"]').click()
})
```

> [!INFO] Cypress вводит текст в поля приближенно к тому, как это делает человек — посимвольно и с меняющейся скоростью

## Работа с сетевыми запросами
### Проверка переадресации после аутентификации
```js
describe('Login page', () => {
	it('Visits contact page', () => {
		cy.visit('/auth')
	})

	it('Input data', () => {
		cy.get('data-cy="username"').type('j0hnd0e')
		cy.get('data-cy="password"').type('qwerty123')
		cy.get('data-cy="submit-button"').click
		cy.location('/pathname').should('eq', '/')
		cy.contains('Welcome back, John Doe!')
	})
})
```

### Ожидание окончания сетевого запроса
```js
it('Intercept', () => {
	cy.visit('/search')
	cy.intercept('GET', '/api/pa/search?q=*').as('search') // сохраняем метод и regex запроса, который нужно отловить
	cy.get('[data-cy="query"]').type('Alabama')
	cy.get('[data-cy="submit"]').click()
	cy.wait('@search')
	cy.get('[data="paragraph"]').find('a').should('have.length', 2)
})
```

## Управление состоянием

Например, для проверок страниц, закрытых авторизацией, необходимо создать универсальную команду "сходи авторизуйся"

Команды находятся в файле `/support/commands.js`
```js
Cypress.Commands.add('login', () => {
	cy.request({
		url: `${URL_API}/api/auth/login`,
		method: 'POST',
		failOnStatusCode: false,
		body: {
			username: 'j0hnd0e',
			password: 'qwerty123'
		}
	})
	.then(({body}) => {
		cy.setCookie('access_token_key', body.access_token)
	})
})

// Чтобы тесты не падали при ошибке команд, надо чтобы внизу было:
Cypress.on'uncaught:exception', (err, runnable) => {
	return false
})
```

Теперь команда доступна в виде метода, который можно использоваться в хуке:

```js
describe('My test', () => {
	beforeEach(() => {
		cy.login()
		cy.visit('/')
	})

	it('Greetings', () => {
		cy.contains('Hello')
	})
})
```

## Фикстуры

Фикстура позволяет имитировать получения объект от бэкенда, если тот не готов к использованию:

Например, `/fixtures/current_user.json`:
```js
{
	"id": 999,
	"username": "Foo",
	"email": "b@a.r",
}
```

Использование в тесте:
```js
describe('Fixture use', () => {
	beforeEach(() => {
		cy.visit('/')
		cy.fixture('current.user') // название файла
		.then((user) => {
			cy.intecept('GET', '/api/user/current', user).as('current_user')
			// подменяем user на current_user
		})
	})

	it('Home and greetings', () => {
		cy.contains('Привет, Foo!')
	})
})
```

# Хорошие практики

1. Разные `.it()` должны быть независимы друг от друга (для проверки можно попробовать сбрасывать страницу в начальное состояние при помощи хука `beforeEach()`)
2. Использовать только относительные пути, добавив домен в качестве значени baseUrl в `/cypress.json`
3. В методе `.get()` можно указывать любые CSS-селекторы, но лучше использовать data-атрибуты и вырезать их при сборке билда
4. В аргументы `.wait()` передавайте используйте алиасы запроса `.intercept`. Числа (как миллисекунды) можно передавать только при работе с анимациями

# Внешние ссылки

Документация Cypress — https://docs.cypress.io/
Источник конспекта — https://youtu.be/rZ_DIDNnlfs
Шпаргалка по синтаксису — https://gist.github.com/samwize/8877226