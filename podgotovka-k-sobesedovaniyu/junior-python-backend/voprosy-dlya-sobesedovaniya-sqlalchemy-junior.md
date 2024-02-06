# Вопросы для собеседования SQLAlchemy junior

1. **Что такое SQLAlchemy?**

[SQLAlchemy](https://ru.wikipedia.org/wiki/SQLAlchemy) — это Python-библиотека для работы с реляционными базами данных с применением технологии ORM. \[1]

<figure><img src="https://lh7-us.googleusercontent.com/tk4axUzdVadlUAbn4G0TbE4o0Tpvyj2Zlf2TXwWZgNueekgJoZC4x_OGtwxmmlPZuqgUoKpqdB-9qWAcyW3B7tdQPgoBjSGiDl3Pt1gbkOBsS4Rr9lI8Wp8yXpa9D8EFUYGM6TkxxpI4Y1zY_c9alR5OLpGaIqxa" alt=""><figcaption></figcaption></figure>

2. **В Python имеются встроенные инструменты для написания сырых SQL-запросов к БД. Для чего тогда использовать сторонние библиотеки?**

* сторонние библиотеки позволяют выполнять запросы к БД на языке Python, не обращая внимания на диалекты каждой СУБД
* предотвращение SQL-инъекций
* замена СУБД без переписывания большого объема кода. Достаточно изменить движок

3. **Что такое ORM и для чего эта технология нужна?**

ORM (Object-Relational Mapping) — это технология, которая позволяет сопоставлять модели, типы которых несовместимы, в нашем случае это таблица базы данных и объект Python. То есть, можно обращаться к объектам классов для управления данными в таблицах БД. Также можно создавать, изменять, удалять, фильтровать и наследовать объекты классов, сопоставленные с таблицами БД. \[2]\[3]

<figure><img src="https://lh7-us.googleusercontent.com/vmpuJ24e3e9Zoj9eVl5b9wgmFVyCRD3CA1LjrJX8NnInSh-nXOenhr8TCiwZwLpZPEZyOE-Zjg_AZ14SAUT2Vv4oyOfvlkixjvIyQfK4xi5Ul39S06lynvOJIpXeG-jzx9iwwQQcO2uJ68uLH_lOFNeXem5fyATE" alt=""><figcaption></figcaption></figure>

4. **Какие еще ORM вам известны и почему именно SQLAlchemy?**

* Peewee, Django ORM
* Преимущества SQLAlchemy:
  * старая и знакомая многим библиотека с большим объемом написанного кода
  * поддерживает asyncio

5\. **В коде приложения вы наткнулись на такой класс. Что можете про него рассказать?**

{% code lineNumbers="true" fullWidth="false" %}
```python
class AsyncDatabaseSession:
    def __init__(self):
        self.engine = create_async_engine(“postgres://user:pass@localhost/dbname”)
        self.session_factory = async_scoped_session(
                             async_sessionmaker(self.engine, expire_on_commit=False),
                             scopefunc=current_task
                            )
    
    @asynccontextmanager
    async def session(self):
        session = self.session_factory()
        try:
            yield session
        except Exception:
            await session.rollback()
        finally:
            await session.close()
```
{% endcode %}

* это класс асинхронной сессии [SQLAlchemy](https://ru.wikipedia.org/wiki/SQLAlchemy) для подключения к БД.
* @asynccontextmanager – декоратор, который возвращает объект асинхронного контекстного менеджера для генераторного объекта. В данном случае генерируются фабрики сессий для подключения к БД. При возникновении исключения все изменения откатываются. По окончании запросов соединение с БД закрывается.
* async\_sessionmaker –  конфигуратор сессии.
* async\_scoped\_session – это хранилище уже созданных сессий, каждая из которых привязана к своему потоку. Если вызвать сконфигурированный экземпляр scoped\_session в новом потоке, он создаст новую сессию. А если потом из этого же потока вызвать scoped\_session во второй раз, он вернёт ту же сессию, а не создаст новую.
* self.session\_factory – объект сессии (фабрика сессий(=транзакций) в рамках одного потока.
* self.engine – интерфейс для работы с конкретной базой данных. Включает драйвер СУБД, точку доступа к БД, логин и пароль пользователя БД и другие настройки.
* session — отдельная транзакция, которая может быть зафиксирована или отменена. Транзакция — это архив для запросов к базе. Она защищает данные по принципу «всё, или ничего». \[4]

6\. **В коде приложения вы наткнулись на такой класс. Что можете про него рассказать?**

<pre class="language-python" data-line-numbers data-full-width="true"><code class="lang-python">class User(Base):
    __tablename__ = "user"
    username: Mapped[str] = mapped_column(String(length=50), unique=True, nullable=False, index=True)
<strong>    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow, server_default=func.now())
</strong>    permissions: Mapped[List["UserPermission"]] = relationship("UserPermission", lazy="selectin")
</code></pre>

* этот класс является отражением таблицы user в базе данных.
* username: Mapped\[str] = mapped\_column – атрибут username соответствует столбцу с таким же именем.
* String(length=50) – столбец может содержать только строковые значения, длина каждой записи не более 50 символов.
* unique=True – повторение записей в одном столбце не допускается.
* nullable=False – пустая запись не допускается.
* index=True – сделать столбец индексируемым для ускорения поиска. Однако индексы замедляют операции добавления, обновления, удаления строк таблицы, поскольку при этом приходится обновлять сами индексы.
* default – запись ячейки в столбце по умолчанию со стороны клиента.
* server\_default – запись ячейки в столбце по умолчанию со стороны сервера.
* foreign\_keys – ссылка на ячейку другой таблицы.
* relationship("UserPermission", lazy="selectin") – связь с таблицей UserPermission.&#x20;
* lazy="selectin" – ленивая загрузка, загружает связанные объекты UserPermission в один запрос с User.

7\. **В коде приложения вы наткнулись на такую функцию. Что можете про неё рассказать?**

<pre class="language-python" data-line-numbers data-full-width="true"><code class="lang-python">async def get_users_with_posts_and_profiles(session: AsyncSession):
<strong>    stmt = (select(User).options(joinedload(User.profile), selectinload(User.posts)).order_by(User.id))
</strong>    users = await session.scalars(stmt)
    return list(users)

</code></pre>

*   эта функция возвращает список пользователей и загружает связанные записи о профилях и постах, используя методы оптимизации запросов (Eager loading):

    * joinedload(User.profile) загружает связанные записи о профилях пользователей через оператор JOIN
    * selectinload(User.posts) генерирует второй оператор SELECT, который загружает связанные дочерние объекты Posts для каждого объекта User



**Список литературы**

1. Официальная документация «SQLAlchemy 2.0 Documentation»:  [https://docs.sqlalchemy.org/en/20/intro.html](https://docs.sqlalchemy.org/en/20/intro.html)
2. Статья «Крадущийся тигр, затаившийся SQLAlchemy. Основы»: [https://habr.com/ru/articles/470285/](https://habr.com/ru/articles/470285/)
3. Статья «Python: Работа с базой данных, часть 2/2: Используем ORM»: [https://habr.com/ru/articles/322086/](https://habr.com/ru/articles/322086/)
4. Статья «Что такое транзакция»: [https://habr.com/ru/articles/537594/](https://habr.com/ru/articles/537594/)
5. Статья «Правильно работаем с сессиями БД в SQLAlchemy»: https://otus.ru/nest/post/250/
