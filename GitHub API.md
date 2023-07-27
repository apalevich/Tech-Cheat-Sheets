GitHub API соответствует архитектуре REST.

# Основы взаимодействия

Базовый URL находится по адресу https://api.github.com

Запрос содержит параметры:
- метод (обязательно)
- путь запроса (обязательно)
- заголовки
- запрос
- тело

Ответ содержит:
- код статуса (обязательно)
- заголовки (обязательно)
- тело

> [!INFO ] Для взаимодействия с GitHub API можно использовать библиотеку [Octokit.js](https://github.com/octokit/octokit.js/#readme), которая рекомендована инструкцией. Однако, я постараюсь описать всё без её использования, обычными запросами через инферфейсы fetch() или axios.

## Аутентификация

Большинство операций требуют аутентификации для выполнения или предоставления дополнительной информации.

Для аутентификации используется токен, который бывает двух видов:

### Fine-grained personal access tokens

Более современная версия, которая позволяет:
- ограничить доступ ресурсами только одного пользователя/организации
- ограничить доступ только одним репозиторием
- Расширенные возможности управления правами доступа
- Каждый токен имеет срок действия (6/12 месяцев)

### Classic personal access tokens

Таким образом, классические токены менее безопасны. Однако, они нужны для некоторых операций благодаря своим правам доступа:

- на запись данных в публичных репозиториях, которые не принадлежат владельцу токены (пользователю, компании или сотруднику)
- на доступ к репозиториям организаций пользователям-коллабораторам (?)
- прочие операции, не указанные в [этом списке](https://docs.github.com/en/rest/overview/endpoints-available-for-fine-grained-personal-access-tokens?apiVersion=2022-11-28)

[Подробнее о токенах](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)

### Примеры кода с аутентификацией

Запрос из командной строки:
```shell
curl --request GET \
--url "https://api.github.com/octocat" \
--header "Authorization: Bearer YOUR-TOKEN"
```

Внутри GitHub Actions:
```yaml
jobs:
  use_api:
    runs-on: ubuntu-latest
    permissions: {}
    steps:
      - env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          curl --request GET \
          --url "https://api.github.com/octocat" \
          --header "Authorization: Bearer $GH_TOKEN"
```

## Заголовки

Большинство операций требуют указания заголовков в запросе:

```shell
curl --request GET \
--url "https://api.github.com/octocat" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer YOUR-TOKEN" \
--header "X-GitHub-Api-Version: 2022-11-28"
```

# Указание параметров

## Параметры в пути

Изменение пути запроса определяет, что вернётся в ответ:

Информация о репозитории —  `/repos/octocat/Spoon-Knife/issues`
Список файлов в основной ветке — `/repos/octocat/Spoon-Knife/contents`
Список ишью — `/repos/octocat/Spoon-Knife/issues`

## Параметры в запросе

Указание параметров в пути определяет дополнительные параметры ответа.
Например, в результате этого запроса мы получим 2 ишью, которые были обновлены последними в нисходящем порядке:

```shell
curl --request GET \
--url "https://api.github.com/repos/octocat/Spoon-Knife/issues?per_page=2&sort=updated&direction=asc" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer YOUR-TOKEN"
```

## Параметры в теле

Параметры в теле позволяют передавать информацию в API.
Например, для создания новых сущностей через POST:

```shell
curl --request POST \
--url "https://api.github.com/repos/octocat/Spoon-Knife/issues" \
--header "Accept: application/vnd.github+json" \
--header "Authorization: Bearer YOUR-TOKEN" \
--data '{
  "title": "Created with the REST API",
  "body": "This is a test issue created by the REST API"
}'
```

Ответ на этот запрос будет содержать свойство `html_url` с адресом созданного ишью и другие свойства.

