# Взаимодействие через Android Intent

Через Android Intent можно вызвать приложение JEDAI из своего приложения и не подключать библиотеку JEDAI.

**Условие:** на устройстве установлено приложение JEDAI (`com.intrtl.app`).

**On-device** распознавание доступно в сборке **JEDAI Offline 4.23** и новее, пакет `tech.jedai.offline`.

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
- [On-device распознавание](#on-device-распознавание)
  - [Сборка и настройки](#сборка-и-настройки)
  - [ACTION_SYNC_PALOMNA](#action_sync_palomna)
  - [Выбор торговых точек](#выбор-торговых-точек)
  - [Ошибки ACTION_SYNC_PALOMNA](#ошибки-action_sync_palomna)
  - [Поля отчета и ActivityResult](#поля-отчета-и-activityresult)
  - [Работа без интернета](#работа-без-интернета)
  - [Рекомендуемый порядок вызовов](#рекомендуемый-порядок-вызовов)
- [Возможные проблемы при интеграции](#возможные-проблемы-при-интеграции)
  - [Особенности Android 11](#особенности-android-11)

## Что нужно для работы

- На устройстве установлено приложение JEDAI.
- В проекте указан пакет `com.intrtl.app`.
- Для on-device указан пакет `tech.jedai.offline` и установлена сборка JEDAI Offline **4.23** или новее.

Чтобы приложение JEDAI открывалось на Android 11 и новее (`targetSdkVersion` 30 и выше), добавьте в `AndroidManifest.xml` блок `<queries>`:

```xml
<queries>
    <package android:name="com.intrtl.app" />
</queries>
```

Для JEDAI Offline укажите пакет сборки:

```xml
<queries>
    <package android:name="tech.jedai.offline" />
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
| `com.ailet.ACTION_SYNC_PALOMNA` | Загружает и обновляет модели, классы и справочники on-device. Сборка JEDAI Offline **4.23+** | `RESULT_OK` или ошибка в extras |

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
| `external_ids` | Нет | `ACTION_SYNC_PALOMNA` | Внешние идентификаторы торговых точек. `ArrayList<String>` или строка через запятую. По умолчанию пустой список |
| `store_ids` | Нет | `ACTION_SYNC_PALOMNA` | Внутренние идентификаторы торговых точек. `int[]`, `long[]` или `ArrayList<Integer>`. По умолчанию пустой список |

Как по `external_ids` и `store_ids` выбираются точки для матриц — в разделе [Выбор торговых точек](#выбор-торговых-точек).

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

Пример `ACTION_SYNC_PALOMNA`:

```java
Intent intent = new Intent("com.ailet.ACTION_SYNC_PALOMNA");
intent.setPackage("tech.jedai.offline");
intent.putExtra("login", user);
intent.putExtra("password", password);
intent.putStringArrayListExtra("external_ids", externalIds);
startActivityForResult(intent, ACTIVITY_RESULT_SYNC_PALOMNA);
```

## Как получить ответ

JEDAI возвращает результат через FileProvider. В `Intent.getData()` приходит `Uri` файла с JSON.

| Поле | Описание |
| --- | --- |
| `error` | Текст ошибки, если `resultCode == RESULT_CANCELED` |
| `data` | `Uri` файла с результатом операции |

### Формат данных ответа

Изучите [формат отчета](#пример-отчета). Поле `status` есть в ответе всегда. Остальные поля отсутствуют в ответе `ACTION_SYNC` и `ACTION_SYNC_PALOMNA`.

| Поле | Описание | Когда есть в ответе |
| --- | --- | --- |
| `status` | Статус выполнения метода | Всегда |
| `user_id` | Идентификатор пользователя | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `external_user_id` | Идентификатор пользователя из системы клиента | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `store_id` | Идентификатор торговой точки | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `task_id` | Идентификатор задачи | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `visit_id` | Идентификатор визита | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `internal_visit_id` | Внутренний идентификатор визита | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `install_id` | Идентификатор установки | Кроме `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |
| `photosCounter` | Количество сделанных фото | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` и не `ACTION_SYNC_PALOMNA` |
| `scenesCounter` | Количество сцен | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` и не `ACTION_SYNC_PALOMNA` |
| `notDetectedPhotosCounter` | Количество фото, по которым не получены данные | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` и не `ACTION_SYNC_PALOMNA` |
| `notDetectedScenesCounter` | Количество сцен, по которым не получены данные | Если `status != ERROR_VISIT_ID_INCORRECT` и метод не `ACTION_SYNC` и не `ACTION_SYNC_PALOMNA` |
| `report` | Отчет | В файле ответа. Файла нет у `ACTION_SYNC` и `ACTION_SYNC_PALOMNA` |

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

[Пример отчета](https://github.com/jedai-tech/jedai-integrations/blob/main/android/library/report_exaple.json) — исторический онлайн-отчет. В `report.result` добавлено поле `source`, в `visit_stats.photo` — `completed_on_device`.

Фрагмент отчета после on-device распознавания, когда все фото обработаны и по визиту есть виджет:

```json
{
  "status": "RESULT_OK",
  "visit_id": "visit-1",
  "photosCounter": 1,
  "report": {
    "result": {
      "source": "on-device",
      "visit_id": "visit-1",
      "total_photos": 1,
      "sended_photos": 0,
      "recognized_photos": 1,
      "code": "RESULT_OK",
      "codeInt": 1,
      "message": "Успешно обработан"
    },
    "visit_stats": {
      "photo": {
        "completed": 1,
        "completed_online": 0,
        "completed_on_device": 1,
        "status": "RESULT_OK"
      }
    }
  }
}
```

`recognized_photos` есть в `report.result`, когда Palomna включена. `completed_on_device` — число фото в обработанных on-device.

## Пример сценария

Чтобы получить отчет по визиту:

1. Вызовите приложение JEDAI методом `com.intrtl.app.ACTION_VISIT`.
1. Сделайте несколько фото в визите.
1. Выйдите из приложения JEDAI.
1. Прочитайте статус в ответе:
    
    * Если статус `RESULT_INPROGRESS`, дождитесь broadcast `com.intrtl.app.BROADCAST_VISIT_COMPLETED`.
    * Если статус `RESULT_OK`, обработайте отчет из файла.
1. Чтобы открыть отчет или сводный отчет, вызовите `ACTION_REPORT` или `ACTION_SUMMARY_REPORT`.

Чтобы получить отчет без интернета, следуйте [рекомендуемому порядку](#рекомендуемый-порядок-вызовов): сначала `ACTION_SYNC_PALOMNA`, затем `ACTION_VISIT`.

## On-device распознавание

С версии **4.23** сборка JEDAI Offline распознает фото на устройстве и отдает отчет без интернета. Пакет приложения — `tech.jedai.offline`.

В той же сборке по-прежнему работают `com.intrtl.app.ACTION_VISIT`, `com.intrtl.app.ACTION_REPORT`, `com.intrtl.app.ACTION_SUMMARY_REPORT` и `com.intrtl.app.ACTION_SYNC`.

### Сборка и настройки

- Установите JEDAI Offline **4.23** или новее.
- В `<queries>` укажите пакет `tech.jedai.offline`.
- В мобильных настройках портала (`/api/v1/md/mobile_settings/`) включено on-device распознавание и заполнены списки:
  - `other.offline_models.android` — модели;
  - `other.offline_classes.android` — классы.

Если распознавание выключено или один из списков пуст, `ACTION_SYNC_PALOMNA` завершается ошибкой `ONDEVICE_NOT_AVAILABLE`.

### ACTION_SYNC_PALOMNA

Метод загружает и обновляет модели, классы и справочники для распознавания на устройстве: матрицы, типы матриц и метрики.

Пока модели и классы еще не скачаны, метод загружает их вместе со справочниками. Когда они уже на устройстве, метод проверяет новую версию моделей и классов. 
Если версия на сервере новее, приложение показывает диалог обновления. Подтверждение запускает загрузку. Отказ возвращает `ONDEVICE_USER_CANCELED_UPDATE`.

Справочники обновляются не чаще одного раза в календарные сутки по UTC. Повторный вызов в те же сутки проверяет модели и классы, а матрицы и метрики, уже обновленные сегодня, пропускает. Вызывайте метод минимум раз в сутки, пока есть сеть.

На Wi-Fi загрузка начинается сразу. На мобильной сети приложение спрашивает подтверждение. Отказ возвращает `ONDEVICE_USER_CANCELED_WITH_MOBILE`.

Если загрузка обрывается, приложение предлагает повторить. Отказ возвращает `ONDEVICE_DOWNLOAD_FAILED`.

Успех: `resultCode == RESULT_OK`, в extras есть `action`, файла отчета нет.

### Выбор торговых точек

Параметры `external_ids` и `store_ids` необязательны. Пустой список обрабатывается так же, как отсутствие параметра. Приложение применяет первое подходящее правило:

1. В `store_ids` один идентификатор — матрицы для 1000 точек, ближайших к координатам этой точки.
2. В `external_ids` один идентификатор — матрицы для 1000 точек, ближайших к координатам этой точки. Это правило срабатывает и когда в `store_ids` несколько идентификаторов.
3. В `store_ids` несколько идентификаторов — матрицы только для этих точек.
4. В `external_ids` несколько идентификаторов — матрицы только для этих точек.
5. Оба списка пустые — 1000 ближайших к текущей геолокации. Если геолокации нет — первые 1000 точек справочника.

Если переданы и один `store_id`, и любой `external_ids`, используется `store_id`. Если переданы несколько `store_ids` и несколько `external_ids`, используются `store_ids`. Если переданы несколько `store_ids` и один `external_id`, используется `external_id`.

Точка без координат или отсутствующая в справочнике дает ошибку загрузки. После отказа от повтора приходит `ONDEVICE_DOWNLOAD_FAILED`. Текст `message`: `Store with id <id> not found` или `Store with externalId <id> not found`.

### Ошибки ACTION_SYNC_PALOMNA

Ошибки приходят при `resultCode == RESULT_CANCELED` в extras `error` и `message`.

| `error` | `message` | Когда |
| --- | --- | --- |
| `ONDEVICE_USER_CANCELED_WITH_MOBILE` | `user don't wont download via mobile network` | Пользователь отказался качать по мобильной сети |
| `ONDEVICE_DOWNLOAD_FAILED` | `download failed` или текст исключения | Загрузка не удалась, пользователь отказался повторить |
| `ONDEVICE_USER_CANCELED_UPDATE` | `user canceled update` | Пользователь отказался обновлять модели и классы |
| `ONDEVICE_NOT_AVAILABLE` | `incorrect app or no on-device data in settings` | Сборка без on-device, распознавание выключено или в настройках нет моделей либо классов |

### Поля отчета и ActivityResult

С версии **4.23** файл отчета и extras успешного ответа содержат источник данных. Extras `code`, `codeInt`, `source` и `message` копируются из `report.result`. Они есть у `ACTION_REPORT` и у ответа, который приложение отдает при выходе из `ACTION_VISIT` и `ACTION_SUMMARY_REPORT`.

`source`:

- `on-device` — в визите есть хотя бы одно фото в пайплайне Palomna, которое сервер еще не пересчитал.
- `online` — таких фото нет.

`visit_stats.photo.completed_on_device` — число фото обработанных on-device 
`completed_online` — число фото обработанных онлайн. 
Блок `visit_stats` есть при `RESULT_OK`, `RESULT_INPROGRESS` и `RESULT_INPROGRESS_OFFLINE`. При `RESULT_EMPTY` блока нет.

Виджет для `on-device` берется из on-device виджета визита, для `online` — из серверного.

| `code` | `codeInt` | Когда                                                                               |
| --- | --- |-------------------------------------------------------------------------------------|
| `RESULT_EMPTY` | `2` | Нет валидных фото и нет ответов на вопросы                                          |
| `RESULT_INPROGRESS_OFFLINE` | `37` | Обработка не закончена, устройство офлайн и Palomna не активирована                 |
| `RESULT_INPROGRESS` | `16` | Обработка не закончена в остальных случаях, в том числе офлайн при активной Palomna |
| `RESULT_OK` | `1` | Все фото обработаны по правилу из `source` и по визиту есть виджет этого источника  |


| `error` | `message` | Когда |
| --- | --- | --- |
| `ONDEVICE_DATA_NOT_LOADED` | `models or classes not loaded` | Нет сети, on-device включен в настройках, модели не скачаны, в визите есть необработанные фото |

### Работа без интернета

`ACTION_VISIT` открывает тот же экран визита, что и JEDAI Offline. Если модели и классы скачаны, при отсутствии сети Palomna включается сама, и фото обрабатываются на устройстве. Если модели не скачаны, визит открывается, а при сборке отчета приходит `ONDEVICE_DATA_NOT_LOADED`.

`ACTION_SUMMARY_REPORT` открывает тот же сводный отчет. Визит уже есть на устройстве — экран открывается без сети и показывает данные on-device, когда Palomna активна. Визита на устройстве нет и сети нет — исторический визит не скачивается, ответ `RESULT_CANCELED`, `message` равен `No visit/Offline.`

### Рекомендуемый порядок вызовов

Сначала подготовить данные на устройстве, затем снимать визит. Вход в учетную запись выполняется внутри вызова.

1. Установите JEDAI Offline **4.23+** и объявите пакет `tech.jedai.offline` в `<queries>`.
2. Пока есть сеть, минимум раз в сутки вызовите `com.ailet.ACTION_SYNC_PALOMNA` и передайте точки маршрута. На Wi-Fi загрузка начинается сразу.
3. Вызовите `ACTION_VISIT` и сделайте фото. После успешного `ACTION_SYNC_PALOMNA` съемка работает без интернета.
4. Прочитайте extras `code`, `codeInt`, `source`, `message` и файл отчета. Либо дождитесь broadcast и прочитайте тот же JSON из extra `result`.
5. Чтобы открыть отчет еще раз, вызовите `ACTION_REPORT` или `ACTION_SUMMARY_REPORT`.

`ACTION_SYNC` отправляет уже снятые фото и забирает серверный результат. Загрузку моделей он не выполняет.

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

Для JEDAI Offline:

```xml
<queries>
    <package android:name="tech.jedai.offline" />
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
    <intent>
        <action android:name="com.ailet.ACTION_SYNC_PALOMNA" />
    </intent>
</queries>
```

Разрешение `QUERY_ALL_PACKAGES` открывает список всех установленных приложений. Google Play принимает его только для узкого набора сценариев, поэтому для интеграции с JEDAI используйте `<queries>`.
