# ContentProvider
Специальный "сервис", выделенный фреймворком android в отдельную категорию `provider`, который позволяет обращаться к заранее описанным данным, посредством URI ([Uniform Resource Identifier](https://ru.wikipedia.org/wiki/URI)) из разных приложений совместно.
Как правило **ContentProvider** используется в паре с базой данных например на **SQLite** (в моем текущем проекте я храню в SQLite кешированные данные запросов к серверу)

В статье "[Создание поставщика контента](https://developer.android.com/guide/topics/providers/content-provider-creating?hl=ru)" от Google Developers рекомендуется не использовать ContentProvider в ряде случаев:
>Вам не нужен поставщик для работы с базой данных SQLite, если ее планируется использовать исключительно в вашем приложении.

Итак, чтобы использовать **ContentProvider** нам необходимо определиться с его идентификатором и способом хранения данных (да, это не обязательно должна быть БД), т.к. **ContentProvider** штука используемая для взаимодействия разных приложений необходимо обеспечить уникальность его идентификатора, рекомендуется использовать имя пакета. например `com.example.appname.provider`
Далее следует прописать наш provider в файле манифеста
```xml
<provider
    android:authorities="com.dogvscat.monday.provider"
    android:name=".service.UsersContentProvider" />
```
Где - `authorities` и есть наш идентификатор.
Т.к. Основой запросов к ресурсам с ипользованием **ContentProvider** является URI вида `content://<authorities>/<content_path>`, нам необходимо продумать как мы назовем наши ресуры (в моем случае - таблицы БД), в строке URI имя таблицы `<content_path>`. Желательно сохранить `authorities` и имена таблиц в статические константы для удобства использования. 

Затем, нужно создать класс реализующий интерфейс базового класса ContentProvider и реализовать его методы **query()**, **insert()**, **update()**, **delete()**, **getType()**, **onCreate()** ([вот неплохая статья по созданию contentProvider на Java](http://developer.alexanderklimov.ru/android/theory/contentprovider.php)).
Все методы кроме **onCreate()** принимают на вход в качестве одного из параметров URI элемент. 

В процессе работы программы возможны разные сценарии использования данных, как правило нам необходимо работать или с конкретной записью или со всеми данными таблицы целиком, чтоб ContentProvider мог различить, какие данные ему передавать был придуман класс **UriMatcher** 
## UriMatcher
Он нужен чтобы "распарсить" наш URI на части и вернуть один из возможных сценариев использования (целочисленную константу), на примере кода это будет выглядеть следующим образом
```Kotlin
/**
 * Пример на Kotlin
 */
//Объявляем константы которые должен вернуть UriMatcher
 val URI_USERS = 1
 val URI_USERS_ID = 2
 val uriMatcher: UriMatcher = UriMatcher(UriMatcher.NO_MATCH)
    init {
//Здесь USER_PATH - имя таблицы
        uriMatcher.addURI(AUTHORITY, USER_PATH, URI_USERS)
        uriMatcher.addURI(AUTHORITY, "$USER_PATH/#", URI_USERS_ID)
    }
```
В примере указанном выше используется символ **#** - это символ подствновки указывающий, что на его месте возможно любое целое число, также допускется использовать символ **"*"**, который укажет на любую последовательность символов (как в регулярном выражении)
>Метод addURI() сопоставляет идентификатор **ContentProvider** и имя талицы с целочисленным значением. 

Использовать UriMatcher необходимо в реализациях методов **ContentProvider** для нашего **CustomContentProvider**, где в зависимости от передаваемого URI, UriMatcher вернет константу (*Метод match() возвращает целочисленное значение для URI*) и нам останется включить switch/case и обратотать разные сценарии

## onCreate()
В этом методе нам необходимо проинициализировать наше хранилище данных (в моем случае создать БД). 
```kotlin
override fun onCreate(): Boolean {
        dbHelper = DBHelper(context)
        return true
    }

 private class DBHelperCP(context: Context?) : SQLiteOpenHelper(context, DATABASE_NAME, null, DATABASE_VERSION) {
        override fun onCreate(db: SQLiteDatabase?) {
            //TABLE_CREATE - SQLite скрипт создающий БД
            db!!.execSQL(TABLE_CREATE)
        }

        override fun onUpgrade(db: SQLiteDatabase?, oldVersion: Int, newVersion: Int) {
            //Здесь указывается код обнавляющий архитектуру БД
        }

    }
```
>Система Android вызывает этот метод сразу после создания вашего поставщика.

>Избегайте слишком длинных операций в методе onCreate(). Отложите выполнение задач инициализации до тех пор, пока они не потребуются. Дополнительные сведения об этом представлены в разделе [Реализация метода onCreate](https://developer.android.com/guide/topics/providers/content-provider-creating?hl=ru#OnCreate).

## query
Запрос в БД. Возвращает данные в виде объекта Cursor.

## insert
Вставка записи в БД. Возвращает URI контента для новой вставленной строки.

## delete
Удаление записей из БД. Возвращает количество удаленных строк.

## update 
Изменение записей в БД. Возвращает количество обновленных строк.

## getType
Возвращает MIME тип данных - нужно чтобы дать понять другим приложениям данные какого типа возвращает наш **ContentProvider** - если приложение возвращает какие то специфические данные, не советую с этим заморачиваться, если у тебя возникла потребность использовать MIME типы то ты и так знаешь что делаешь😁 [статья про MIME типы в ContentProvider](https://developer.android.com/guide/topics/providers/content-provider-creating?hl=ru#MIMETypes)

```kotlin
//константы определяющие MIME типы
companion object {
    const val USER_CONTENT_TYPE = "vnd.android.cursor.dir/vnd.$AUTHORITY.$PATH"
    const val USER_CONTENT_ITEM_TYPE = "vnd.android.cursor.item/vnd.$AUTHORITY.$PATH"
}
```

## ContentProvider для нескольких таблиц
Разумеется мы можем использовать **ContentProvider** более чем для 1 таблицы ([Вот отличная статья о том как это сделать на StackOverflow](https://stackoverflow.com/questions/3814005/best-practices-for-exposing-multiple-tables-using-content-providers-in-android))

Для этого нам необходимо в нашем классе ***MyContentProvider*** определить URI обоих таблиц

```kotlin
val USER_CONTENT_URI: Uri = Uri.parse("content://$AUTHORITY/$PATH_1")
val USER_CONTENT_URI: Uri = Uri.parse("content://$AUTHORITY/$PATH_2")
```
а также указать все возможные сценарии (константы) для **UriMatcher**
```kotlin
private val URI_PATH_1 = 101
private val URI_PATH_1_ID = 102
private val URI_PATH_2 = 201
private val URI_PATH_2_ID = 202
private val uriMatcher: UriMatcher = UriMatcher(UriMatcher.NO_MATCH)
    init {
        uriMatcher.addURI(AUTHORITY, PATH_1, URI_PATH_1)
        uriMatcher.addURI(AUTHORITY, "$PATH_1/#", URI_PATH_1_ID)
        uriMatcher.addURI(AUTHORITY, PATH_2, URI_PATH_2)
        uriMatcher.addURI(AUTHORITY, "$PATH_2/#", URI_PATH_2_ID)
    }
```
И не забыть прописать все switch/case для разных сценариев

### PS: Будь акуратней с использованием `NotNull` переменных в `Kotlin`, я потратил около 6 часов на поиск ошибки в query **ContentProvider** оказалось дело было в NotNull переменной 