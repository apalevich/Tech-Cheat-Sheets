# Основы

> OpenAI API может применяться к любым задачам, которые подразумевают понимание или создание текстов, кода или изображений.
> *— Официальная документация*

## Термины

**Модель (Model)** — в области ML так называют любой систему, которая подготовлена для выполнения конкретных целей. Текстовые модели в OpenAI называют "GPT", которые обучены для обработки и генерации естественного (human-like) языка.

**GPT (Generative Pre-trained Transformer)** — это тип моделей ML для работы с текстом, который используется в OpenAI. Хотя "трансформеры" являются популярным подходом в ML, термин "GPT" подразумевает модели именно компании OpenAI.

**Запрос (Prompt)** — это входные данные для модели. Создание промпта обычно подразумевает "программирование" модели, как правило, через инструкции или примеры по выполнению конкретных задач.

**Ассистент (Assistant)** — это мета-сущность OpenAI API, которая оперирует в рамках контекста, который задан промптом. Ассистент может использовать и нетекстовые инструменты OpenAI (например, обрабатывать файл или генерировать изображения).

**Эмбеддинг (Embedding)** — это репрезентация данных на языке ML (аналогично байт-коду для компьютеров). В эмбеддинге важно, чтобы похожие по смыслу данные имели похожую репрезентацию.

**Токены (Tokens)** — единица текста, с которой работает модель. Представляет из себя короткое слово или часть длинного слова. Часто начинается с пробела. Для английского языка 1 токен ≈ 4 символа ≈ 0.75 слова. Для GPT размер в токенах указывает на сумму "промпт+результат"
## Модели

OpenAI API содержит множество моделей, которые делятся по возможностям и стоимости.

|Группа моделей|Описание|
|-|-|-|
| GPT-4 | Модели для текста и кода (улучшенные и с большим контекстом) |
| GPT-3.5 | Модели для текста и кода |
| DALL-E | Генерирует и редактирует изображения|
| Whisper | Преобразует речь в текст |
| Whisper | Преобразует речь в текст |

Каждый запрос в API необходимо сопровождать идентификатором модели.
Полный список моделей и их идентификаторов: https://platform.openai.com/docs/models

Если не уверены, с какой модели начать, используйте `gpt-3.5-turbo` или `gpt-4`
## Промпт

Качественный промпт — залог того, что модель ответит наилучшим образом. Вот основной алгоритм:

1. Cоставь запрос
2. Добавь проверенные данные (напр. примеры)
3. Проверь настройки

Чтобы улучшить навык писать качественные промпты, можно пройти [курс от OpenAI и DeepLearning](https://learn.deeplearning.ai/chatgpt-prompt-eng) или посмотреть файл [ChatGPT Prompt Engineering for Developers](ChatGPT%20Prompt%20Engineering%20for%20Developers.md Prompt Engineering for Developers>)

> [!NOTE] Самый эффективный способ разработки нужного промпта — итерациями: напиши простой промпт, проверь результат, добавь уточнение, проверь результат, добавь ещё и так далее.
# Подготовка

## Настройка аккаунта

Для аутентификации в API необходимо сгенерировать ключ тут: https://platform.openai.com/account/api-keys

## Установка SDK

Хотя работать с OpenAI API можно обычными веб-запросами, мы будем использовать SDK для [Node.js](Node.js.md). После установки Node.js в директории проекта необходимо выполнить команду:

```shell
npm install openai
```

### Тестовый запрос

После установки можно сделать тестовый запрос:

```js
import OpenAI from "openai";

const openai = new OpenAI({apiKey: "USE_YOUR_KEY_HERE"});

async function main() {
  const completion = await openai.chat.completions.create({
    messages: [{ role: "system", content: "You are a helpful assistant." }],
    model: "gpt-3.5-turbo",
  });

  console.log(completion.choices[0]);
}

main();
```


# Assistant API

Я начну объяснение с Assistant API. 

Ассистент — это применение моделей GPT, оптимизированных для поддержания разговора на заданную тему. Можно сказать, что это высокоуровневая надстройка над GPT.

> [!tip] Принципиальная особенность Ассистентов: вы имеете более простой интерфейс за счёт меньшего контроля над внутренними процессами модели

| Критерий | GPT Модель | Assistant API |
|----------|------------|---------------|
| **Основное использование** | Обширные задачи обработки языка, такие как генерация текста, перевод, подведение итогов. | Оптимизирован для создания естественного и последовательного диалога. |
| **Гибкость и настройка** | Высокая гибкость в настройке запросов и управлении выходными данными. | Более стандартизированные ответы, оптимизированные для поддержания диалога. |
| **Управление контекстом** | Не специализирован на поддержании контекста в диалоге. | Эффективно управляет контекстом в рамках многоходовых диалогов. |
| **Сложность интеграции** | Может потребовать более сложной интеграции для специализированных задач. | Легче интегрировать для приложений, требующих естественного диалога. |
| **Доменная специализация** | Подходит для широкого спектра задач, включая технически сложные запросы. | Лучше всего подходит для приложений, где требуется естественное ведение диалога. |

> [!NOTE] Ассистенты находятся в стадии beta, так что в будущем особенности их применения могут быть скорректированы

## Как использовать ассистента

Покажем на примере SDK, как работает Ассистент. Для этого его нужно создать, потом создать его экземпляр (называемый Thread или Диалог), потом можно добавить сообщения и после этого мы запускаем выполнение (Run).
### 1. Создаём ассистента

```js
const assistant = await openai.beta.assistants.create({
  name: "Math Tutor",
  instructions: "You are a personal math tutor. Write and run code to answer math questions.",
  tools: [{ type: "code_interpreter" }],
  model: "gpt-4-1106-preview"
});
```

Мы используем метод SDK .create(), передавая ему объект конфигурации. Вот самые важные свойства конфигурации:
- model — идентификатор модели (обязательно)
- instructions — промпт, настраивающий ассистента (максимум 32768 символов)
- tools — массив с указанием [инструментов](#Tools), до 128 штук
- file_ids — массив с указанием файлов, до 20 штук

В ответ вернётся объект Ассистента из которого самым важным свойством является уникальный id ассистента.

### 2. Создаём диалог

Thread — это конкретный диалог вместе с контекстом, поэтому как правило на каждого пользователя создаётся свой тред (пользовательская сессия).

```js
const thread = await openai.beta.threads.create();
```

> [!tip] Треды не имеют ограничения по размеру — при приближении к лимиту Ассистент оптимизирует историю диалога чтобы уменьшить её.

В ответ вернётся объект диалога из которого самым важным свойством является уникальный id.
### 3. Добавляем сообщения

Сообщение можешь быть текстом или файлом. И оно должно содержать id диалога, чтобы попасть в нужную беседу.

Если необходимо, тут же можно передать индивидуальные данные о пользователе в сообщении, чтобы сделать беседу более персонализированной.

```js
const message = await openai.beta.threads.messages.create(
  thread.id,
  {
    role: "user",
    content: "I need to solve the equation `3x + 11 = 14`. Can you help me?"
  }
);
```

### 4. Запускаем Ассистента

Выполнение диалога называется Run. Оно запускается через соответствующий метод:

```js
const run = await openai.beta.threads.runs.create(
  thread.id,
  { 
    assistant_id: assistant.id,
    instructions: "Please address the user as Jane Doe. The user has a premium account."
  }
);
```

> [!IMPORTANT] При указании instructions в запуске Run, новое значение перезаписывает инструкции, которые были заданы при создании Ассистента. Таким образом можно изменять инструкции для конкретного пользователя

### 5. Проверяем статус запуска

После запуска Run уходит в состояние `queued`. Нам необходимо время от времени запрашивать статус чтобы узнать, когда запуск будет выполнен и статус сменится на `completed`.

```js
const run = await openai.beta.threads.runs.retrieve(
  thread.id,
  run.id
);
```

### 6. Получаем ответ

Когда Run получает статус `completed`, мы можем запросить массив сообщений чтобы отобразить их пользователю:

```js
const messages = await openai.beta.threads.messages.list(
  thread.id
);
```

В ответ вернётся объект message с массивом data, который содержит историю переписки.

## Tools

TODO: Заполнить из https://platform.openai.com/docs/assistants/tools
# Подробнее о возможностях

Разберём, что можно делать с помощью моделей OpenAI
## Генерация текста

Генерация текста — общее название, которое позволяет:

- Создавать черновики документов
- Писать компьютерный код
- Отвечать на вопросы по известной теме
- Проводить анализ текста
- Давать пользователям ПО возможность общаться обычным языком
- Тренировать знание по некоторым темам
- Переводить языки
- Имитировать персонажа игр
и многое другое
## Completion

API, связанный с генерацией текста, называется Completion API. Он поддерживает как сложные разговоры, так и простые задачи без какого-либо разговора.

Completions API может принимать множество параметров, которыми можно настраивать:
- Выбор модели — указывается в строке `model`, обязателен
- Контекст беседы — содержится в массиве `messages`, обязателен. В каждом объекте у сообщения есть одна из трёх ролей:
	1. `system` для задания поведения ассистента
	2. `assistant` с ответами модели
	3. `user` c сообщениями пользователя

> [!tip] В отличие от Assistant API, Completion и другие API не хранят у себя историю беседы и её надо хранить самостоятельно, передавая в каждом запросе

Необязательные параметры:
- Оригинальность и креативность ответа — с помощью числовых значений в параметрах `frequency_penalty`, `presence_penalty`, `temperature`, `top_p`,  и прочих
- Длину ответа — указывается в токенах через свойство `max_tokens`
- Формат ответа — в объекте `response_format`
- Стоп-сигналы — для прекращения генерации, что помогает придерживаться заданной темы (свойство `stop`)
- Постепенное поступление ответа — чтобы имитировать живого собеседника и одновременно сокращать время ожидания ответа, с помощью свойства `stream`
- Инструменты — в массиве `tools` и строке `tool_choice`

Пример:

```js
import OpenAI from "openai";

const openai = new OpenAI();

async function main() {
  const completion = await openai.chat.completions.create({
    messages: [{"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Who won the world series in 2020?"},
        {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
        {"role": "user", "content": "Where was it played?"}],
    model: "gpt-3.5-turbo",
  });
}
main();
```

Результатом запроса будет объект:
```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "The 2020 World Series was played in Texas at Globe Life Field in Arlington.",
        "role": "assistant"
      }
    }
  ],
  "created": 1677664795,
  "id": "chatcmpl-7QyqpwdfhqwajicIEznoc6Q47XAyW",
  "model": "gpt-3.5-turbo-0613",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 17,
    "prompt_tokens": 57,
    "total_tokens": 74
  }
}
```

Как видно, ответ модели содержится в свойстве `completion.choices[0].message.content`.

Тут есть и другая полезная информация. В частности, `finish_reason` содерижт
# Внешние ресурсы
Документация — https://platform.openai.com/docs/introduction/overview
Курс по Prompt Engineering — https://learn.deeplearning.ai/chatgpt-prompt-eng