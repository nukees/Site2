## __Virtual environment__ ##
___workon___ - список созданных виртуальных сред

___workon DjangoENV___ - активация выбранной виртуальной среды (в данном случае DjangoENV)

___deactivate___ - деактивация виртуальных сред, возврат к базовой среде

## __Django__ ##
Информация для изучения получена из книги "Django 2.1 Практика создания веб-сайтов на Python", автор Дронов В.А.

### 1. Создание проекта сайта ###
Создаем проект сайта командой: 

___django-admin startproject site-name___, где

___site-name___ - это название сайта/проекта, который мы создаем.
В результате выполнения команды будет создана структура папок и файлов
___site-name___(внешняя корневая папка(1)) -> ___site-name___(внутренняя конфигурационная папка (2), которая содержит 
различные конфигурационные файлы) и файл управления ___manage.py___

```
|--- site-name(1)
         |---site-name(2)
         |       |- __init__.py
         |       |- settings.py
         |       |- urls.py
         |       |- wsgi.py
         |- manage.py
```

### 2. Запуск отладочного веб-сервера ###

___manage.py runserver___

Запускает отладочные сервер на локальной машине и работает по 8000 порту.
_http://localhost:8000_ или _http://127.0.0.1:8000_

### 3. Состав сайта ###
Сайт состоит из одного или нескольких пакетов/приложений/элементов (???) сайта. Далее будет использован термин 
___приложение___. Приложение отвечает а работу сайта или его определенной части (например, если создать приложение 
forum и в него вынести форумную часть, независящую от сайта, рассылка и т.д.). В сайте должно быть как минимум одно 
приложение (имхо от 26/12/2019, точно пока не выяснил). Приложение создается командой:

___manage.py startapp app-name___, где ___app-name___ это название приложения

При добавление приложения, необходимо внести информацию о данном приложении в конфигурационный файл _settings.py_.
Строка добавляется в раздел _INSTALLED_APS_. Пример добавления ниже.
```
INSTALLED_APPS = [
    ...
    'bboard.apps.BboardConfig'
]
```
### 4. Контроллеры ###
Что-бы в приложении что-то работало создаются контроллеры. Контроллер - это код, запускаемый в ответ на поступление 
клиентского запроса на веб-сервер. В них выполняются все действия по подготовке данных для вывода, равно как и 
обработка данных.

Добавляем в _views_.py приложения контроллер. Обязательно добавляем библиотеку для работы функции контроллера. 
Например:
```
# Добавляем библиотеку
from django.http.import HttpResponse 

# Добавляем контроллер-функцию
def index(request):
    return HttpResponse('Текст')
```

Кроме этого необходимо в конфигурационном файле _urls.py_ добавить пути и ссылку на функцию
```
# Указываем где искать функцию
from bboard.views import index

# Добавляем путь
urlpatterns = [
    path('bboard/', index),
    ....
]
```

Основной вариант прописания путей(маршрутов) контроллеров, это создание в папке приложения файла _urls.py_ с кодом, 
практически копирующим код основного конфигурационного файла _urls.py_
```
# Используем базовую библиотеку
from django.urls import path

# Импортируем контролллер из views.py
from .views import index

# Добавляем пути для контроллеров приложения
urlpatterns = [
    path('', index),
]

```
В конфигурационный файл _urls.py_ тоже вносим необходимые корректировки
```
from django.contrib import admin

# Импортируем из библиотеки дополнительную функцию include
from django.urls import path, include

# Добавляем ссылку на файл urls приложения
urlpatterns = [
    path('bboard/', include('bboard.urls')),
    path('admin/', admin.site.urls),
]
```

### 5. Модели данных и миграции ###
Создаются в файле _models.py_, расположенного в папке приложения
```
# Создание модели объявления
from django.db import models

class Bb(models.Model):
    title = models.CharField(max_length=50)
    content = models.TextField(null=True, bank=True)
    price = models.FloatField(null=True, bank=True)
    published = mnodels.DateTimeField(auto_now_add=True, db_index=True)
```

Для сохранения структуры соданных данных используются функции миграции.
_manage.py makemigrations bboard_

Результурющий SQL-код миграции (для просмотра)
_manage.py sqlmigrate bboard 0001_

Выполнение миграции для всех приложений проекта
_manage.py migrate_

Запуск консоли Django
_manage.py shell_

Пропускаем ненужное...
Дальше идет работа в консоли, создание в консоли моделей и т.д.

Создание взаимосвязанных моделей. Модель Bb зависит от модели Rubric. Связываются записью
rubric = models.ForeignKey('Rubric', null=True, on_delete=models.PROTECT, verbose_name='Рубрика')
```
class Bb(models.Model):
    title = models.CharField(max_length = 50, verbose_name='Товар')
    content = models.TextField(null = True, blank = True, verbose_name='Описание')
    price = models.FloatField(null = True, blank = True, verbose_name='Цена')
    published = models.DateTimeField(auto_now_add = True, db_index = True, verbose_name='Опубликовано')
    rubric = models.ForeignKey('Rubric', null=True, on_delete=models.PROTECT, verbose_name='Рубрика')

    class Meta:
        verbose_name_plural = 'Объявления'
        verbose_name='Объявление'
        ordering = ['-published']

class Rubric(models.Model):
    name = models.CharField(max_length = 20, db_index=True, verbose_name = 'Название')

    def __str__(self):
        return self.name

    class Meta:
        verbose_name_plural = 'Рубрики'
        verbose_name = 'Рубрика'
        ordering = ['name']
```

### 6. Шаблоны ###
(ВАЖНО!) Для шаблонов специально в папке приложения создается папка _Templates_
Рекомендуется в папке _templates_ создать подпапку с названием приложения и в ней хранить шаблоны страниц приложения, 
т.к. шаблоны в основной папке будут использоватся для специфических нужд (надо проверить, не факт что оно так)
Остальная информация второстепенна. Нужно создать шаблон, внести в views.py изменения, чтобы использовались шаблоны и 
т.д. (см книгу)

### 7. Адимнистративные функции и настройка страницы администратора ###
Задать суперпользователя/админа/root

_manage.py createsuperuser_

После ввода команды добавляем пользователя. Логин, почта. пароли. Все как обычно.

Для доступа из админовского режима к моделям и данным приложений, надо внести корректировки в файл _admin.py_, который 
находится в папке приложения Называется регистрация приложения в списке административного сайта.

```
from django.contrib import admin
from .models import Bb

admin.site.register(Bb) # Регистрация модели данных
```

Параметры полей и моделей
Для отображения названия полей модели необходимо добавить в код модели описание полей путем ключа 
__verbose_name = 'Description text'__. Пример кода ниже.
```
content = models.TextField(null = True, blank = True, verbose_name='Описание')
```
Так же создается класс Meta: для описания самой модели. Важно, минус перед полем означает сортировку в обратном 
порядке. Код ниже.
```
ordering = ['-published']
```

Для удобочитаемости моделей в _admin.py_ можно добавить отображение модели на странице администратора. Пример ниже. 
Параметры задаются объектами _tuple_. Если поле всего одно, то запись выглядит примерно так list_diplay = ('name',)
```
class BbAdmin(admin.ModelAdmin):
    list_display = ('title', 'content', 'price', 'published') # Отображаемые поля
    list_display_links = ('title', 'content') # Поля, по кторым можно перейти к объекту
    search_fields = ('title', 'content') # Поля по которым осуществляется контекстный поиск

admin.site.register(Bb, BbAdmin)
```

### 8. Параметризированные запросы ###
Очень, очень много нюансов. Нужно быть очень внимательным. Не допускать опечаток, т.к. искать их потом очень 
проблематично. Возможно можно настроить линтер для Django, но пока оно так... Общая взаимосвязь - Добавляем в _urls.py_ 
приложения код, который указывает что параметризированный запрос сссылается на метод (не пропускаем / 
в формируемых путях, без них работать не будет)
```
urlpatterns = [
    path('<int:rubric_id>/', by_rubric), # rubric_id ссылается на метод by_rubric
    path('', index),
]
```
- Добавляем в _views.py_ приложения код, сдержащий метод добавленный выше
```
def by_rubric(request, rubric_id):
    bbs = Bb.objects.filter(rubric=rubric_id)
    rubrics = Rubric.objects.all()
    current_rubric = Rubric.objects.get(pk=rubric_id)
    context = {'bbs':bbs, 'rubrics':rubrics, 'current_rubric':current_rubric}
    return render(request, 'bboard/by_rubric.html', context)
```
- Добавляем шаблон параметризированной html страницы __by_rubric.html__

__Общая связка МОДЕЛЬ-URL-VIEW-TEMPLATE__

__MODEL - добавляем модель данных в models.py__

__URL - создаем связку данных с методом отображающим саму html страницу или ее шаблон в urls.py__

__VIEW - создаем метод отбражения html страницы или ее шаблона в views.py__

__TEMPLATE - в папке _Templates_ создаем шаблон или html страницу__

Можно реализовать именнованые запросы заменив явную ссылку на ее название, что в последующем облегчит редактирование 
или модифицирование сайта
```
# В файле urls.py
urlpatterns = [
    path('<int:rubric_id>/', by_rubric, name = 'by_rubric'), # Добавлено название для ссылки
    path('', index, name = 'index'),
]
...
# В шаблонах страниц (пример)
...
        <div>
            <a href="{% url 'index' %}">Main Page</a> # Именованная ссылка
            {% for rubric in rubrics %}
            <a href="{% url 'by_rubric' rubric.pk %}">{{ rubric.name }}</a>
            {% endfor %}
        </div>
...
```
### 8. Формы для передачи данных в приложение ###




