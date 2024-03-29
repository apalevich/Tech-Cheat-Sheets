Расширения (или аддоны) — это программы, которые позволяют изменять или расширять опыт пользователя в браузере.

Расширения используют стандартные веб-технологии: HTML, CSS и JavaScript. Они имеют доступ к стандартным браузерным API ([список](https://developer.mozilla.org/ru/docs/Web/API)), а также к специальным API для расширений. Таким образом, расширение позволяет сделать больше, чем обычная веб-страница.

Например:
- Дополнить информацию на сайте
- Изменить контент веб-страницы
- Изменить внешний вид браузера
- Расширить возможностей браузера
- Игры
- Инструменты разработки
# Совместимость

Хотя сейчас существует 2 стандарта — WebExtensions API и Extensions API для Chromium — на текущий момент браузеры их реализуют с отличиями, поэтому лучше следовать инструкциям из документации конкретного браузера.

|Web browser|Chromium-based?|Extension development webpage|
|---|---|---|
|Safari|No|[Safari App Extensions](https://developer.apple.com/documentation/safariservices/safari_app_extensions)|
|Firefox|No|[Browser Extensions](https://developer.mozilla.org/docs/Mozilla/Add-ons/WebExtensions)|
|Chrome|Yes|[API Reference](https://developer.chrome.com/extensions)|
|Opera|Yes|[Extensions Documentation](https://dev.opera.com/extensions)|
|Brave|Yes|Uses [Chrome Web Store](https://chrome.google.com/webstore/category/extensions)|
|Microsoft Edge|Yes|[Microsoft Edge Add-ons Developer](https://developer.microsoft.com/microsoft-edge/extensions)|
|Arc|Yes|[Extensions in Arc: How to Import, Add, & Open](https://resources.arc.net/en/articles/6452167-extensions-in-arc-how-to-import-add-open)|

Кроме того, расширение регламентируется метаданными в файле [Манифест](#Манифест), для которого актуальной является версия 3, но её не поддерживают Edge и Opera.

Для разработки расширений есть различные инструменты, которые могу облегчить процесс, например:
- [webextension-toolbox](https://github.com/webextension-toolbox/webextension-toolbox) — CLI для кросс-браузерной сборки
- [webpack-webextension-plugin](https://github.com/webextension-toolbox/webpack-webextension-plugin) — плагин для сборщика webpack
- [web-ext](https://extensionworkshop.com/documentation/develop/getting-started-with-web-ext/#testing-out-an-extension) — CLI чтобы облегчить и ускорить разработку
и другие.

В этой заметке я постараюсь указывать на различия для разных браузеров.
# Архитектура расширения

Точное содержание проекта расширения зависит от его функциональности. Но в любом случае его файлы будут относиться к одному из следующих типов:

- Манифест — *обязательный файл в корне c метаданными и необходимой информацией*
- HTML-страницы — *отображение расширения*
	- Попапы — *открывается при нажатии на иконку расширения*
	- Настройки — *страница для настройки расширения*
	- Боковые панели — *типы дополнительного пространства, которое располагается сбоку от содержимого веб-страницы и не перекрывает его*
	- Оверрайды — *замена стандартных страниц Chrome (закладки, история и пр.)*
	- и другие...
- Скрипты — *функциональность расширения*
	- Сервис воркеры — *скрипты, выполняющие в фоне действия и обрабатывающие события. Не зависят и не имеют доступа к веб-страницам*
	- Контент-Скрипты — *скрипты, которые выполняются в контексте веб-страниц, как будто они являются частью самой страницы.*
- Ассеты — *изображения, стили, шрифты и прочие ресурсы*

## Пример архитектуры расширения

Манифест определяет:
1. Иконку для магазина
2. Browser action — в примере это иконка в тулбаре, по нажатию на которую открывается popup (html+css+js)
3. Content script, который взаимодействует со страницей

Плюс тут находятся несколько изображений.

![](webextension-architecture-example.png)
# Манифест

Файл `manifest.json` — это единственный обязательный файл в расширении.

Он включает в себя мета-данные (версию, имя, описание) и определяет ресурсы, связанные с функциональностью и UI (скрипты, иконки).

Файл включает в себя обязательные, рекомендуемые и опциональные свойства:

```json
{
	// ОБЯЗАТЕЛЬНЫЕ МЕТАДАННЫЕ
	"manifest_version": 3, // Edge и Opera поддерживают версию =<2
	"name": "Hello Extensions", // название
	"version": "1.0", // версия расширения

	// ФУНКЦИОНАЛЬНОСТЬ
	"description": "Base Level Extension",
	"default_locale": "en", // только при наличии папки _locales
	"icons": { // ссылки на файлы иконок, рекомендован PNG
		"128": "icon.128.png", // обязательный размер
	},
	"author": {
		"email": "dev@example.com" // должно совпадать с адресом разработчика для публикации в Web Store
	},
	"content_scripts": [{
		// запускаются в контексты веб-страницы и DOM
	}]
}
```

Опциональных свойств намного больше. Полный список  — в документации [MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json) и [Chrome](https://developer.chrome.com/docs/extensions/mv3/manifest/).

Свойства Манифеста можно получить в самом расширении через функцию `browser.runtime.getManifest()`.

> [!WARNING] Манифест поддерживает комментарии, начинающиеся с `//`, во время разработки, но перед публикацией в Chrome Web Store они должны быть удалены

# Пользовательский интерфейс

Мы начнём с пользовательского интерфейса (UI).

Action — это действие, которое происходит при запуске расширения (как правило, по клику на его иконку). Оно регулируется Action API.

Есть несколько способов создать UI на этом этапе:
- [Попап](Chrome%20Extensions.md#Попап)
- [Боковые панели](#Боковая%20панель)
- [Адресная строка (Омнибокс)](#Адресная%20строка%20(Омнибокс))
- [Контекстное меню](#Контекстное%20меню)
- [Произвольный скрипт](#Произвольный%20скрипт)
## Попап

Попап — это UI, который отображается поверх браузера. Он представляет из себя обычную HTML-страницу. В свою очередь, эта веб-страница может содержать код CSS и JS (в тегах или в подключаемых файлах).

Размеры попапа: от 25х25 до 800х600 пикселей.

```json
{
...
	"action": { 
		"default_icon": "images/icon16.png",  // путь или объект, 32х32 рекомендовано
		"default_popup": "popup.html",  // путь к странице
	    "default_title": "Click Me",   // используется при наведении и на скринридерах
	},
...
}
```

Попап может быть установлен динамически через `action.setPopup()`:

```js
chrome.storage.local.get('signed_in', (data) => {
	if (data.signed_in) {
		chrome.action.setPopup({
			popup: 'popup.html'
		});
	} else {
		chrome.action.setPopup({
			popup: 'popup_sign_in.html'
		});
	}
});
```

## Боковая панель

Другой вариант отображения UI — это боковая панель (sidePanel). Её основные отличия:

- Не перекрывает контент веб-страниц
- Может оставаться открытой при переключении между вкладками

Для её настройки обязательно добавить соответствующие свойства в Манифест:

```json
{
	"permissions": [
		"sidePanel"
	],
	"side_panel": {
		"default_path": "index.html"
	}
}
```

По умолчанию такая боковая панель доступна на любом сайте. Чтобы её ограничить, нужно использовать объект `chrome.sidePanel.setOptions` в СВ. Подробнее — в спецификации [SidePanel API](https://developer.chrome.com/docs/extensions/reference/sidePanel/)

## Адресная строка (Омнибокс)

Чтобы включить эту функциональность, необходимо задать ключевое слово в Манифесте:

```json
{	
	"omnibox": { "keyword" : "aaron" }
}
```

Теперь, если ввести ключевое слово в адресную строку, то после пробела пользователь зачинает взаимодействовать с расширением:

![](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/T0jCZDUVfuEANigPV6bY.png)

Подробнее — в спецификации [Omnibox API](https://developer.chrome.com/docs/extensions/reference/omnibox/)
## Контекстное меню

Для включения контекстного меню нужно соответствующее разрешение в Манифесте:

```json
{
	"permissions": ["contextMenus"],
	"background": {
	    "service_worker": "background.js",
	}
}
```

После этого в контекстном меню страницы появится новый пункт с иконкой расширения в начале:
![](https://wd.imgix.net/image/BrQidfK9jaQyIHwdw91aVpkPiib2/jpA0DLCg2sEnwIf4FkLp.png)
Наполнение меню настраивается по событию `onInstalled` через `contextMenu.create()`:

```js
chrome.runtime.onInstalled.addListener(async () => {
	chrome.contextMenus.create({
		id: 123,
		title: 'Do something'
	});
}});
```

Подробнее — в спецификации [ContextMenus API](https://developer.chrome.com/docs/extensions/reference/contextMenus/)
## Произвольный скрипт

Вместо попапа можно выполнить скрипт через хендлер Action API: `chrome.action.onClicked`

> [!WARNING] Этот способ не сработает если установлен `default_popup`

Пример: запустим скрипт на открытой вкладке:

*manifest.json:*
```json
{
...
	"action": {
		"default_title": "Click to show an alert"  },
		"permissions": ["activeTab", "scripting"], 
		"background": {
			"service_worker": "background.js"
		}
	}
...
```

*background.js*:
```js
chrome.action.onClicked.addListener((tab) => {
	chrome.scripting.executeScript({
		target: {tabId: tab.id},
		files: ['content.js']
	});
});
```

*content.js*
```js
alert(`Robot time: ${Date.now()}`);
```

Больше о возможностях скриптов — в спецификации [Action API](https://developer.chrome.com/docs/extensions/reference/action/#method)

# Контент скрипты

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


## Сервсис воркеры (Фоновые скрипты)

Сервис воркеры (они же background scripts или фоновые скрипты) в расширениях Хрома — это центры обработки событий. Они предназначены для обработки данных и управлением различными частями расширения.

Но есть и ряд общих черт:
- СВ загружается когда он нужен и выгружается когда уходит в спячку
- По загрузке он работает до тех пор, пока получает события + таймаут
- СВ не имеет доступа к DOM

СВ имеет доступ ко всем [API расширения](https://developer.chrome.com/docs/extensions/reference/) , но выбранное API нужно указать в Манифесте и получить разрешение пользователя. СВ не может использовать DOM API и изменять контент веб-страницы.



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
