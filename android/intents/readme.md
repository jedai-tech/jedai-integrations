# Взаимодействие через Android Intent

Через Android Intent можно вызвать приложение JEDAI из своего приложения и не подключать библиотеку JEDAI.

**Условие:** на устройстве установлено приложение JEDAI (`com.intrtl.app`).

Чтобы [подключить библиотеку JEDAI](../library/readme.md), используйте отдельную инструкцию.

- [Что нужно для работы](#что-нужно-для-работы)
- [Как вызвать метод](#как-вызвать-метод)
  - [Методы](#методы)
  - [Параметры вызова](#параметры-вызова)
  - [Пример вызова метода](#пример-вызова-метода)
- [Как получить ответ](#как-получить-ответ)
  - [Формат данных ответа](#формат-данных-ответа)
  - [Как получить изображения из report](#как-получить-изображения-из-report)
  - [Статусы](#статусы)
  - [Пример обработки ответа](#пример-обработки-ответа)
- [Broadcast-сообщение](#broadcast-сообщение)
  - [Содержимое broadcast-сообщения](#содержимое-broadcast-сообщения)
  - [Пример обработки broadcast-сообщения](#пример-обработки-broadcast-сообщения)
- [Пример отчета](#пример-отчета)
- [Пример сценария](#пример-сценария)
- [Возможные проблемы при интеграции](#возможные-проблемы-при-интеграции)
  - [Особенности Android 11](#особенности-android-11)

## Что нужно для работы

- На устройстве установлено приложение JEDAI.
- В проекте указан пакет `com.intrtl.app`.

Чтобы приложение JEDAI открывалось на Android 11 и новее (`targetSdkVersion` 30 и выше), добавьте в `AndroidManifest.xml` блок `<queries>`:

```xml
<queries>
    <package android:name="com.intrtl.app" />
</queries>
```

Другие варианты объявления — в разделе [Особенности Android 11](#особенности-android-11).

## Как вызвать метод

Чтобы вызвать метод приложения JEDAI, создайте `Intent` с нужным `action` и передайте параметры через `putExtra`.

Примеры ниже используют `startActivityForResult`. В AndroidX тот же сценарий закрывается через `registerForActivityResult`.

### Методы

| Метод | Что делает | Как возвращает результат |
| --- | --- | --- |
| `com.intrtl.app.ACTION_VISIT` | Создает или редактирует визит | Открывает экран (activity) |
| `com.intrtl.app.ACTION_REPORT` | Возвращает отчет по визиту | JSON-файл по `Uri` |
| `com.intrtl.app.ACTION_SUMMARY_REPORT` | Открывает сводный отчет по визиту | Открывает экран (activity) |
| `com.intrtl.app.ACTION_SYNC` | Запускает фоновую отправку фото и получение результатов | Статус в ответе |

### Параметры вызова

`action` задается методом `Intent.setAction`, остальные поля — через `Intent.putExtra`.

| Параметр | Обязательный | Методы | Описание |
| --- | --- | --- | --- |
| `login` | Да | Все | Логин пользователя |
| `password` | Да | Все | Пароль пользователя |
| `id` | Да, если вход идет через технического пользователя | Все | Внешний идентификатор пользователя |
| `visit_id` | Да | `ACTION_VISIT`, `ACTION_REPORT`, `ACTION_SUMMARY_REPORT` | Идентификатор визита |
| `task_id` | Нет | `ACTION_VISIT`, `ACTION_REPORT`, `ACTION_SUMMARY_REPORT` | Идентификатор задачи |
| `store_id` | Да | `ACTION_VISIT` | Идентификатор торговой точки |

### Пример вызова метода

```java
Intent intent = new Intent("com.intrtl.app.ACTION_VISIT");
intent.putExtra("login", user);
intent.putExtra("password", password);
intent.putExtra("id", user_id);
intent.putExtra("visit_id", visit_id);
intent.putExtra("store_id", store_id);
startActivityForResult(intent, ACTIVITY_RESULT_START_IR_VISIT);
```

## Как получить ответ

JEDAI возвращает результат через FileProvider. В `Intent.getData()` приходит `Uri` файла с JSON.

| Поле | Описание |
| --- | --- |
| `error` | Текст ошибки, если `resultCode == RESULT_CANCELED` |
| `data` | `Uri` файла с результатом операции |

### Формат данных ответа

Изучите [формат отчета](#пример-отчета). Поле `status` есть в ответе всегда. Остальные поля отсутствуют в ответе `ACTION_SYNC`.

| Поле | Описание | Когда есть в ответе |
| --- | --- | --- |
| `status` | Статус выполнения метода | Всегда |
| `user_id` | Идентификатор пользователя | Кроме `ACTION_SYNC` |
| `external_user_id` | Идентификатор пользователя из системы клиента | Кроме `ACTION_SYNC` |
| `store_id` | Идентификатор торговой точки | Кроме `ACTION_SYNC` |
| `task_id` | Идентификатор задачи | Кроме `ACTION_SYNC` |
| `visit_id` | Идентификатор визита | Кроме `ACTION_SYNC` |
| `internal_visit_id` | Внутренний идентификатор визита | Кроме `ACTION_SYNC` |
| `install_id` | Идентификатор установки | Кроме `ACTION_SYNC` |
| `photosCounter` | Количество сделанных фото | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` |
| `scenesCounter` | Количество сцен | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` |
| `notDetectedPhotosCounter` | Количество фото, по которым не получены данные | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` |
| `notDetectedScenesCounter` | Количество сцен, по которым не получены данные | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` |
| `report` | Отчет | Если `status == RESULT_OK` и метод не `ACTION_SYNC` |

### Как получить изображения из report

На Android 9 и новее берите путь к изображению из `image_uri`, а не из `image_path`.

Чтобы получить изображения из отчета:

```kotlin
private fun readBitmapFromUri(uri: Uri): Bitmap? {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
        val source = ImageDecoder.createSource(this.contentResolver, uri)
        ImageDecoder.decodeBitmap(source)
    } else {
        MediaStore.Images.Media.getBitmap(this.contentResolver, uri)
    }
}
```

```kotlin
val photosJSON = json.getJSONObject("report").getJSONObject("photos")
val photoNamesList = ArrayList<String>()
for (i in 0 until photosJSON.length()) {
    photoNamesList.add(photosJSON.names()[i] as String)
}
val arrayOfBitmap = photoNamesList.map {
    val photoUri = Uri.parse(photosJSON.getString(it))
    readBitmapFromUri(photoUri)
}
```

### Статусы

| Статус | Описание |
| --- | --- |
| `RESULT_OK` | Метод выполнен успешно |
| `RESULT_INPROGRESS` | Данные еще обрабатываются |
| `RESULT_INPROGRESS_OFFLINE` | Данные еще обрабатываются, приложение в режиме офлайн |
| `RESULT_EMPTY` | Отчет пустой |
| `ERROR_NOVISIT` | Визит не существует |
| `ERROR_READONLY_VISIT` | Визит доступен только для чтения |
| `ERROR_INCORRECT_INPUT_PARAMS` | Неверные входные параметры |
| `ERROR_VISIT_ID_INCORRECT` | Некорректный идентификатор визита |
| `ERROR_AUTH` | Ошибка авторизации |
| `ERROR_PHOTO` | Ошибка обработки фото |
| `ERROR_BUSY` | Метод уже выполняется |
| `ERROR_CANT_LOAD_VISIT` | Невозможно загрузить визит: нет интернета |

### Пример обработки ответа

`requestCode` — это константа, которую вы передали в `startActivityForResult`. По ней отличите ответ `ACTION_VISIT`, `ACTION_REPORT` и `ACTION_SUMMARY_REPORT`.

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (data == null) {
        return;
    }

    if (resultCode == RESULT_OK && data.getData() != null) {
        String result = readFromUri(data.getData());
        try {
            JSONObject json = new JSONObject(result);
            Log.i("report", json.toString());
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return;
    }

    if (resultCode == RESULT_CANCELED) {
        String error = data.getStringExtra("error");
        Log.e("report", error);
    }
}

private String readFromUri(Uri uri) {
    try {
        InputStream inputStream = getContentResolver().openInputStream(uri);
        InputStreamReader inputStreamReader = new InputStreamReader(inputStream);
        BufferedReader reader = new BufferedReader(inputStreamReader);
        StringBuffer stringBuffer = new StringBuffer();
        String string;
        while ((string = reader.readLine()) != null) {
            stringBuffer.append(string);
        }
        reader.close();
        inputStreamReader.close();
        inputStream.close();
        return stringBuffer.toString();
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

## Broadcast-сообщение

После `ACTION_VISIT` и съемки JEDAI в фоне отправляет фото и запрашивает отчеты. Когда обработка закончится, приложение отправит широковещательное сообщение (broadcast) `com.intrtl.app.BROADCAST_VISIT_COMPLETED`.

### Содержимое broadcast-сообщения

| Поле | Описание |
| --- | --- |
| `visit_id` | Идентификатор визита |
| `internal_visit_id` | Внутренний идентификатор визита |
| `user_id` | Идентификатор пользователя |
| `store_id` | Идентификатор торговой точки |
| `total_photos` | Общее количество фото. Фото плохого качества не входят в счетчик |
| `completed_photos` | Количество обработанных фото |
| `result` | Строка с `Uri` файла [отчета](#пример-отчета) |

### Пример обработки broadcast-сообщения

На Android 13 и новее укажите флаг экспорта: broadcast приходит из другого приложения.

```java
BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        Bundle extras = intent.getExtras();
        if (extras != null) {
            try {
                String reportString = readFromUri(Uri.parse(extras.getString("result")));
                JSONObject reportJson = new JSONObject(reportString);
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    }
};

IntentFilter intentFilter = new IntentFilter("com.intrtl.app.BROADCAST_VISIT_COMPLETED");
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
    registerReceiver(broadcastReceiver, intentFilter, Context.RECEIVER_EXPORTED);
} else {
    registerReceiver(broadcastReceiver, intentFilter);
}
```

## Пример отчета

Тот же JSON приходит в поле `result` broadcast-сообщения и в `getData()` в `onActivityResult`.

[Пример отчета](https://github.com/intrtl/AiletLibraryExamples/blob/master/Android/IrIntentExample/report_exaple.json)

## Пример сценария

Чтобы получить отчет по визиту:

1. Вызовите приложение JEDAI методом `com.intrtl.app.ACTION_VISIT`.
1. Сделайте несколько фото в визите.
1. Выйдите из приложения JEDAI.
1. Прочитайте статус в ответе:
    
    * Если статус `RESULT_INPROGRESS`, дождитесь broadcast `com.intrtl.app.BROADCAST_VISIT_COMPLETED`.
    * Если статус `RESULT_OK`, обработайте отчет из файла.
1. Чтобы открыть отчет или сводный отчет, вызовите `ACTION_REPORT` или `ACTION_SUMMARY_REPORT`.

## Возможные проблемы при интеграции

### Особенности Android 11

На Android 11 при `targetSdkVersion` 30 и выше система скрывает сторонние пакеты. Без `<queries>` вызов приложения JEDAI не сработает.

Чтобы приложение JEDAI открывалось, добавьте в `AndroidManifest.xml` один из вариантов.

**Вариант 1.** Укажите пакет приложения JEDAI:

```xml
<queries>
    <package android:name="com.intrtl.app" />
</queries>
```

**Вариант 2.** Укажите intent-фильтры методов:

```xml
<queries>
    <intent>
        <action android:name="com.intrtl.app.ACTION_VISIT" />
    </intent>
    <intent>
        <action android:name="com.intrtl.app.ACTION_REPORT" />
    </intent>
    <intent>
        <action android:name="com.intrtl.app.ACTION_SUMMARY_REPORT" />
    </intent>
    <intent>
        <action android:name="com.intrtl.app.ACTION_SYNC" />
    </intent>
</queries>
```

Разрешение `QUERY_ALL_PACKAGES` открывает список всех установленных приложений. Google Play принимает его только для узкого набора сценариев, поэтому для интеграции с JEDAI используйте `<queries>`.
