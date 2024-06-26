# 1. Integration with intents

- [1. Integration with intents](#1-интеграция-через-интенты)
  - [1.1. Method call (visit example)](#11-вызов-метода-на-примере-метода-visit)
  - [1.2 Methods](#12-методы)
    - [1.2.1 Method Visit](#121-метод-visit)
    - [1.2.2 Method Report](#122-метод-report)
    - [1.2.3 Method Summary Report](#123-метод-summary-report)
    - [1.2.4 Method Sync](#124-метод-sync)
  - [1.3 Broadcast message](#13-широковещательное-broadcast-сообщение)
  - [1.4 Example reports](#14-примеры-отчетов)

## 1.1. Method call (visit example)

Action is specified in the format ```com.ailet.[method]```

```kotlin
private fun visit() {
    Intent().apply {
        action = "com.ailet.ACTION_VISIT"
        flags = 0
        putExtra("login", "логин")
        putExtra("password", "пароль")
        putExtra("id", "ИД пользователя")
        putExtra("visit_id", "ИД визита")
        putExtra("store_id", "ИД ТТ")
        
        startActivityForResult(this, VISIT_RESULT)
    }
}

override fun onActivityResult(requestCode: Int, resultCode: Int, intent: Intent?) {
    super.onActivityResult(requestCode, resultCode, intent)
        
    when (resultCode) {
        RESULT_OK -> {
            intent?.data?.let { uri ->
                val result = readFromUri(uri)
                try {
                    val json = JSONObject(result)
                } catch (t: Throwable) {
                    t.printStackTrace()
                }
            }
         }

        else -> {
            intent?.extras?.get("error")?.let { log("error: $it") }
            intent?.extras?.get("message")?.let { log("error message: $it") }
        }
    }
}

```
```Intent.extras``` contains parameters:

Parameter | Type | Description 
---------|-----|----------
action | String | Method
error | String | Error type (if it was)
message | String | Error description (if it was)

```Intent.data``` contains ```uri``` file report ([Example report](method_result.json)) for methods ```visit, report, summaryReport```.


**Error types**

Type | Description 
---------|-----
ERROR | Error while method execute
AUTH | Authentication error
INCORRECT_INPUT | Incorrect input data

## 1.2 Methods

Method  | Description
--- | ---
[ACTION_VISIT](#121-метод-visit) | Start/edit visit 
[ACTION_REPORT](#122-метод-report) | Recieve visit repot
[ACTION_SUMMARY_REPORT ](#123-метод-summary-report) | Visit summary report
[ACTION_SYNC](#124-метод-sync) | Launch synchronization

### 1.2.1 Visit method

Method launch photo shooting while visit.

Parameter | Type | Description | Necessary 
---------|-----|----------|:-:
login           |String      | User login in Ailet system      | + | 
password        |String      | User password in Ailet system     | + | 
id  |String      | User id 
visit_id         |String      | Visit id        | + | 
store_id         |String      | Store id        | + | 
task_id       |String      | External task id         | |  

### 1.2.2 Report method

Method recieve visit repot ([Example report](method_result.json)).

Parameter | Type | Description | Necessary 
---------|-----|----------|:-:
login           |String      | User login in Ailet system      | + | 
password        |String      | User password in Ailet system     | + | 
id  |String      | User id 
visit_id         |String      | Visit id        | + | 
store_id         |String      | Store id        | + | 
task_id       |String      | External task id         | |  

### 1.2.3 Summary Report method

Launch window summary report of visit.

Parameter | Type | Description | Necessary 
---------|-----|----------|:-:
login           |String      | User login in Ailet system      | + | 
password        |String      | User password in Ailet system     | + | 
id  |String      | User id 
visit_id         |String      | Visit id        | + | 
store_id         |String      | Store id        | + | 
task_id       |String      | External task id         | |  

### 1.2.4 Sync method

Method launch synchronization.

Parameter | Type | Description | Necessary
---------|-----|----------|:-:
login           |String      | User login in Ailet system     | + | 
password        |String      | User password in Ailet system     | + | 
id  |String      | User id 

Method returns only resultCode, if RESULT_OK - data for synchronize exists and synchronization service launched, else, for example RESULT_CANCELED - there is no data to synchronize.

## 1.3 Broadcast message 

При получении всех данных по визту приложение Ailet генерирует широковещательное сообщение с ```intent.action = com.ailet.app.BROADCAST_WIDGETS_RECEIVED``` (or ```com.ailet.russia.BROADCAST_WIDGETS_RECEIVED```).

**Proccesing message example**

```kotlin
broadcastReceiver = object : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        parseBroadcaseMesasge(intent)
    }
}

registerReceiver(
    broadcastReceiver,
    IntentFilter(IR_BROADCAST_V3)
)
...

private const val NOT_SET = "not set"
private const val VISIT_ID = "visit_id"
private const val INTERNAL_VISIT_ID = "internal_visit_id"
private const val STORE_ID = "store_id"
private const val TASK_ID = "task_id"
private const val TOTAL_PHOTOS = "total_photos"
private const val COMPLETED_PHOTOS = "completed_photos"
private const val RESULT = "result"

private fun parseBroadcaseMesasge(intent: Intent) {
    val extras = intent.extras
    val visitId = extras?.getString(VISIT_ID, NOT_SET)    
    val internalVisitId = extras?.getString(INTERNAL_VISIT_ID, NOT_SET)    
    val storeId = extras?.getString(STORE_ID, NOT_SET)
    val taskId = extras?.getString(TASK_ID, NOT_SET)
    val totalPhotos = extras?.getString(TOTAL_PHOTOS, NOT_SET)
    val completedPhotos = extras?.getString(COMPLETED_PHOTOS, NOT_SET)
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

**Intent extras**

Parameter | Type | Description 
---------|-----|----------
internal_visit_id           |String      | Internal (Ailet) visit id
visit_id           |String      | Visit id
store_id           |String      | Store id
user_id           |String      | User id (Ailet)
total_photos           |Int      | Count visit photos
completed_photos           |Int      | Count completed photos
result           | String | Uri report file ([Example report](broadcast_result.json))

## 1.4 Example reports
[Example report returned by methods (except ACTION_SYNC method)](method_result.json)
[Example report returned in broadcast message](broadcast_result.json)
