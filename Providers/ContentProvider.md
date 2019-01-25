# ContentProvider
Специальный "сервис", выделенный фреймворком android в отдельную категорию `provider`, который позволяет обращаться к заранее описанным данным, посредством URI ([Uniform Resource Identifier](https://ru.wikipedia.org/wiki/URI)) из разных приложений совместно.
Как правило **ContentProvider** используется в паре с базой данных например на **SQLite** (в моем текущем проекте я храню в SQLite кешированные данные запросов к серверу)

В статье "[Создание поставщика контента](https://developer.android.com/guide/topics/providers/content-provider-creating?hl=ru)" от Google Developers рекомендуется не использовать ContentProvider в ряде случаев:
>Вам не нужен поставщик для работы с базой данных SQLite, если ее планируется использовать исключительно в вашем приложении.

Итак чтобы использовать ContentProvider нам необходимо определиться с его идентификатором и способом хранения данных (да, это не обязательно должна быть БД), т.к. ContentProvider штука используемая для взаимодействия разных приложений необходимо обеспечить уникальность его идентификатора, рекомендуется использовать имя пакета. например `com.example.appname.provider`
Далее следует прописать наш provider в файле манифеста
```xml
<provider
    android:authorities="com.dogvscat.monday.provider"
    android:name=".service.UsersContentProvider" />
```
