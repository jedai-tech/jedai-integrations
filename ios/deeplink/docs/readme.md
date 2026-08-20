# Взаимодействие с приложением Ailet с помощью deeplinks.

- [Взаимодействие с приложением Ailet с помощью deeplinks.](#взаимодействие-с-приложением-ailet-с-помощью-deeplinks)
  - [Вызов метода](#вызов-метода)
    - [Создание URL](#создание-url)
    - [Параметры вызова:](#параметры-вызова)
    - [Описание методов](#описание-методов)
    - [Поведение методов в зависимости от параметра task_id](#поведение-методов-в-зависимости-от-параметра-task_id)
  - [Результат выполнения метода](#результат-выполнения-метода)
    - [Статус выполенения метода](#статус-выполенения-метода)
    - [Статус и доступные отчёты в зависимости от наличия задач в визите, фото и обязательности их исполнения](#статус-и-доступные-отчёты-в-зависимости-от-наличия-задач-в-визите-фото-и-обязательности-их-исполнения)
  - [Примеры использования](#примеры-использования)
    - [Использование метода sync (например при съемке в оффлайн)](#использование-метода-sync-например-при-съемке-в-оффлайн)
    - [Для iOS13 и SwiftUI:](#для-ios13-и-swiftui)
    - [Для iOS ниже 13-й версии или без SwiftUI:](#для-ios-ниже-13-й-версии-или-без-swiftui)
  - [Примеры отчета](#примеры-отчета)
    - [Без task_id](without_task_id_response.json)
    - [С task_id](with_task_id_response.json)

## Вызов метода
### Создание URL
Сгенерируйте URL вида `intelligenceretail:?param1=value1&param2=value2` любым удобным для вас способом, например через `URLComponents` и откройте его через UIApplication.shared.open:
```swift
var components = URLComponents()
components.scheme = "intelligenceretail"
components.queryItems = [URLQueryItem(name: "method", value: methodName),
                         URLQueryItem(name: "login", value: login),
                         URLQueryItem(name: "password", value: password),
                         URLQueryItem(name: "user_id", value: userId),
                         URLQueryItem(name: "store_id", value: storeId),
                         URLQueryItem(name: "visit_id", value: visitId),
                         URLQueryItem(name: "task_id", value: taskId),
                         URLQueryItem(name: "back_url_scheme", value: "integrationtestapp")]
let url = components.url!
UIApplication.shared.open(url, options: [:]) { (completed) in
    // Handle completion if needed
}
```

### Параметры вызова:

| Параметр | Обязательно | Описание |
| --- | --- | --- |
| method | Да | Наименование метода (visit, report, summaryReport, showVisitReport, sync, syncCatalogs) |
| login | Да | Логин пользователя |
| password | Да | Пароль пользователя |
| user_id | Да для пользователей, использующих внешний id | внешний идентификатор пользователя |
| store_id | Да для метода visit | идентификатор торговой точки |
| visit_id | Да для методов visit, report, summaryReport, showVisitReport | идентификатор визита |
| task_id | Нет  | идентификатор задачи, используется в методах visit, report, summaryReport, showVisitReport |
| back_url_scheme | Да для метода report и если необходим возврат в вызывающее приложение | значение кастомной URL схемы вашего приложения |

### Описание методов 

| Метод | Описание | Параметры |
| --- | --- | --- |
| visit | Создание/редактирование визита. Открывает экран в режиме съёмки. Пока пользователь не получит отчеты по всем фото из визита, кнопка Назад не будет работать. При отсутствии соединения с интернетом и отсутствия у пользователя неподтвержденных фото, приложение откроет стороннее приложение со статусом IR_ERROR_NO_INET  | method, login, password, user_id, visit_id, store_id, task_id |
| report | Отчет по визиту. Открывает ваше приложение через URL с отчетом в виде JSON-строки в параметре "report". | method, login, password, user_id, task_id, visit_id, back_url_scheme |
| summaryReport | Открытие экрана со сводным отчётом. | method, login, password, user_id, visit_id, task_id |
| showVisitReport | Открытие экрана отчёта по визиту: при `task_id` открывает отчёт по задаче, без `task_id` открывает экран тоговой точки со списком задач или сводный отчёт, если задач в торговой точке нет. | method, login, password, user_id, visit_id, task_id |
| sync | Запуск фонового процесса передачи фото и получения результатов. В случае наличия данных для отправления и успешного запуска процесса синхронизации метод возвращает статус IR_RESULT_OK. В случае отсутствия данных для синхронизация (все фото отправлены, отчёты для фото и визитов получены) метод вернёт статус IR_RESULT_EMPTY. | method, login, password, user_id |
| syncCatalogs | Запускает авторизацию и загрузку необходимых для работы справочников. Возвращает в приложение-клиент статус IR_RESULT_OK в случае успешного завершения. При повторном вызове метода загружает обновления по справочникам.  | method, login, password, user_id |

### Поведение методов в зависимости от параметра task_id

| Содержимое task_id | Наличие задач | Поведение в зависимости от метода|
| --- | :-: | --- |
| ID задачи, отсутствующей на портале Ailet | не имеет значения | **visit** - откроется съемка визита с учетом указанной задачи<br>**report**, **summaryReport** - отчет по всему визиту |
| ID задачи на портале Ailet | есть | **visit** - откроется карточка указанной задачи<br>**report**, **summaryReport** - отчет в разрезе указанной задачи |
| ID задачи на портале Ailet | нет | **visit** - откроется съемка визита с учетом указанной задачи<br> **report**, **summaryReport** - отчет по всему визиту |
| нет | есть | **visit** - откроется карточка торговой точки со списком задач<br>**report**, **summaryReport** - отчет по всему визиту |
| нет | нет | **visit** - откроется съемка визита<br>**report**, **summaryReport** - отчет по всему визиту |

## Результат выполнения метода

Для передачи результатов отчёта приложение Ailet открывает url вида `{ваша_кастомная_url_схема}:?report={значение_в_виде_json}`.  Кастомная url схема передается через параметр `back_url_scheme`, описанный выше. Подробнее про использование url схем можно прочитать [в документации Apple](https://developer.apple.com/documentation/uikit/inter-process_communication/allowing_apps_and_websites_to_link_to_your_content/defining_a_custom_url_scheme_for_your_app?language=swift).

Для обработки результатов отчета используйте метод `scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>)` в `SceneDelegate` для **iOS 13 и выше и использовании SwiftUI** или `application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any])` в `AppDelegate` во всех остальных случаях.
Ответ приходит в ключе "result" виде JSON, в котором есть следующие значения:

| Ключ | Тип | Обязательно | Описание |
| --- | :-: | :-: | --- |
| photosCounter | Int | Нет | Количество фото (в визите, если указан task_id - в задаче) |
| scenesCounter | Int | Нет | Количество сцен (в визите, если указан task_id - в задаче) |
| notDetectedScenesCounter | Int | Нет | количество нераспознанных сцен (сцен, где есть как минимум одна нераспознанная фото) |
| notDetectedPhotosCounter | Int | Нет | фото, по которым не получен отчёт, с учётом не отправленных  (в визите, если указан task_id - в задаче)|
| status | String | Да | Статус IR_RESULT_OK или ошибка для визита|
| report | JSON | Нет | Отчёт по визиту, если указан task_id - по задаче |


### Статус выполенения метода

| Статус | Код | Описание |
|---|:-:|---|
| IR_RESULT_OK | 1 | Успешно* |
| IR_RESULT_EMPTY | 2 | Нет данных** |
| IR_ERROR | 5 | Неизвестная ошибка |
| IR_ERROR_NO_INET | 6 | Отсутствует интернет |
| IR_ERROR_TOKEN | 7 | Ошибка токена |
| IR_ERROR_STORE_ID_INCORRECT | 10 | Некорректный ИД ТТ|
| IR_ERROR_VISIT_ID_INCORRECT | 12 | Некорректный ИД визита |
| IR_ERROR_AUTH | 13 | Ошибка авторизации |
| IR_RESULT_INPROGRESS | 16 | Данные в обработке |
| IR_ERROR_NOVISIT | 17 | Отсутствует визит с указанным ИД |

*В методе sync статус IR_RESULT_OK означает наличие неотправленных данных и успешный запуск синхронизации.

**В методе sync статус IR_RESULT_EMPTY означает отсутствие данных для отправки (все фото отправлены, все отчеты по фото и визитам получены)


### Статус и доступные отчёты в зависимости от наличия задач в визите, фото и обязательности их исполнения

В визите обязательно должны быть фотографии (нет задач / в обязательных задачах есть та, где нужно фотографировать):

| Данные в визите | Статус (код) | Отчеты |
|---|:-:|---|
| Есть фото, не все фото обработаны, есть ответы на вопросы | 16 | visit_stats, photos, share_shelf, share_shelf_by_metrics, custom, assortment_achievement, perfect_store |
| Есть фото, не все фото обработаны, нет ответов на вопросы | 16 | visit_stats, photos, share_shelf, share_shelf_by_metrics, custom, assortment_achievement |
| Нет фото, есть ответы на вопросы | 2 | visit_stats, perfect_store |
| Нет фото, нет ответов на вопросы | 2 | visit_stats |
| Есть фото, все отправлены, есть ответы на вопросы | 1 | visit_stats, assortment_achievement, share_shelf, share_shelf_by_metrics, custom, photos, perfect_store |
| Есть фото, все отправлены, нет ответов на вопросы | 1 | visit_stats, assortment_achievement, share_shelf, share_shelf_by_metrics, custom, photos, perfect_store |

В визите не обязательно должны быть фотографии (в обязательных задачах нет тех, где нужно фотографировать):

| Данные в визите | Статус (код) | Отчеты |
|---|:-:|---|
| Есть фото, не все фото обработаны, есть ответы на вопросы | 16 | visit_stats, photos, share_shelf, share_shelf_by_metrics, custom, assortment_achievement, perfect_store |
| Есть фото, не все фото обработаны, нет ответов на вопросы | 16 | visit_stats, photos, share_shelf, share_shelf_by_metrics, custom, assortment_achievement |
| Нет фото, есть ответы на вопросы | 1 | visit_stats, perfect_store |
| Нет фото, нет ответов на вопросы | 2 | visit_stats |
| Есть фото, все отправлены, есть ответы на вопросы | 1 | visit_stats, assortment_achievement, share_shelf, share_shelf_by_metrics, custom, photos, perfect_store |
| Есть фото, все отправлены, нет ответов на вопросы | 1 | visit_stats, assortment_achievement, share_shelf, share_shelf_by_metrics, custom, photos, perfect_store |

### 

## Примеры использования

### Использование метода sync (например при съемке в оффлайн)

Так как время выполнения фоновых процессов в iOS ограниченно, то может потребоваться принудительно запускать синхронизацию (например если визит выполняется оффлайн), в этом случае можно использовать метод sync. Когда есть данные для отправки, данный метод запустит синхронизацию и вернет статус IR_RESULT_OK, если же синхронизация не требуется, то метод вернет IR_RESULT_EMPTY.

### Для iOS13 и SwiftUI:

```swift
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
    guard 
        let url = URLContexts.first?.url,
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
        let reportQueryItem = components.queryItems?.filter({ $0.name == "result" }).first,
        let report = reportQueryItem.value
        else { return }
    // Do something with report string.
}
```

### Для iOS ниже 13-й версии или без SwiftUI:

```swift 
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    guard 
        let components = URLComponents(url: url, resolvingAgainstBaseURL: false),
        let reportQueryItem = components.queryItems?.filter({ $0.name == "result" }).first,
        let report = reportQueryItem.value
    else { return false }
    // Do something with report string.
    return true
}
```
## Примеры отчета

### Без task_id
[Ответ без task_id](without_task_id_response.json)

### С task_id
[Ответ с task_id](with_task_id_response.json)
