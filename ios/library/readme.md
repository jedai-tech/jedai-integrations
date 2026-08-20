![Latest Release](https://img.shields.io/badge/latest%20release-6.5.4-brightgreen)

# Интеграция библиотеки JEDAI

Библиотека JEDAI встраивает съемку визита, отчеты и синхронизацию в ваше iOS-приложение.

Фреймворк: `IrLibSwift`. Клиент API: `IRInteractManager`. С версии 5.10 это асинхронный API.

Для асинхронного API достаточно `IrLibSwift`. Он нужен и в проекте на Swift, и в проекте на Objective-C.

Чтобы вызвать JEDAI без библиотеки, используйте [взаимодействие через iOS deeplink](../deeplink/readme.md).

Изучите [справочник методов для Swift](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md) и [справочник для Objective-C](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-objc.md).

- [Пример сценария](#пример-сценария)
- [Что нужно для работы](#что-нужно-для-работы)
- [Как установить через CocoaPods](#как-установить-через-cocoapods)
- [Как обновить фреймворк](#как-обновить-фреймворк)
- [Как инициализировать библиотеку](#как-инициализировать-библиотеку)
- [Как запустить съемку](#как-запустить-съемку)
- [Как получить отчет](#как-получить-отчет)
- [Синхронный API](#синхронный-api)

## Пример сценария

Чтобы провести визит и получить отчет:

1. Подключите `IrLibSwift` через CocoaPods.
2. Вызовите `IRInteractManager.setup(...)`.
3. Вызовите `IRInteractManager.startShooting(...)`.
4. Запросите отчет через `IRInteractManager.report(visitId:)` или подпишитесь на `IRNotification`.

## Что нужно для работы

- CocoaPods.
- Токен начальной авторизации (`guestToken`). Его выдает команда JEDAI.

## Как установить через CocoaPods

Чтобы [установить через CocoaPods](https://cocoapods.org), добавьте в `Podfile` репозитории, `use_frameworks!` и pod `IrLibSwift`:

```ruby
source 'https://github.com/CocoaPods/Specs.git'
source 'https://github.com/intrtl/specs'

use_frameworks!

target 'YourTarget' do
  pod 'IrLibSwift'
end
```

Затем в каталоге проекта выполните:

```bash
pod install
```

## Как обновить фреймворк

Чтобы обновить уже установленный `IrLibSwift`, в каталоге проекта выполните:

```bash
pod update IrLibSwift --repo-update
```

## Как инициализировать библиотеку

Чтобы начать работу, вызовите [`setup`](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md#setup). Метод авторизует пользователя и загружает данные для работы библиотеки.

```swift
IRInteractManager.setup(
    username: "user123",
    password: "securePassword",
    guestToken: "guestToken123"
) { result in
    switch result {
    case .success:
        // библиотека готова к работе
    case .failure(let error):
        // разберите IRError
    }
}
```

## Как запустить съемку

Чтобы открыть камеру JEDAI, вызовите [`startShooting`](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md#start-shooting):

```swift
do {
    try IRInteractManager.startShooting(
        in: viewController,
        externalStoreId: "store123",
        externalVisitId: "visit456"
    )
} catch {
    // разберите IRError
}
```

## Как получить отчет

Чтобы получить локальный отчет по визиту, вызовите [`report(visitId:)`](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md#retrieve-report-data-for-specific-visit):

```swift
do {
    let report = try IRInteractManager.report(visitId: "visit123")
} catch {
    // разберите ошибку
}
```

Чтобы получать обновления по распознаванию фото, подпишитесь на [`IRNotification`](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md#subscribe-for-notifications) через `NotificationCenter`.

Полный список методов, параметров и классов — в [справочнике для Swift](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md).

## Синхронный API

Синхронный API остается во фреймворке `IRLib`. Для новой интеграции подключайте только `IrLibSwift`.
