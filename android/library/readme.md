[![Latest Release](https://img.shields.io/badge/latest%20release-4.23-brightgreen)](https://github.com/intrtl/IRLib/packages)

# Интеграция библиотеки JEDAI

Библиотека JEDAI встраивает съемку визита, отчеты и синхронизацию в ваше Android-приложение.

Классы API: `Ailet`, `AiletClient`.

Пакеты Maven:

| Пакет | С версии | Что умеет |
| --- | --- | --- |
| `com.ailet.android:lib` | — | Съемка, отчеты, синхронизация. Распознавание только на сервере |
| `com.ailet.android:lib-offline` | **4.23** | То же, плюс on-device распознавание (Palomna) |

Подключите только один из пакетов. Пакеты `lib` и `lib-offline` вместе подключать нельзя.

Чтобы вызвать JEDAI без библиотеки, используйте [взаимодействие через Android Intent](../intents/readme.md).

- [Пример сценария](#пример-сценария)
- [Что нужно для работы](#что-нужно-для-работы)
  - [Как создать GitHub personal access token](#как-создать-github-personal-access-token)
  - [Как подключить репозиторий Maven](#как-подключить-репозиторий-maven)
  - [Как добавить зависимости](#как-добавить-зависимости)
  - [Правила ProGuard](#правила-proguard)
- [Как инициализировать библиотеку](#как-инициализировать-библиотеку)
- [Как вызывать методы](#как-вызывать-методы)
- [On-device распознавание (Palomna)](#on-device-распознавание-palomna)
  - [Что изменилось в 4.23](#что-изменилось-в-423)
  - [Как подготовить устройство](#как-подготовить-устройство)
- [Справочник методов](#справочник-методов)
  - [getServers()](#getservers)
  - [init()](#init)
  - [start()](#start)
  - [getReports()](#getreports)
  - [showSummaryReport()](#showsummaryreport)
  - [setPortal()](#setportal)
  - [requestSyncCatalogs()](#requestsynccatalogs)
  - [showVisit()](#showvisit)
  - [finishVisit()](#finishvisit)
  - [logout()](#logout)
  - [getTotalSyncStat()](#gettotalsyncstat)
  - [syncPalomna()](#syncpalomna)
  - [syncPalomnaCatalogs()](#syncpalomnacatalogs)
  - [updatePalomna()](#updatepalomna)
- [Широковещательное сообщение](#широковещательное-сообщение)
- [Миграция с IntRtl](#миграция-с-intrtl)
- [Пример отчета](#пример-отчета)
- [Известные проблемы](#известные-проблемы)
  - [Gradle 8.x и обфускация](#gradle-8x-и-обфускация)

## Пример сценария

Чтобы провести визит и получить отчет:

1. Создайте GitHub personal access token с правом `read:packages`.
2. Подключите репозиторий Maven и зависимость `com.ailet.android:lib` или `com.ailet.android:lib-offline`.
3. Вызовите `Ailet.initialize` в классе `Application`.
4. Вызовите `init()` через `Ailet.getClient()`.
5. Вызовите `start()` и сделайте фото.
6. Дождитесь broadcast о готовности отчета или вызовите `getReports()`.

## Что нужно для работы

- Токен начальной авторизации. Его выдает команда JEDAI.
- GitHub-аккаунт с правом читать пакеты `intrtl/IRLib`.

Актуальную версию библиотеки смотрите в [списке пакетов](https://github.com/intrtl/IRLib/packages). Для on-device нужна версия **4.23** и выше пакета `com.ailet.android:lib-offline`.

### Как создать GitHub personal access token

Чтобы скачивать пакеты JEDAI из GitHub Packages:

1. Откройте GitHub.
2. Нажмите аватар в правом верхнем углу.
3. Выберите **Settings**.
4. Откройте **Developer settings**.
5. Откройте **Personal access tokens** → **Tokens (classic)**.
6. Нажмите **Generate new token**.
7. Включите право `read:packages`.
8. Нажмите **Generate token**.
9. Сохраните токен: GitHub покажет его один раз.

Не коммитьте токен в репозиторий. Вынесите логин и токен в `gradle.properties` или в переменные окружения.

### Как подключить репозиторий Maven

Чтобы Gradle скачивал библиотеку JEDAI, добавьте репозиторий одним из способов.

**Вариант 1.** Добавьте репозиторий в `settings.gradle`:

```groovy
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()

        maven {
            url 'https://maven.pkg.github.com/intrtl/irlib'
            credentials {
                username 'your GitHub username'
                password 'personal GitHub access token'
            }
        }
    }
}
```

**Вариант 2.** Добавьте репозиторий в корневой `build.gradle`:

```groovy
allprojects {
    repositories {
        maven {
            url 'https://maven.pkg.github.com/intrtl/irlib'
            credentials {
                username 'your GitHub username'
                password 'personal GitHub access token'
            }
        }
    }
}
```

### Как добавить зависимости

Чтобы подключить библиотеку, добавьте в `build.gradle` модуля одну зависимость. Подставьте версию из [списка пакетов](https://github.com/intrtl/IRLib/packages):

Онлайн-распознавание:

```groovy
implementation "com.ailet.android:lib:4.23.0"
```

Онлайн и on-device распознавание (доступно с версии **4.23**):

```groovy
implementation "com.ailet.android:lib-offline:4.23.0"
```

Не подключайте пакеты `lib` и `lib-offline` в одной сборке: это разные публикации одного API.

Динамическая версия `+` подтянет последний пакет и может сломать сборку без изменения кода. Для рабочих сборок указывайте конкретную версию.

Чтобы включить модуль техподдержки, добавьте зависимость той же версии, что и у библиотеки:

```groovy
implementation "com.ailet.android:lib-feature-techsupport-intercom:4.23.0"
```

### Правила ProGuard

Чтобы обфускация не ломала библиотеку, добавьте в `proguard-rules.pro`:

```proguard
-keep class com.ailet.** { *; }
-keep class com.ailet.lib3.** { *; }
-keep interface com.ailet.lib3.** { *; }
-keep enum com.ailet.lib3.** { *; }

-keepclassmembers class com.ailet.lib3.** {
    *** *(...);
}

-keepnames class com.ailet.lib3.** { *; }
-keepattributes *Annotation*
-keepattributes Signature

-keep class com.google.gson.** { *; }
-keep class sun.misc.Unsafe { *; }
-keep interface com.google.gson.TypeAdapter
-keep interface com.google.gson.JsonSerializer
-keep interface com.google.gson.JsonDeserializer

-dontwarn com.ailet.lib3.**
```

## Как инициализировать библиотеку

Чтобы начать работу, вызовите `Ailet.initialize` в наследнике `Application`:

```kotlin
class App : Application() {

    override fun onCreate() {
        super.onCreate()

        val features = setOf<AiletFeature>(
            DefaultStockCameraFeature(),
            IntercomTechSupportManager(this),
            HostAppInstallInfoProviderFeature(
                this,
                BuildConfig.VERSION_NAME,
                BuildConfig.VERSION_CODE,
                AiletLibInstallInfo
            )
        )

        val accessToken = "..."

        Ailet.initialize(this, accessToken, features)
    }
}
```

Модули в `features` необязательны:

- `DefaultStockCameraFeature` — стоковая камера;
- `IntercomTechSupportManager` — техподдержка;
- `HostAppInstallInfoProviderFeature` — идентификация сборки для диагностики.

После `initialize` вызывайте методы через `Ailet.getClient()`.

Чтобы экран камеры JEDAI не закрывался сразу после открытия, когда разрешение на камеру уже выдано, добавьте в `features`:

```kotlin
DefaultAiletPermissionsFeature(
    excludedPermissions = setOf(AiletPermissionsFeature.Exclude.CAMERA)
)
```

## Как вызывать методы

Вызов метода возвращает `AiletCall`. Дальше выполните его асинхронно через `execute()` или синхронно через `executeBlocking()`.

Асинхронный вызов:

```kotlin
Ailet.getClient()
    .setPortal(portalName)
    .execute({ result ->
        when (result) {
            // обработка результата
        }
    }, { throwable ->
        // обработка ошибки
    })
```

Синхронный вызов. Поток исполнения выбираете вы:

```kotlin
val result = Ailet.getClient()
    .setPortal(portalName)
    .executeBlocking()
```

В Java у параметров нет значений по умолчанию. Передайте все аргументы явно:

```java
Ailet.getClient().init(
    "login",
    "password",
    null,
    false,
    null,
    false
).execute(
    result -> {
        // обработка результата
        return null;
    },
    throwable -> {
        // обработка ошибки
        return null;
    },
    () -> {
        // завершение вызова
        return null;
    }
);
```

## On-device распознавание (Palomna)

С версии **4.23** on-device распознавание поставляется пакетом `com.ailet.android:lib-offline`. Пакет `com.ailet.android:lib` распознает фото только на сервере.

Если подключили `lib`, пропускайте пометки «`lib-offline`, с 4.23».

### Что изменилось в 4.23

- Отдельный Maven-артефакт `com.ailet.android:lib-offline`.
- Методы [`syncPalomna()`](#syncpalomna), [`syncPalomnaCatalogs()`](#syncpalomnacatalogs), [`updatePalomna()`](#updatepalomna).
- В методах `getReports`, broadcast и `getTotalSyncStat` — поля `source` (`online` / `on-device`) и `completed_on_device`.
- В продуктах отчета — поле `eye_level` для онлайн и on-device.
- Понятные исключения загрузки: `OnDeviceNotAvailableException`, `OnDeviceDownloadMobileException`, `OnDeviceNeedUpdateException`, `OnDeviceDownloadFailedException`, `OnDeviceNoStoreException`.
- Метод `getReports` бросает `OnDeviceNotAvailableException`, если нет сети, в визите есть нераспознанные фото, on-device включен, но модели не загружены.
- Статус `RESULT_OK` выставляется, когда все фото обработаны и по визиту есть виджет. Для `source = on-device` обработанными считаются фото в статусах `COMPLETE` и `COMPLETED_WITH_PALOMNA`.
- В `getTotalSyncStat.total_stat.current_problem` попадает последняя ошибка синхронизации или on-device распознавания.

### Как подготовить устройство

1. Подключите зависимость `com.ailet.android:lib-offline:4.23.x`.
2. Вызовите метод `init()`.
3. Пока есть сеть, вызовите [`syncPalomna()`](#syncpalomna). По умолчанию загрузка идет только по Wi-Fi.
4. Вызовите метод `start()`. Если модели и классы загружены, то без интернета фото обрабатываются на устройстве.

Чтобы догрузить справочники (матрицы, метрики, типы матриц), вызовите метод [`syncPalomnaCatalogs()`](#syncpalomnacatalogs). Чтобы обновить уже загруженные модели и классы, вызовите [`updatePalomna()`](#updatepalomna).

## Справочник методов

| Метод | Что делает |
| --- | --- |
| [`init`](#init) | Авторизует пользователя, поднимает библиотеку и загружает справочники |
| [`getServers`](#getservers) | Возвращает список доступных порталов |
| [`start`](#start) | Запускает съемку визита |
| [`getReports`](#getreports) | Возвращает отчет по визиту |
| [`showSummaryReport`](#showsummaryreport) | Открывает сводный отчет по визиту |
| [`setPortal`](#setportal) | Устанавливает активный портал |
| [`requestSyncCatalogs`](#requestsynccatalogs) | Загружает справочники |
| [`showVisit`](#showvisit) | Открывает фотографии визита |
| [`finishVisit`](#finishvisit) | Завершает визит |
| [`logout`](#logout) | Выходит из учетной записи |
| [`getTotalSyncStat`](#gettotalsyncstat) | Возвращает статистику синхронизации визитов |
| [`syncPalomna`](#syncpalomna) | **(пакет `lib-offline`, с версии 4.23)** Загружает модели и справочники для on-device распознавания |
| [`syncPalomnaCatalogs`](#syncpalomnacatalogs) | **(пакет `lib-offline`, с версии 4.23)** Догружает справочники Palomna (матрицы, метрики, типы матриц) |
| [`updatePalomna`](#updatepalomna) | **(пакет `lib-offline`, с версии 4.23)** Обновляет уже загруженные модели и классы |

Чтобы [мигрировать с `IntRtl`](#миграция-с-intrtl), используйте `AiletClient`.

### getServers()

`getServers()` возвращает список серверов `AiletServer`. Вызывайте его только в мультипортальном режиме. Затем передайте выбранный сервер в `init()`.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `login` | `String` | Да | — | Логин пользователя в JEDAI |
| `password` | `String` | Да | — | Пароль пользователя в JEDAI |
| `externalUserId` | `String` | Нет | `null` | Внешний идентификатор пользователя |

**Ошибки**

| Ошибка | Описание |
| --- | --- |
| `BackendApiException` | Ошибка сервера с [HTTP-кодом](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) |

### init()

`init()` авторизует пользователя, поднимает библиотеку и загружает справочники. При необходимости запускает сервис синхронизации.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `login` | `String` | Да | — | Логин пользователя в JEDAI |
| `password` | `String` | Да | — | Пароль пользователя в JEDAI |
| `externalUserId` | `String` | Нет | `null` | Внешний идентификатор пользователя |
| `multiPortalMode` | `Boolean` | Нет | `true` | Включить мультипортальный режим |
| `server` | `AiletServer` | Нет | `null` | Сервер, на который выполняется вход |
| `isNeedSyncCatalogs` | `Boolean` | Нет | `true` | Синхронизировать каталоги при входе |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `DataInconsistencyException` | `Current auth state data is null` | Данные аутентификации некорректны |
| `IllegalStateException` | `Inconsistency! server is null` | Сервер пустой |
| `IllegalStateException` | `No portals available` | Нет доступных порталов |
| `BackendApiException` | — | Ошибка сервера с [HTTP-кодом](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) |
| Другие типы исключений | — | Внутренняя ошибка библиотеки. Обратитесь в поддержку |

### start()

`start()` запускает съемку в рамках визита.

> **On-device (пакет `lib-offline`, с версии 4.23):** если модели и классы загружены, при отсутствии интернета фото обрабатываются на устройстве.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `storeId` | `AiletMethodStart.StoreId` | Да | — | Внешний идентификатор торговой точки |
| `externalVisitId` | `String` | Нет | `null` | Внешний идентификатор визита |
| `sceneGroupId` | `Int` | Нет | `null` | Идентификатор группы сцен |
| `taskId` | `String` | Нет | `null` | Внешний идентификатор задачи |
| `visitType` | `String` | Нет | `null` | Тип визита (`before`, `after`) |
| `visitUuid` | `String` | Нет | `null` | Внутренний идентификатор визита |
| `retailTaskIterationUuid` | `String` | Нет | `null` | Идентификатор итерации (ритейл) |
| `retailTaskId` | `String` | Нет | `null` | Идентификатор задачи (ритейл) |
| `retailTaskActionId` | `String` | Нет | `null` | Идентификатор действия (ритейл) |
| `sceneTypes` | `List` | Нет | `listOf()` | Список типов сцен |
| `launchConfig` | `LaunchConfig` | Нет | `AiletMethodStart.LaunchConfig()` | Конфигурация запуска |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Uneditable(historical) visit` | Визит завершен и недоступен для редактирования |
| `IllegalStateException` | `Inconsistent AiletClient state: (Unknown, Warning, Error)` | Несогласованное состояние клиента |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |

### getReports()

`getReports()` возвращает отчет по визиту в JSON. Изучите [формат отчета](#пример-отчета).

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `externalVisitId` | `String` | Да | — | Внешний идентификатор визита |
| `taskId` | `String` | Нет | `null` | Внешний идентификатор задачи |
| `visitType` | `String` | Нет | `null` | Тип визита (`before`, `after`) |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `AiletException` | `Visit with externalId [externalId] is not found` | Визит с таким идентификатором не найден |
| `OnDeviceNotAvailableException` | `On-device recognition not available` | **(пакет `lib-offline`, с версии 4.23)** Нет сети, в визите есть нераспознанные фото, on-device включен, но модели не загружены |

> **On-device (пакет `lib-offline`, с версии 4.23):** в поля `result` и `report.result` добавляются:
> - `source`: `"online"`  — если все обработанные фото уже на сервере; `"on-device"` — если есть фото в Palomna-пайплайне без онлайн-пересчета.
> - `completed_on_device`: количество фото со статусом `COMPLETED_WITH_PALOMNA`.
>
> Статус `RESULT_OK` выставляется, когда все фото обработаны и по визиту есть виджет. Для `source = on-device` обработанными считаются фото `COMPLETE` и `COMPLETED_WITH_PALOMNA`.
>
> В объектах продуктов отчета добавляется параметр `eye_level`: `1` — на уровне глаз, `0` — нет, `-1` — определить нельзя. Поле есть и в онлайн, и в on-device.

### showSummaryReport()

`showSummaryReport()` открывает экран сводного отчета по визиту.

> **On-device (пакет `lib-offline`, с версии 4.23):** экран показывает данные on-device распознавания.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `externalVisitId` | `String` | Да | — | Внешний идентификатор визита |
| `taskId` | `String` | Нет | `null` | Внешний идентификатор задачи |
| `visitType` | `String` | Нет | `null` | Тип визита (`before`, `after`) |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |
| `IllegalArgumentException` | `No store for externalId [externalId]` | Нет торговой точки с таким внешним идентификатором |
| `IllegalArgumentException` | `No store for storeId [storeId]` | Нет торговой точки с таким идентификатором |
| `IllegalArgumentException` | `No visit for summary report request $param found` | Визит для сводного отчета не найден |
| `IllegalArgumentException` | `Incorrect historical visit params` | Некорректные параметры завершенного визита |
| `BackendApiException` | — | Ошибка сервера с [HTTP-кодом](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) |
| `RuntimeException` | `No visit/Offline` | Нет визита, устройство офлайн |

### setPortal()

`setPortal()` устанавливает текущий портал в мультипортальном режиме.

> **On-device (пакет `lib-offline`, с версии 4.23):** при переключении портала сохраняются и применяются настройки on-device распознавания для этого портала. После переключения вызовите `syncPalomna()`, если для портала еще не загружали модели.

| Параметр | Тип | Обязательный | Описание |
| --- | --- | --- | --- |
| `portalName` | `String` | Да | Идентификатор портала |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |
| `IllegalArgumentException` | `No [server] found in local portals list` | Сервер не найден в локальном списке порталов |
| `AiletException` | `no [server] in servers list` | Сервера нет в списке |

### requestSyncCatalogs()

`requestSyncCatalogs()` загружает справочники выбранного портала в мультипортальном режиме.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `syncMode` | `AiletMethodSyncCatalogs.SyncMode` | Нет | `AiletMethodSyncCatalogs.SyncMode.EAGER` | `EAGER` — все справочники. `SOFT` — только обязательные |
| `strategy` | `AiletMethodSyncCatalogs.Strategy` | Нет | `AiletMethodSyncCatalogs.Strategy.Schedule` | `Schedule` — поставить загрузку в очередь. `SyncRightNow` — синхронизировать сразу |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |

Чтобы загрузить справочники для всех порталов:

```kotlin
Ailet.getClient()
    .getServers(
        "login",
        "password",
        "external_user_id"
    )
    .execute({ result ->
        runBlocking {
            result.servers.forEach { server ->
                Ailet.getClient().setPortal(
                    portalName = server.name
                ).executeBlocking()

                Ailet.getClient().requestSyncCatalogs(
                    syncMode = AiletMethodSyncCatalogs.SyncMode.EAGER,
                    strategy = AiletMethodSyncCatalogs.Strategy.SyncRightNow
                ).executeBlocking()
            }
        }
        // действия после загрузки всех справочников
    }, { throwable ->
        // обработка ошибки
    })
```

### showVisit()

`showVisit()` открывает экран фотографий визита. Открывается первая фотография.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `externalVisitId` | `String` | Да | — | Внешний идентификатор визита |
| `taskId` | `String` | Нет | `null` | Внешний идентификатор задачи |
| `visitType` | `String` | Нет | `null` | Тип визита (`before`, `after`) |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |
| `IllegalArgumentException` | `No visit with id: [visitId]` | Визит не найден |
| `IndexOutOfBoundsException` | `No photos in visit with id: [$visitId]` | В визите нет фото |
| `RuntimeException` | `No visit/Offline` | Нет локального визита, устройство офлайн. Проверить визит на сервере нельзя |
| `BackendApiException` | — | Ошибка сервера с [HTTP-кодом](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) |

### finishVisit()

`finishVisit()` закрывает визит. После этого визит доступен только для просмотра.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `externalVisitId` | `String` | Да | — | Внешний идентификатор визита |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |
| `AiletException` | `Visit with externalId [externalVisitId] is already finished` | Визит уже завершен |
| `AiletException` | `Visit with externalId [externalVisitId] is not found` | Визит не найден |

### logout()

`logout()` выходит из учетной записи и очищает служебные данные.

> **On-device (пакет `lib-offline`, с версии 4.23):** если on-device распознавание включено в мобильных настройках, библиотека удалит загруженные модели, классы и справочники. Это произойдет только если следующий вызов метода `init()` будет выполнен для другого пользователя.

### getTotalSyncStat()

Доступно в версии 4.17.3 и выше. Поля `source` и `completed_on_device` есть в пакете `lib-offline` с версии **4.23**.

`getTotalSyncStat()` возвращает статистику по фотографиям и запускает сервис синхронизации, если он остановлен. Результат — JSON-строка.

> **On-device (пакет `lib-offline`, с версии 4.23):** в каждый элемент `items` и в `total_stat` добавляются поля `source` (`online` / `on-device`) и `completed_on_device`. В `total_stat.current_problem` попадает последняя ошибка синхронизации или on-device распознавания.
>
> `source = on-device`, если есть фото в Palomna-пайплайне без онлайн-пересчета. `completed_on_device` — число фото в статусе `COMPLETED` с данными on-device распознавания. `RESULT_OK` — все фото обработаны и есть виджет (онлайн или on-device, в зависимости от `source`).

**Пример ответа**

```json
{
    "items": [
        {
            "visit_id": "3",
            "source": "online",
            "visit_external_id": "156r459",
            "total_photos": 10,
            "sent_photos": 10,
            "completed_photos": 10,
            "completed_on_device": 2,
            "code": "RESULT_OK",
            "code_int": 1,
            "message": "Успешно обработан"
        },
        {
            "visit_id": "2",
            "source": "on-device",
            "visit_external_id": "156r46",
            "total_photos": 10,
            "sent_photos": 1,
            "completed_photos": 0,
            "completed_on_device": 9,
            "code": "IN_PROGRESS",
            "code_int": 16,
            "message": "Выполняется синхронизация"
        }
    ],
    "total_stat": {
        "total_photos": 20,
        "sent_photos": 11,
        "completed_photos": 10,
        "completed_on_device": 11,
        "current_problem": "Ошибка отправки фото на сервер"
    }
}
```

### syncPalomna()

Доступно в пакете `lib-offline` с версии **4.23**.

Метод `syncPalomna()` заранее загружает и обновляет модели, классы и справочники для распознавания на устройстве без интернета.

Если поля `storeIds` и `externalIds` пустые, библиотека загружает все торговые точки (если их еще нет) и матрицы для ближайших 1000 точек — по геолокации, если она есть, иначе для первых 1000 из справочника. Если передана одна торговая точка, загружаются матрицы для 1000 ближайших к ней. Если передано несколько — только для указанных идентификаторов.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `storeIds` | `List<Long>` | Нет | `listOf()` | Внутренние идентификаторы торговых точек в библиотеке |
| `externalIds` | `List<String>` | Нет | `listOf()` | Внешние идентификаторы торговых точек |
| `useMobile` | `Boolean` | Нет | `false` | Разрешить синхронизацию через мобильную сеть |
| `isAutoUpdate` | `Boolean` | Нет | `false` | Обновлять модели без запроса пользователя. Если `false` и есть обновление, метод бросит `OnDeviceNeedUpdateException` |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `OnDeviceNotAvailableException` | `On-device not available` | On-device распознавание выключено в настройках или нет моделей и классов |
| `OnDeviceDownloadMobileException` | `Can't download via mobile network` | Загрузка через мобильную сеть запрещена (`useMobile = false`) |
| `OnDeviceNeedUpdateException` | `Need update N model(s)` | Есть обновление моделей, но `isAutoUpdate = false` |
| `OnDeviceDownloadFailedException` | `On-device download failed` | Загрузка моделей или справочников не удалась |
| `OnDeviceNoStoreException` | `Store not found` / `Stores catalog empty` | Торговая точка не найдена или справочник торговых точек пуст |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |

При изменении статуса загрузки Palomna библиотека отправляет широковещательное сообщение (broadcast) с `intent.action = SYNC_PALOMNA_STATE`.

| Extra | Описание |
| --- | --- |
| `dataSetsProgress` | Прогресс загрузки моделей |
| `matricesProgress` | Прогресс загрузки матриц |
| `matricesTypesProgress` | Прогресс загрузки типов матриц |
| `metricsProgress` | Прогресс загрузки метрик |
| `imagesProgress` | Прогресс загрузки изображений |

### syncPalomnaCatalogs()

Доступно в пакете `lib-offline` с версии **4.23**.

Метод `syncPalomnaCatalogs()` догружает справочники Palomna: матрицы, типы матриц и метрики. Модели и классы метод не обновляет — для этого используйте метод [`updatePalomna()`](#updatepalomna) или [`syncPalomna()`](#syncpalomna).

Передайте координаты, чтобы выбрать ближайшие торговые точки. Если координаты не переданы, библиотека использует уже сохраненное местоположение или первые точки из справочника.

| Параметр | Тип | Обязательный | По умолчанию | Описание |
| --- | --- | --- | --- | --- |
| `lat` | `Double` | Нет | `null` | Широта для выбора ближайших торговых точек |
| `lng` | `Double` | Нет | `null` | Долгота для выбора ближайших торговых точек |

**Ошибки**

| Ошибка | Текст ошибки | Описание |
| --- | --- | --- |
| `OnDeviceNoStoreException` | `Store not found` / `Stores catalog empty` | Торговая точка не найдена или справочник торговых точек пуст |
| `OnDeviceDownloadFailedException` | `On-device download failed` | Загрузка справочников не удалась |
| `Throwable` | `Unauthorized` | Пользователь не авторизован |

Прогресс загрузки приходит тем же broadcast `SYNC_PALOMNA_STATE`, что и у `syncPalomna()`.

### updatePalomna()

Доступно в пакете `lib-offline` с версии **4.23**.

Метод `updatePalomna()` обновляет уже загруженные модели и классы, если сервер отдал новые версии. Параметров нет.

Вызывайте метод, когда `syncPalomna()` вернул `OnDeviceNeedUpdateException`, или чтобы проверить обновления отдельно.

**Ошибки**

| Ошибка | Описание |
| --- | --- |
| `OnDeviceDownloadFailedException` | Загрузка обновления не удалась |
| `Throwable` | Внутренняя ошибка библиотеки. Обратитесь в поддержку |

## Широковещательное сообщение

Когда библиотека получит все данные по визиту, она отправит broadcast с `intent.action = com.ailet.app.BROADCAST_WIDGETS_RECEIVED` или `com.ailet.russia.BROADCAST_WIDGETS_RECEIVED`.

На Android 13 и новее укажите флаг экспорта: сообщение приходит из библиотеки как отдельный компонент.

Чтобы обработать сообщение:

```kotlin
broadcastReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        parseBroadcastMessage(intent)
    }
}

val intentFilter = IntentFilter(IR_BROADCAST_V3)
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    registerReceiver(broadcastReceiver, intentFilter, Context.RECEIVER_EXPORTED)
} else {
    registerReceiver(broadcastReceiver, intentFilter)
}

private const val NOT_SET = "not set"
private const val VISIT_ID = "visit_id"
private const val INTERNAL_VISIT_ID = "internal_visit_id"
private const val STORE_ID = "store_id"
private const val TASK_ID = "task_id"
private const val TOTAL_PHOTOS = "total_photos"
private const val COMPLETED_PHOTOS = "completed_photos"
private const val COMPLETED_ON_DEVICE = "completed_on_device"
private const val SOURCE = "source"
private const val RESULT = "result"

private fun parseBroadcastMessage(intent: Intent) {
    val extras = intent.extras
    val visitId = extras?.getString(VISIT_ID, NOT_SET)
    val internalVisitId = extras?.getString(INTERNAL_VISIT_ID, NOT_SET)
    val storeId = extras?.getString(STORE_ID, NOT_SET)
    val taskId = extras?.getString(TASK_ID, NOT_SET)
    val totalPhotos = extras?.getInt(TOTAL_PHOTOS, 0)
    val completedPhotos = extras?.getInt(COMPLETED_PHOTOS, 0)
    val completedOnDevice = extras?.getInt(COMPLETED_ON_DEVICE, 0)
    val source = extras?.getString(SOURCE, "online")
    val result = extras?.getString(RESULT, null)

    result?.let { uriString ->
        try {
            val fileFromUri = readFromUri(Uri.parse(uriString))
        } catch (t: Throwable) {
            t.printStackTrace()
        }
    }
}
```

`IR_BROADCAST_V3` — константа action в вашем проекте. Подставьте `com.ailet.app.BROADCAST_WIDGETS_RECEIVED` или `com.ailet.russia.BROADCAST_WIDGETS_RECEIVED` в зависимости от сборки.

| Параметр | Тип | Описание |
| --- | --- | --- |
| `internal_visit_id` | `String` | Внутренний идентификатор визита в JEDAI |
| `visit_id` | `String` | Идентификатор визита |
| `store_id` | `String` | Идентификатор торговой точки |
| `user_id` | `String` | Идентификатор пользователя в JEDAI |
| `total_photos` | `Int` | Количество фото в визите |
| `completed_photos` | `Int` | Количество обработанных фото |
| `completed_on_device` | `Int` | **(`lib-offline`, с 4.23)** Количество фото, распознанных on-device |
| `source` | `String` | **(пакет `lib-offline`, с версии 4.23)** Источник данных (`online` / `on-device`) |
| `result` | `String` | `Uri` файла отчета |

## Миграция с IntRtl

Начиная с версии 3.0 класс-клиент `IntRtl` отмечен как устаревший. Используйте `AiletClient`.

Методы нового клиента соответствуют [методам устаревшего клиента](https://github.com/intrtl/AiletLibraryExamples/blob/master/Android/IrLibExample/readme.md#методы).

В аннотацию `Deprecated` каждого метода `IntRtl` добавлены блоки `ReplaceWith`. Android Studio может заменить старый вызов на новый по подсказке.

Отличия нового клиента:

1. Методы не блокируют поток. Вызов возвращает `AiletCall`. Дальше используйте `execute()` или `executeBlocking()`.
2. При `executeBlocking()` поток исполнения выбираете вы.

До версии 3.0.0:

```kotlin
client.setPortal(portalName)
```

Начиная с версии 3.0.0:

```kotlin
Ailet.getClient()
    .setPortal(portalName)
    .execute({ result ->
        when (result) {
            // обработка результата
        }
    }, { throwable ->
        // обработка ошибки
    })
```

## Пример отчета

Полный JSON лежит в файле [report_exaple.json](./report_exaple.json).

## Известные проблемы

### Gradle 8.x и обфускация

Чтобы сборка с Gradle 8.x не падала при обфускации, проверьте правила в разделе [Правила ProGuard](#правила-proguard). Для Gradle 8.x нужен keep для пакета `com.ailet.**`.
