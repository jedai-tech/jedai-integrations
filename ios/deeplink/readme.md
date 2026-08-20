# Взаимодействие через iOS deeplink

Через URL-схему `intelligenceretail` можно вызвать приложение JEDAI из своего приложения и не подключать библиотеку JEDAI.

**Условие:** на устройстве установлено приложение JEDAI.

Чтобы [подключить библиотеку JEDAI](../library/readme.md), используйте отдельную инструкцию.

- [Что нужно для работы](#что-нужно-для-работы)
- [Как вызвать метод](#как-вызвать-метод)
  - [Методы](#методы)
  - [Параметры вызова](#параметры-вызова)
  - [Пример вызова метода](#пример-вызова-метода)
  - [Как task_id меняет экраны](#как-task_id-меняет-экраны)
- [Как получить ответ](#как-получить-ответ)
  - [Формат данных ответа](#формат-данных-ответа)
  - [Статусы](#статусы)
  - [Какие отчеты приходят в ответе](#какие-отчеты-приходят-в-ответе)
  - [Как обработать ответ в SceneDelegate](#как-обработать-ответ-в-scenedelegate)
  - [Как обработать ответ в AppDelegate](#как-обработать-ответ-в-appdelegate)
  - [Как обработать ответ в SwiftUI](#как-обработать-ответ-в-swiftui)
- [Как запустить синхронизацию](#как-запустить-синхронизацию)
- [Пример сценария](#пример-сценария)
- [Примеры отчета](#примеры-отчета)

## Что нужно для работы

- На устройстве установлено приложение JEDAI.
- В вашем приложении зарегистрирована URL-схема. JEDAI вернет результат на нее через параметр `back_url_scheme`.

Чтобы [зарегистрировать URL-схему](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app?language=swift), добавьте ее в `Info.plist`:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>yourappscheme</string>
        </array>
    </dict>
</array>
```

Чтобы проверять установку JEDAI через `canOpenURL`, добавьте схему `intelligenceretail` в `LSApplicationQueriesSchemes`:

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>intelligenceretail</string>
</array>
```

## Как вызвать метод

Чтобы вызвать метод приложения JEDAI:

1. Соберите URL со схемой `intelligenceretail` и нужными параметрами.
2. Откройте URL через `UIApplication.shared.open`.

Формат URL: `intelligenceretail:?param1=value1&param2=value2`.

### Методы

| Метод | Что делает | Как возвращает результат |
| --- | --- | --- |
| `visit` | Создает или редактирует визит и открывает съемку. Кнопка «Назад» недоступна, пока нет отчетов по всем фото. Если нет интернета и нет неподтвержденных фото, JEDAI вернет `IR_ERROR_NO_INET` | Возврат в ваше приложение по `back_url_scheme` |
| `report` | Возвращает отчет по визиту | JSON в query-параметре `result` |
| `summaryReport` | Открывает сводный отчет | Экран в JEDAI |
| `showVisitReport` | С `task_id` открывает отчет по задаче. Без `task_id` открывает карточку торговой точки со списком задач или сводный отчет, если задач нет | Экран в JEDAI |
| `sync` | Запускает фоновую отправку фото и получение отчетов | `IR_RESULT_OK`, если в очереди были данные. `IR_RESULT_EMPTY`, если отправлять нечего |
| `syncCatalogs` | Авторизует пользователя и загружает справочники. Повторный вызов подтягивает обновления | `IR_RESULT_OK` |

### Параметры вызова

| Параметр | Обязательный | Методы | Описание |
| --- | --- | --- | --- |
| `method` | Да | Все | Имя метода: `visit`, `report`, `summaryReport`, `showVisitReport`, `sync`, `syncCatalogs` |
| `login` | Да | Все | Логин пользователя |
| `password` | Да | Все | Пароль пользователя |
| `user_id` | Да, если вход идет через внешний идентификатор | Все | Внешний идентификатор пользователя |
| `store_id` | Да | `visit` | Идентификатор торговой точки |
| `visit_id` | Да | `visit`, `report`, `summaryReport`, `showVisitReport` | Идентификатор визита |
| `task_id` | Нет | `visit`, `report`, `summaryReport`, `showVisitReport` | Идентификатор задачи |
| `back_url_scheme` | Да для `report` и если нужен возврат в ваше приложение | `report`, при возврате — остальные | URL-схема вашего приложения |

### Пример вызова метода

```swift
var components = URLComponents()
components.scheme = "intelligenceretail"
components.queryItems = [
    URLQueryItem(name: "method", value: methodName),
    URLQueryItem(name: "login", value: login),
    URLQueryItem(name: "password", value: password),
    URLQueryItem(name: "user_id", value: userId),
    URLQueryItem(name: "store_id", value: storeId),
    URLQueryItem(name: "visit_id", value: visitId),
    URLQueryItem(name: "task_id", value: taskId),
    URLQueryItem(name: "back_url_scheme", value: "yourappscheme")
]
guard let url = components.url else { return }
UIApplication.shared.open(url, options: [:]) { completed in
    // факт открытия URL
}
```

### Как task_id меняет экраны

Таблица описывает методы `visit`, `report` и `summaryReport`. Поведение `showVisitReport` — в таблице [Методы](#методы).

| Содержимое `task_id` | Задачи на портале JEDAI | Поведение |
| --- | --- | --- |
| Идентификатор задачи, которой нет на портале | Не важно | **visit** — съемка визита с учетом указанной задачи. **report**, **summaryReport** — отчет по всему визиту |
| Идентификатор задачи с портала | Есть | **visit** — карточка указанной задачи. **report**, **summaryReport** — отчет в разрезе задачи |
| Идентификатор задачи с портала | Нет | **visit** — съемка визита с учетом указанной задачи. **report**, **summaryReport** — отчет по всему визиту |
| Нет | Есть | **visit** — карточка торговой точки со списком задач. **report**, **summaryReport** — отчет по всему визиту |
| Нет | Нет | **visit** — съемка визита. **report**, **summaryReport** — отчет по всему визиту |

## Как получить ответ

Чтобы вернуть результат, JEDAI открывает URL вашей схемы:

```text
yourappscheme://?result={json}
```

Схему передайте в `back_url_scheme`. JSON в query-параметре приходит в percent-encoding. `URLComponents` декодирует его сам.

Query-параметр возврата — `result`. Поле `report` лежит внутри JSON, а не в имени параметра URL.

### Формат данных ответа

Изучите [примеры отчета](#примеры-отчета).

| Ключ | Тип | Обязательный | Описание |
| --- | --- | --- | --- |
| `status` | `String` | Да | Статус выполнения метода |
| `photosCounter` | `Int` | Нет | Количество фото в визите. Если передан `task_id` — в задаче |
| `scenesCounter` | `Int` | Нет | Количество сцен в визите. Если передан `task_id` — в задаче |
| `notDetectedScenesCounter` | `Int` | Нет | Количество сцен, где есть хотя бы одно нераспознанное фото |
| `notDetectedPhotosCounter` | `Int` | Нет | Количество фото без отчета, включая неотправленные. Если передан `task_id` — в задаче |
| `report` | JSON | Нет | Отчет по визиту. Если передан `task_id` — по задаче |

### Статусы

| Статус | Код | Описание |
| --- | --- | --- |
| `IR_RESULT_OK` | 1 | Метод выполнен успешно. Для `sync`: в очереди были данные, синхронизация запустилась |
| `IR_RESULT_EMPTY` | 2 | Нет данных. Для `sync`: отправлять нечего — фото уже ушли, отчеты получены |
| `IR_ERROR` | 5 | Неизвестная ошибка |
| `IR_ERROR_NO_INET` | 6 | Нет интернета |
| `IR_ERROR_TOKEN` | 7 | Ошибка токена |
| `IR_ERROR_STORE_ID_INCORRECT` | 10 | Некорректный идентификатор торговой точки |
| `IR_ERROR_VISIT_ID_INCORRECT` | 12 | Некорректный идентификатор визита |
| `IR_ERROR_AUTH` | 13 | Ошибка авторизации |
| `IR_RESULT_INPROGRESS` | 16 | Данные еще обрабатываются |
| `IR_ERROR_NOVISIT` | 17 | Визит с указанным идентификатором не найден |

### Какие отчеты приходят в ответе

Состав поля `report` зависит от фото, ответов на вопросы и того, обязательна ли съемка.

Фото обязательны, если в визите нет задач или среди обязательных задач есть съемка:

| Данные в визите | Статус | Отчеты |
| --- | --- | --- |
| Есть фото, обработаны не все, есть ответы на вопросы | `IR_RESULT_INPROGRESS` (16) | `visit_stats`, `photos`, `share_shelf`, `share_shelf_by_metrics`, `custom`, `assortment_achievement`, `perfect_store` |
| Есть фото, обработаны не все, нет ответов на вопросы | `IR_RESULT_INPROGRESS` (16) | `visit_stats`, `photos`, `share_shelf`, `share_shelf_by_metrics`, `custom`, `assortment_achievement` |
| Нет фото, есть ответы на вопросы | `IR_RESULT_EMPTY` (2) | `visit_stats`, `perfect_store` |
| Нет фото, нет ответов на вопросы | `IR_RESULT_EMPTY` (2) | `visit_stats` |
| Есть фото, все отправлены, есть ответы на вопросы | `IR_RESULT_OK` (1) | `visit_stats`, `assortment_achievement`, `share_shelf`, `share_shelf_by_metrics`, `custom`, `photos`, `perfect_store` |
| Есть фото, все отправлены, нет ответов на вопросы | `IR_RESULT_OK` (1) | `visit_stats`, `assortment_achievement`, `share_shelf`, `share_shelf_by_metrics`, `custom`, `photos`, `perfect_store` |

Если съемка не обязательна (в обязательных задачах нет фотографирования), строки совпадают, кроме одной: нет фото, но есть ответы на вопросы → статус `IR_RESULT_OK` (1), а не `IR_RESULT_EMPTY` (2). Набор отчетов тот же: `visit_stats`, `perfect_store`.

### Как обработать ответ в SceneDelegate

Чтобы обработать ответ в приложении со сценами (iOS 13 и новее), реализуйте метод в `SceneDelegate`:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard
        let url = URLContexts.first?.url,
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
        let result = components.queryItems?.first(where: { $0.name == "result" })?.value
    else { return }
    // разберите JSON из result
}
```

### Как обработать ответ в AppDelegate

Чтобы обработать ответ в приложении без сцен, реализуйте метод в `AppDelegate`:

```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey: Any] = [:]) -> Bool {
    guard
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
        let result = components.queryItems?.first(where: { $0.name == "result" })?.value
    else { return false }
    // разберите JSON из result
    return true
}
```

### Как обработать ответ в SwiftUI

Чтобы обработать ответ в SwiftUI, используйте `.onOpenURL`:

```swift
.onOpenURL { url in
    guard
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
        let result = components.queryItems?.first(where: { $0.name == "result" })?.value
    else { return }
    // разберите JSON из result
}
```

## Как запустить синхронизацию

Фоновые задачи в iOS живут недолго. Если визит шел офлайн, синхронизация может не успеть закончиться сама.

Чтобы принудительно отправить фото и получить отчеты:

1. Вызовите метод `sync`.
2. Если статус `IR_RESULT_OK`, синхронизация запустилась: в очереди были данные.
3. Если статус `IR_RESULT_EMPTY`, отправлять нечего: фото уже ушли, отчеты получены.

## Пример сценария

Чтобы получить отчет по визиту:

1. Зарегистрируйте URL-схему своего приложения в `Info.plist`.
2. Откройте JEDAI методом `visit`.
3. Сделайте фото и выйдите из JEDAI.
4. Прочитайте `status` в параметре `result`.
5. Если статус `IR_RESULT_INPROGRESS`, вызовите `sync` и дождитесь повторного возврата.
6. Если статус `IR_RESULT_OK`, разберите JSON отчета.
7. Чтобы открыть отчет или сводный отчет, вызовите `report` или `summaryReport`.

## Примеры отчета

Изучите [пример отчета без task_id](without_task_id_response.json) и [пример отчета с task_id](with_task_id_response.json).
