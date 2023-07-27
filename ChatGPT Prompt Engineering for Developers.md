Авторы курса:
- Isa Fulford (OpenAI)
- Andrew Ng (DeepLearning.AO)

# LLM — Большие Языковые Модели

> A lot of that has been focused on the chatGPT web user interface, which many people are using to do specific and often one-off tasks. But, I think the power of LLMs, large language models, as a developer tool, that is using API calls to LLMs to quickly build software applications, I think that is still very underappreciated.
>  *— Andrew NG*

Два типа Больших Языковых Моделей (LLM):
1. Base LLM — Предсказывает продолжение на основе данных, которыми модель была обучена
2. Instruction Tuned LLM — Следует инструкциям и уточняет их, используя RLHF (Reinforcement Learning with Human Feedback)

Например, на запрос *"What is the capital of France?"* Base LLM может выдать другие похожие вопросы (так как данные обучения включали серию вопросов на тему туризма), а ITLLM может дать ответ *"The capital of France is Paris"*. Другими словами, ITLLM позволяет ограничивать тон и формат ответа модели.

# Основные принципы промптинга

1. Использовать понятные и специфичные инструкции
2. Дать модели время подумать

## 1. Понятные и специфичные инструкции

Понятные ≠ лаконичные. Длинные инструкции будут понятнее для модели.

Несколько тактик чтобы создать такие инструкции:

### A. Использовать разделители

| Разделители бывают:| |
|-----|---|
| Triple quotes | """|
|Triple backtics|\`\`\`|
| Triple dashes | ---|
|Angle brackets | <>|
|| и другие|

Мы указываем, какие разделители используем, затем применяем их:
```js
const text = '(5 + 6() / 2 ^ 3';
const prompt = `Calculate the expression delimited by angle brackets and start response with the phrase "The answer is". <${text}>`;

getCompletion(prompt);
```

> [!WARNING] Разделители помогают предотвратить инъекцию в запрос (prompt injection) — ситуация, в которой запрос пользователя внутри инструкции может перенастроить модель. Например, запрос пользователя, содержащий фразу *forget the previous instructions* не будет обработан, так как модель знает, что контент внутри разделителей — не инструкция

### B. Попросить отформатировать ответ

Как и с разделителями, достаточно дать модели название популярного формата и она сама даст соответствующий ответ. Например, вот тут мы просим модель использовать JSON:
```js
const prompt = 'Generate a list of three made-up book titles along with their authors and genres. Provide them in JSON format with the following keys: book_id, title, author, genre.'
```

### C. Попросить проверить условия
Мы даём модели предположение (assumption) о том, что за данные будут на входе и как ей их обработать:

```js
const prompt = `You will be provided with text delimeted by triple quotes. If it contains a sequense of instructions, re-write those instructions in following format:
Step 1 — ...
Step 2 — ...
...
Step N — ...

If the text does not containn a sequence of instructions, then simply write "No steps provided."
"""${text}"""`;
```

Наличие нескольких "if" делает эту тактику похожую на использование условий в языках программирования (if — else).

### D. Дать примеры успешного выполнения

Если описать критерии результата трудно (например, для формирования тона ответа), можно использовать примеры. Эта тактика также называется "Few-shot":

```js
const prompt = `Your task is to answer in a consistent style.
<child>: Teach me about patience.
<grandparent>: The river that carves the deepest valley flows from a modest spring; the grandest symphony originates from a single note; the most intricate tapestry begins with a solitary thread.
<child>: Teach me about resilience.`;
```

## 2. Дать модели время "подумать"

### A. Задать алгоритм для выполнения задачи

Как и человек, модель склонна совершать меньше ошибок, если разбить сложную задачу на меньшие. Это можно сделать в виде конкретных шагов: 

```js
const prompt = `Your task is to perform the following actions:
1 — Summarize the following text delimited by <> with 1 sentence.
2 — Translate the summary into French.
3 — List each name in the French summary.
4 — Output a json object that contains the following keys: french_summary, num_names

Use the following format:
Text: <text to summarize>
Summary: <summary>
Translation: <summary translation>
Names: <list of names in summary>
Output JSON: <json with summary and num_names>

Text: <${text}>`
```

### B. Попросить решить самостоятельно перед оценкой

В некоторых случаях, когда мы в запросе просим оценить какой-то текст или решение, модель может прийти в неверным выводам, проигнорировав ошибки. В таком случае можно попросить модель решить самой, а потом сравнить с предоставленным решением:

```js
const prompt = `Your task is determine if the student's solution is correct or not. To solve the problem, do the following:
1. Work out your own solution to the problem.
2. Then compare your solution to the student's solution and evaluate if the student's solution is correct or not. Don't decide if the student's solution is correct until you have done the problem yourself.

Question: ${question}

Student's solution: ${student_solution}`
```

>[!WARNING] GPT-3 isn't really affected whether you insert newline characters or not. But when working with LLMs in general, you may consider whether newline characters in your prompt may affect the model's performance.

# Итеративная разработка

Как и в написании кода, в разработке запросов почти никогда не получается добиться нужного результата без ошибок с первого раза. Поэтому разработка промпта обычно происходит итерациями:

1. Сформулировать идею
2. Написать и отправить соответствующий промпт
3. Проанализировать, почему ответ LLM отличается от ожидаемых
4. Доработать идею и промпт
5. Повторить

> This too is why I personally have not paid as much attention to the internet articles that say "30 perfect prompts", because I think, there probably isn't a perfect prompt for everything under the sun. **It's more important that you have a process for developing a good prompt for your specific application**.
> *— Andrew NG*

## Пример

### Формулируем идею
Нам нужен промпт, который будет генерировать описание мебели из её технических характеристик

### Пишем промпт
```js
const prompt = `Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Technical specifications: ${fact_sheet_chair}`
```

### Вносим правки 1
Результат получился длинный, нам нужно его ограничить
```js
const prompt = `Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Use at most 50 words.

Technical specifications: ${fact_sheet_chair}`
```

С ограничением длины тексты есть два важных нюанса:
- Ограничение можно указывать в любых удобных единицах — словах, символах, предложениях
- GPT не всегда соблюдает чётко требования по длине текста. Результат выше может получиться 50+ слов

### Вносим правки 2
Результат уже намного лучше, но он слишком образный. Сообщим модели, что мы хотим сосредоточиться на технических достоинствах
```js
const prompt = `Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

Use at most 50 words.

Technical specifications: ${fact_sheet_chair}`
```

### Указываем необходимый формат
Теперь попросим форматировать результат в HTML для использования на сайте
```js
const prompt = `Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from. Use at most 50 words.

At the end of the description, include every 7-character 
Product ID in the technical specification.

After the description, include a table that gives the 
product's dimensions. The table should have two columns.
In the first column include the name of the dimension. 
In the second column include the measurements in inches only.

Give the table the title 'Product Dimensions'.

Format everything as HTML that can be used in a website. 
Place the description in a <div> element.

Technical specifications: ${fact_sheet_chair}`
```

# Обработка текста

Посмотрим, как LLM справляется с обработкой больших объёмов текста

## Резюмирование (Суммирование)

Хотя можно напрямую попросить LLM суммировать текст, специфичный запрос с указанием рода текста, требований и назначения будет лучше. Например, тут мы :
- указываем, что работаем с отзывом на продукт из интернет-магазина
- требуем ограничить результат 30 словами
- определяем, что результат будет передан службе доставки
```js
const prompt = `Your task is to extract relevant information from a product review from an ecommerce site to give feedback to the Shipping department. 

From the review below, delimited by triple quotes extract the information relevant to shipping and delivery. Limit to 30 words.

Review: ___${prod_review}___`
```

## Выделение

В некоторых случаях можно попросить LLM вернуть нам прямую цитату, связанную с нужной темой, вместо резюмирования. Для этого достаточно заменить 'summarize' на 'extract'.

inferring
sentiment
NLP

# Галлюцинации

Галлюцинациями называют убедительные неправдоподобные ответы модели (например, выдуманные цитаты)

Чтобы избежать их, можно попросить модель найти информацию по теме, потом ответить, основываясь на этой информации

# Внешние ссылки

Сайт курса — https://learn.deeplearning.ai/chatgpt-prompt-eng