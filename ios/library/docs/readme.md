# Новый API

Начиная с версии `5.10`, доступен новый асинхронный API — предпочтительный способ взаимодействия с фреймворком `IrLibSwift`.
Используйте `IRInteractManager` для вызовов через новый API. Подробная документация по новым методам и классам доступна для [Swift](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-swift.md) и [Objective-C](https://github.com/intrtl/AiletLibraryExamples/blob/master/iOS/IrLibSwiftAsyncAPI/IrLibSwift-docs-objc.md).

# Совместимость со старым API

Вы можете продолжать использовать старый синхронный API через фреймворк `IRLib`, однако новый асинхронный API более предпочтителен, надёжен и удобен.

# Установка

Если вы хотите использовать новый асинхронный API, вам потребуется только фреймворк `IrLibSwift` — независимо от того, работаете вы со Swift или Objective-C проектом.

## Установка через [CocoaPods](https://cocoapods.org) ##

1. Добавьте репозиторий Intelligence Retail specs и официальный репозиторий CocoaPods specs в `Podfile` вашего проекта:

```
     source 'https://github.com/CocoaPods/Specs.git'
     source 'https://github.com/intrtl/specs'
```

2. Добавьте параметр `use_frameworks!` в ваш `Podfile`.

3. Добавьте pod `IrLibSwift` как зависимость для targets вашего проекта:

```
  target 'YourTarget' do
    pod 'IrLibSwift'
  end
```

4. Выполните `pod install` в терминале в каталоге с вашим проектом.

5. Чтобы обновить версию ранее установленного фреймворка, выполните `pod update IrLibSwift --repo-update` в терминале в каталоге с вашим проектом.
