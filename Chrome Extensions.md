Расширения браузера Google Chrome (далее — Хром). Они используют стандартные технологии для веб-приложений:
- HTML для разметки
- CSS для стилизации
- JavaScript для логики

Расширения имеют доступ как к стандартным браузерным API ([список](https://developer.mozilla.org/ru/docs/Web/API)), так и к специфическим Chrome API ([подробнее](https://developer.chrome.com/docs/extensions/reference/)

# Структура
Точное наполнение расширения зависит от его функциональности. Чаще всего можно встретить следующие файлы:

- Манифест — обязательный файл `manifest.json` в корневой директории, содержащий метаданные и необходимую информацию том, чем является расширение
- Сервис воркер — обработчик браузерных событий. Он не может взаимодействовать с веб-страницей напрямую
- Скрипты — JS-код, который может взаимодействовать с веб-страницей: читать, менять DOM и тд. Он может получать данные через Chrome API или сервис-воркер
- Различные страницы — HTML-код с доступом к Chrome API

## Манифест
Файл `manifest.json` — это единственный файл в составе расширения, который обязательно:
- должен существовать
- находиться в корне проекта

Файл включает в себя обязательные, рекомендуемые и опциональные свойства:

```json
{
	// ОБЯЗАТЕЛЬНЫЕ
	"manifest_version": 3, // неизменяемо
	"name": "Hello Extensions", // название расширения
	"version": "1.0", // версия

	// РЕКОМЕНДУЕМЫЕ
	"description": "Base Level Extension",
	"default_locale": "en", // только при наличии папки _locales
	"icons": { // ссылки на файлы иконок
		// поддерживаются форматы PNG, GIF, ICO, JPEG, Blink, BMP
		"128": "icon.128.png", // обязательный размер
	},
	"action": {
		"default_icon": {              // иконка для тулбара
	      "16": "images/icon16.png",
	    },
	    "default_title": "Click Me",   // используется при наведении и на скринридерах
	    "default_popup": "popup.html"  // окно, всплывающее по клику (от 25х25 до 800х600)
	},

	// ОПЦИОНАЛЬНЫЕ
	"author": {
		"email": "dev@example.com" // должно совпадать с адресом разработчика для публикации в Web Store
	},
	"content_scripts": [{
		// запускаются в контексты веб-страницы и DOM
	}]
}
```

Опциональных свойств намного больше. Полный список  — [в документации](https://developer.chrome.com/docs/extensions/mv3/manifest/)

## Сервис воркеры

Сервис воркеры в расширениях Хрома — это центры обработки событий. Они отличаются от обычных сервис воркеров в вебе

Главные отличия:
- СВ в расширениях получают дополнительные события браузера (н. взаимодействие с вкладками)
- Иначе регистрируются и обновляются

Но есть и ряд общих черт:
- СВ загружается когда он нужен и выгружается когда уходит в спячку
- По загрузке он работает до тех пор, пока получает события + таймаут
- СВ не имеет доступа к DOM

### Этапы жизненного цикла

#### Установка
Этим этапом считается как непосредственно установка, так и обновление расширения. Неважно, было ли оно установлено из Web Store или локально.

| Событие | Срабатывает |
|------|---|
| `ServiceWorkerRegistration.install` | во время установки |
| `chrome.runtime.onInstalled` | когда расширение установлено, обновлено или браузер обновлён |
|  `ServiceWorkerRegistration.active` | по окончанию установки и событию activate |

#### Запуск

Имеет только событие `chrome.runtime.onStartup`, которое срабатывает по запуску профиля пользователя

#### Бездействие и остановка

Стандартно Хром останавливает СВ в одном из следующих случаев:
- По таймауту 30 секунд бездействия или с последнего события или вызова Extension API
- Ответ внутри fetch() длится дольше 30 секунд
- Запрос обрабатывается дольше 5 минут

Документация требует делать СВ устойчивыми к непредвиденным остановкам. Для этого вместо глобальных переменных следует сохранять данные в одно из следующих хранилищ:

- __chrome.storage API__ — local, session, managed (domain) и sync-хранилища,  которые устойчивы к ручной очистке веб-кэша браузера. Прост с использовании, но подходит только для JSON
- __IndexedDB API__ — низкоуровневый API. Сложен в использовании, но подходит для хранения файлов и бинарных данных
- __CacheStorage API__ — специфическое хранилище для пар запросов и ответов, разработанное специально для СВ.

Подробнее о СВ — см. [The Offline Cookbook](https://web.dev/offline-cookbook/)

## Контент скрипты

КС — файлы, которые выполняются в контексте открытой веб-страницы, используя DOM API. Тем не менее, они выполняются в песочнице — "Isolated world", что помогает избежать конфликтов со скриптами на странице или в других расширениях. Таким образом, КС имеют доступ к странице, а она к ним — нет.

Например, у нас есть следующая страница:
```html
<button id="mybutton">click me</button>
<script>
	var greeting = "hello, ";
	var button = document.getElementById("mybutton");
	button.person_name = "Bob";
	button.addEventListener(
		"click", () => alert(greeting + button.person_name + "."), false
	);
</script>
```

И на ней запущено расширение со следующим КС:

```js
var greeting = "hola, ";
var button = document.getElementById("mybutton");
button.person_name = "Roberto";
button.addEventListener(
	"click", () => alert(greeting + button.person_name + "."), false
	);
```

Результатом будет последовательность алертов по нажатию на кнопку.

### Размещение скриптов

КС могут быть объявлены статически, динамически или программно

#### Статическое объявление

Используется для файлов, которые должны запускаться автоматически на заранее известных страницах. Работает через указание в манифесте в свойствах объекта `content_scripts`:

```json
{
	...
	"content_scripts": [{
		"matches": ["https://*.example.com/*"], // обязательно, указывет страницы для запуска КС
		"exclude_matches": ["https://*.example.com/business*"], // для исключений страниц
		"css": ["styles.css"], // стили, добавляемые на страницы перед созданием DOM, в порядке расположения в массиве
		"js": ["content-script.js"] // аналогично, для скриптов
	}]
}
```

Менее популярные свойства указаны [в документации](https://developer.chrome.com/docs/extensions/mv3/content_scripts/#static-declarative)

#### Динамическое объявление

Похоже на статическое объявление, но:
- используется не в манифесте, а в СВ
- использует Scripting API (через `chrome.scripting`), чьи методы позволяют
	- регистрировать КС,
	- получать список зарегистрированных КС,
	- обновлять список зарегистрированных КС,
	- удалять зарегистрированные КС
- позволяет определять параметры динамически

##### Примеры

Регистрация:
```js
chrome.scripting.registerContentScripts([{
	id: "session-script",
	js: ["content.js"],
	persistAcrossSessions: false,
	matches: ["*://example.com/*"],
	runAt: "document_start",
}])
.then(() => console.log("registration complete"))
.catch((err) => console.warn("unexpected error", err))
```

Обновление:
```js
chrome.scripting
.updateContentScripts([{
	id: "session-script",
	excludeMatches: ["*://admin.example.com/*"],
}])
.then(() => console.log("registration updated"));
```

Удаление:
```js
chrome.scripting
.unregisterContentScripts({ ids: ["session-script"] })
.then(() => console.log("un-registration complete"));
```

#### Программное объявление

Используется, когда КС нужно запустить в ответ на событие или специфические условия. Требует специальных разрешений, запрашиваемых в манифесте.

##### Пример

Добавим необходимые разрешения в манифест:
```json
{
	...
	"permissions": [
		"activeTab",
		"scripting"
	],
	"background": {
		"service_worker": "background.js"
	},
	"action": {
		"default_title": "Action Button"
	}
}
```

Теперь подготовим КС:
```js
document.body.style.backgroundColor = "orange";
```

И используем его в СВ:
```js
chrome.action.onClicked.addListener((tab) => {
	chrome.scripting.executeScript({
		target: { tabId: tab.id },
		files: ["content-script.js"] // может быть заменен на func с функцией из этого же файла
	});
});
```

**Либо** вместо файла с КС можно передать функцию:
```js
function changeBackground(color) {
	document.body.style.backgroundColor = color;
}

chrome.action.onClicked.addListener((tab) => {
	chrome.scripting.executeScript({
		target: { tabId: tab.id },
		func: changeBackground,
		args: ["orange"],
	});
});
```

> [!IMPORTANT] Функция в методе executeScript() является копией переданной функцией, поэтому ссылки на переменные за пределами объявления функции повлекут за собой ошибку

### Доступ к файлам проекта

КС имеет доступ к файлам расширения несколькими способами.

1. Сначала они должны быть объявлены в манифесте:
```json
{
...
"web_accessible_resources": [
	{
		"resources": [ "images/*.png" ],
		"matches": ["https://example.com/*" ]
	}
],
...
}
```

2. Теперь в JS можно использовать интерфейс chrome.runtime.getURL():
```js
let image = chrome.runtime.getURL("images/my_image.png")
```
А в CSS — @@extension_id:
```css
body {
	background-image:url('chrome-extension://__MSG_@@extension_id__/background.png');
}
```

# Локализация
https://developer.chrome.com/docs/extensions/reference/i18n/

# Публикация в Chrome Web Store
https://developer.chrome.com/docs/extensions/mv3/single_purpose
https://developer.chrome.com/docs/webstore/cws-enterprise/
https://developer.chrome.com/docs/webstore/publish/

# Внешние ссылки
Гайды от Google: https://developer.chrome.com/docs/extensions/mv3/getstarted/extensions-101/
Cписок стандартных браузерных API — https://developer.mozilla.org/ru/docs/Web/API
Документаци Chrome API — https://developer.chrome.com/docs/extensions/reference/
