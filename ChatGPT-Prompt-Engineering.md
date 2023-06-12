>  A lot of that has been focused on the chatGPT web user interface, which many people are using to do specific and often one-off tasks. But, I think the power of LLMs, large language models, as a developer tool, that is using API calls to LLMs to quickly build software applications, I think that is still very underappreciated.
>  *— Andrew NG (автор из DeepLearning.AI)*

# LLM — Большие Языковые Модели

Два типа Больших Языковых Моделей (LLM):
1. Base LLM — Предсказывает продолжение на основе данных, которыми модель была обучена
2. Instruction Tuned LLM — Следует инструкциям и уточняет их, используя RLHF (Reinforcement Learning with Human Feedback)

Например, за запрос *"What is the capital of France?"* Base LLM может выдать другие похожие вопросы (так как данные обучения включали серию вопросов на тему туризма), а IT LLM может дать ответ *"The capital of France is Paris"*. Другими словами, IT LLM позволяет ограничивать тон и формат ответа модели.

# Основные принципы

1. Использовать понятные и специфичные инструкции
2. Дать модели время подумать

## 1. Понятные и специфичные инструкции

Понятные ≠ лаконичные. Длинные инструкции будут понятнее для модели.

Несколько тактик чтобы создать такие инструкции:

### A. Использовать разделители, например:
```js
const text = '(5 + 6() / 2 ^ 3';
const prompt = `Calculate the expression delimited by angle brackets and start response with the phrase "The answer is". <${text}>`;

getCompletion(prompt);
```

> [!WARNING] Разделители помогают предотвратить инъекцию в запрос (prompt injection) — ситуация, в которой запрос пользователя внутри инструкции может перенастроить модель. Например, запрос пользователя, содержащий фразу *forget the previous instructions* не будет обработан, так как модель знает, что контент внутри разделителей — не инструкция

| Примеры разделителей| |
|-----|---|
| Triple quotes | """|
|Triple backtics|\`\`\`|
| Triple dashes | ---|
|Angle brackets | <>|
| и другие||

### B. Форматировать ответ (например, JSON)

```js
const prompt = 'Generate a list of three made-up book titles along with their authors and genres. Provide them in JSON format with the following keys: book_id, title, author, genre.'
```

### C. Проверить, что условия соблюдены
Требует сделать предположения (assumptions) о том, что требуется для достижения цели.

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

Если описать критерии результата трудно (например, для формирования тона ответа), можно использовать примеры:

```js
const prompt = `Your task is to answer in a consistent style.
<child>: Teach me about patience.
<grandparent>: The river that carves the deepest valley flows from a modest spring; the grandest symphony originates from a single note; the most intricate tapestry begins with a solitary thread.
<child>: Teach me about resilience.`;
```

## 2. Дать модели время на размышления

### A. Задать алгоритм для выполнения задачи

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

### B. Попросить найти решение самостоятельно

В некоторых случаях, когда мы в запросе просим оценить какой-то текст или решение, модель может прийти в неверным выводам, проигнорировав ошибки. В таком случае можно попросить модель решить самой, а потом сравнить с предоставленным решением:

```js
const prompt = `Your task is determine if the student's solution is correct or not. To solve the problem, do the following:
1. Work out your own solution to the problem.
2. Then compare your solution to the student's solution and evaluate if the student's solution is correct or not. Don't decide if the student's solution is correct until you have done the problem yourself.

Question: ${question}

Student's solution: ${student_solution}`
```



# Галлюцинации

Галлюцинациями называют убедительные неправдоподобные ответы модели (например, выдуманные цитаты)

Чтобы избежать их, можно попросить модель найти информацию по теме, потом ответить, основываясь на этой информации

# Внешние ссылки

Сайт курса — https://learn.deeplearning.ai/chatgpt-prompt-eng
