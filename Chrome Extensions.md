Расширения Google Chrome (далее Хром) — это программы, которые позволяют изменять опыт работы пользователя в Хроме.

Они используют стандартные веб-технологии:
- HTML для разметки
- CSS для стилизации
- JavaScript для логики

Расширения имеют доступ как к стандартным браузерным API ([список](https://developer.mozilla.org/ru/docs/Web/API)), так и к специальным Chrome API ([подробнее](https://developer.chrome.com/docs/extensions/reference/))

# Основные сущности

Точное содержание проекта расширения зависит от его функциональности. Но в любом случае его файлы будут относиться к одному из следующих типов:

- Манифест — обязательный файл `manifest.json` в корневой директории, содержащий метаданные и необходимую информацию том, чем является расширение
- Контент-Скрипты — могут взаимодействовать со страницей, DOM, получать данные через Chrome API или сервис-воркер
- Сервис-воркеры — обработчик браузерных событий. не может взаимодействовать с веб-страницей напрямую
- Ассеты — HTML-страницы с доступом к Chrome API, изображения, стили, шрифты и прочее. Мы их не будем 

## Манифест

Файл `manifest.json` — это единственный файл, который должен находиться в расширении. Остальные опциональны.

Файл включает в себя обязательные, рекомендуемые и опциональные свойства:

```json
{
	// ОБЯЗАТЕЛЬНЫЕ
	"manifest_version": 3, // неизменяемо
	"name": "Hello Extensions", // название
	"version": "1.0", // версия

	// РЕКОМЕНДУЕМЫЕ
	"description": "Base Level Extension",
	"action": {
		"default_icon": "images/icon16.png",  // иконка тулбара
		"default_popup": "popup.html",  // страница по клику (от 25х25 до 800х600)
	    "default_title": "Click Me",   // используется при наведении и на скринридерах
	},
	"default_locale": "en", // только при наличии папки _locales
	"icons": { // ссылки на файлы иконок
		// рекомендован PNG
		"128": "icon.128.png", // обязательный размер
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

> [!WARNING] Манифест поддерживает комментарии, начинающиеся с `//`, во время разработки, но они должны быть удалены перед публикацией в Chrome Web Store

## Контент скрипты

КС — файлы, которые выполняются в контексте открытой веб-страницы, используя DOM API. Тем не менее, они выполняются в [песочнице](https://developer.chrome.com/docs/extensions/mv3/content_scripts/#isolated_world), что помогает избежать конфликтов со скриптами на странице или в других расширениях. Таким образом, КС имеют доступ к странице, а страница к ним — нет.

> [!INFO] **Host pages** — это название страниц, с которыми  взаимодействует КС

Например, у нас есть следующая страница:
```html
<button id="mybutton">click me</button>
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

КС должны быть объявлены в Манифесте:

```json
{
	...
	"content_scripts": [
		{
			"css": ["my-styles.css"], // относительный путь файла CSS...
			"js": ["content-script.js"], // или JS
			"matches": ["https://*.example.com/*"], // Где выполнять скрипт (обязательно)
			"exclude_matches": ["*://*/*foo*"],
			"include_globs": ["*example.com/???s/*"],
			"exclude_globs": ["*bar*"],
			"all_frames": false,
			"match_origin_as_fallback": false,
			"match_about_blank": false,
			"run_at": "document_idle",
			"world": "ISOLATED",
		}
	],
}
```

### Объявление скриптов

В большинстве случаев КС объявляются статически — записываются в манифест. Но если это не подходит, то можно их объявлять динамически или программно. Ниже все три способа
#### Статическое объявление

Используется для файлов, которые должны запускаться автоматически на заранее известных страницах. Работает через указание в манифесте в свойствах объекта `content_scripts`:

```json
{
	...
	"content_scripts": [{
		"matches": ["https://*.example.com/*"], // страницы для запуска КС, обязательно
		"exclude_matches": ["https://*.example.com/business*"], // для исключений страниц
		"js": ["content-script.js"], // скрипты, добавляемые на страницы
		"css": ["styles.css"], // стили, добавляемые на страницы
	}]
}
```

Паттерны свойства matches указываются по схеме `<scheme>://<host><path>`, более подробное описание [тут](https://developer.chrome.com/docs/extensions/mv3/match_patterns/). Это свойство влияет на содержание предупреждения для пользователя:

![](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/rKDdOyri9x8VkhTEXbO6.png?auto=format&w=676)

#### Динамическое объявление

Похоже на статическое объявление, но:
- позволяет определять параметры динамически
- указывается не в манифесте, а в СВ
- использует Scripting API (через `chrome.scripting`), чьи методы позволяют
	- регистрировать КС,
	- получать список зарегистрированных КС,
	- обновлять список зарегистрированных КС,
	- удалять зарегистрированные КС

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

Добавим необходимые разрешения в манифест:
```json
{
	//...
	"permissions": [
		"activeTab",
		"scripting",
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


## Сервис воркеры

Сервис воркеры в расширениях Хрома — это центры обработки событий. Они предназначены для обработки данных и управлением различными частями расширения.

СВ имеет доступ ко всем [API расширения](https://developer.chrome.com/docs/extensions/reference/), но не может использовать DOM API и изменять контент веб-страницы

СВ в расширениях Хрома отличаются от обычных сервис воркеров в вебе.
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

# Передача сообщений

Мы рассмотрели части, из которых состоят расширения. Теперь рассмотрим способы коммуникации между ними.

Сообщения — это универсальный способ передачи информации между частями одного расширения или даже от одного расширения к другому. Сообщения организованы в каналы, по которым стороны слушают и отвечают.

Содержание сообщения — любой валидный JSON, то есть null, boolean, number, string, array или object.

Для Cообщений существуют 3 API:
- одноразовые\запросы
- продолжительные соединения
- коммуникация между расширениями
## Одноразовые запросы

Реализуются через `runtime.sendMessage()` в КС или `tabs.sendMessage()` в СВ.

Для обработки запроса необходимо использовать Promise (рекомендуется), либо колбэк-функцию.

Так посылается сообщение от КС к СВ:
```js
(async  () => {
	const response = await chrome.runtime.sendMessage(payload);
	console.log(response);
})();
```

Отправка  от СВ к КС похожа, но нужно дополнительно указать вкладку. В данном случае указываем открытую вкладку:

```js
(async () => {
	// получаем данные вкладки
	const [tab] = await chrome.tabs.query({active: true, lastFocusedWindow: true});
	// посылаем сообщения во вкладку
	const response = await chrome.tabs.sendMessage(tab.id, payload);
	console.log(response);
})();
```

Принимающая сторона должна иметь листенер чтобы получить и обработать отправленное сообщение. Листенер реализуется одинаково в КС и СВ:

```js
chrome.runtime.onMessage.addListener(
	function (request, sender, sendResponse) {
		console.log(sender.tab
			? `from a content script: ${sender.tab.url}`
			: `from the extension`);
		sendResponse({message: `Hello, ${request.name}`});
	}
)
```

> [!faq] В примере выше `sendResponse()` вызван как синхронная функция. Её можно вызвать асинхронно, если добавить `return true` в обработчик событий `onMessage` (TODO: добавить что это значит + пример)

> [!WARNING] Если листенер onEvent указан на нескольких страницах, сработает только первый `sendResponse()`, остальные будут проигнорированы

## Продолжительные соединения

# Хранение данных

# Доступ к файлам проекта

HTML-файлы могут использовать ассеты через стандартные HTML-тэги.

КС имеет доступ к файлам в проекте расширения в два этапа.
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

А в CSS ассеты доступны через `@@extension_id`:
```css
body {
	background-image:url('chrome-extension://__MSG_@@extension_id__/background.png');
}
```

# Тестирование

Самый простой способ протестировать расширение — открыть в браузере chrome://extensions, включить "Developer mode", нажать кнопку "Load unpacked" и указать папку проекта.

Расширение сразу будет добавлено в браузер. Его можно отметить булавкой, чтобы закрепить справа от адресной строки.

По умолчанию расширение обновляется каждый раз при сохранении файлов в проекте. Это можно отключить на странице [Расширений](chrome://extensions):

![](https://wd.imgix.net/image/BhuKGJaIeLNPW9ehns59NfwqKxF2/4Ph3qL9aUyswxmhauRFB.png?auto=format&w=1000)

Для дебага расширения можно использовать инспектор, который вызывается по правой кнопке мыши на попапа расширения или его иконке.

Также ошибки будут показываться на странице [Расширения](chrome://extensions) по нажатию на кнопку Errors
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
