WebSocket (WS) — это протокол обмена данными в сети. Он является альтернативой HTTP, предоставляя следующие преимущества:

- Сервер и клиент могут посылать сообщения друг другу
- Сервер и клиент могут обмениваться данными одновременно (full-duplex)
- Требуется меньше дополнительных данных, что уменьшает задержку

При этом WS требует больше настройки и кода. Поэтому WS наилучшим образом подходит для случаев, когда нужно живущее соединение и обмен real-time данными.

# WebSocket как протокол

Хотя WS является альтернативой HTTP, работая поверх TCP, он разработан так, чтобы быть совместимым с ним, используя порты, прокси и прочее от HTTP.

Как устанавливается соединение:

1. Сначала клиента  инициализирует соединение (handshake) через HTTP: посылает запрос с заголовками `Upgrade: WebSocket`, `Connection: Upgrade` (могут быть и другие, специфические для WS)
2. Сервер прислает HTTP-ответ с кодом `101 Switching Protocols` и теми же заголовками, подтверждая переход на прокол WS. Если сервер не поддерживает WS, от него принято ожидать код `426 Upgrade Required`
3. После этого соединение установлено и можно начинать передачу данных по протоколу `ws://` вместо HTTP и `wss://` вместо HTTPS

![](websocket.png)

> [!NOTE] `ws://` и `wss://` соответствуют стандарту [URI](https://en.wikipedia.org/wiki/Uniform_Resource_Identifier), поэтому могут читаться как обычные адреса сайтов, за исключением поддержки # и символов после

# The WebSocket API в JavaScript

Реализация WS на клиенте происходит через новый объект конструктора WebSocket. Он принимает в качестве аргумента URL и выполняет handshake автоматически. После этого управление подключением и данными идёт через события:

```js
// Создание объекта для подключения
const socket = new WebSocket("ws://localhost:8080");

// Выполняется когда подключение открыто
socket.addEventListener("open", (event) => {
  socket.send("Hello Server!");
});

// Выполняется когда приходит сообщение от сервера
socket.addEventListener("message", (event) => {
  console.log("Message from server ", event.data);
});

// ВМЕСТО addEventListener можно передавать обработчики в свойства событий:

// Когда подключение закрывается
socket.onclose = function(event) {
  console.log("Connection has been closed");
};

// Когда подключение закрывается с ошибкой
socket.onerror = function(error) {
  console.log("Connection has been closed because of an error:", error);
};
```

# WebSocket на Node.js

Для работы с WS среде сервера Node.js обычно используют популярную библиотеку [ws](https://github.com/websockets/ws)

Она использует похожий API событий, которые могут быть обёрнуты друг в друга:

```js
import { WebSocketServer } from 'ws'

const params = {
	port: 8080,
	host: 'wss://websocket-echo.com/'
}

const wss = new WebSocketServer(params)

wss.on('connection', ws => {
  ws.on('error', console.error)

  ws.on('message', message => {
    ws.send('Got your message')
  };

  ws.send('something')
});
```

# Работа с несколькими клиентами
В некоторых приложениях (например, групповых чатах) обновление данных на сервере должно уведомлять все подключенные клиенты об изменениях.

Один из способов это сделать — с помощью библиотеки [Socket.io](https://socket.io), который добавляет в интерфейс необходимый метод `emit`.
# Внешние источники
MDN — https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API
Стандарт — https://datatracker.ietf.org/doc/html/rfc6455
Книга Flavio Copes "Node.js" — https://flaviocopes.com/access/