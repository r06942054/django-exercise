# django-exercise

開發環境: VS code, python 3.6.9, django 2.2.5

## Part 0

建立虛擬環境

```
$ conda create --name django-exercise python=3.6
```
安裝Django
```
$ pip install Django
```

# Part 1
* 確認版本是2.2

    python -m django --version

* 開資料夾用來放code，執行完下面指令會在資料夾下建立一個mysite資料夾，裡面是Django的一些database configuration, instance, Django-specific options and application-specific settings.
    ```
    $ mkdir django-exercise
    $ django-admin startproject mysite
    ```
    不要取與python內建模組相同的名稱，會出error

* 驗證Django project是work的
    
    cd django-exercise\mysite

    python manage.py runserver

    然後打開網頁到http://127.0.0.1:8000/

* 創建Polls app

    python manage.py startapp polls

* 新增第一個view
    
    打開polls/views.py
    ```
    from django.http import HttpResponse


    def index(request):
        return HttpResponse("Hello, world. You're at the polls index.")
    ```
    create a file called urls.py
    ```
    from django.urls import path

    from . import views

    urlpatterns = [
        path('', views.index, name='index'),
    ]
    ```
    將 root URLconf 指到polls.urls module
    In mysite/urls.py
    ```
    from django.contrib import admin
    from django.urls import include, path

    urlpatterns = [
        path('polls/', include('polls.urls')),
        path('admin/', admin.site.urls),
    ]
    ```
    執行server，然後到http://localhost:8000/polls/ 看結果，Hello, world. You’re at the polls index最在頁面左上角
    ```
    $ python manage.py runserver
    ```
# Part 2

* Database setup: 
    
    settings.py可以設定database的engine、時區、安裝的app等等

    執行migrate，其會根據settings的 INSTALLED_APPS建立任何需要用到的database tables

    $ python manage.py migrate

* 建立models

    Now we’ll define your models – essentially, your database layout, with additional metadata.

    Edit the polls/models.py

    ```
    from django.db import models

    class Question(models.Model):
        question_text = models.CharField(max_length=200) #變數命名就是column的名稱
        pub_date = models.DateTimeField('date published')

    class Choice(models.Model):
        question = models.ForeignKey(Question, on_delete=models.CASCADE) # 刪除Question的時候，Choice也跟著刪除
        choice_text = models.CharField(max_length=200)
        votes = models.IntegerField(default=0)
    ```

* Activating models

    上點的model可以讓Django得到許多資訊，Django能夠:
    建立這個app的database schema
    為Question和Choice建立python database-access的API

    但必須先讓project知道polls已經被安裝

    需修改mysite/settings.py，在INSTALLED_APPS內新增'polls.apps.PollsConfig'，指向polls/apps.py的PollsConfig 這類別

    python manage.py makemigrations polls

    makemigrations可以讓Django知道你做了些改變，可以到polls/migrations/0001_initial.py去看目前的migrations

    python manage.py migrate

    用這指令to apply those changes to the database.

    remember the three-step guide to making model changes:

    * Change your models (in models.py).
    * Run python manage.py makemigrations to create migrations for those changes
    * Run python manage.py migrate to apply those changes to the database.

* Playing with the API

    python manage.py shell

    ```
    >>> from polls.models import Choice, Question  # Import the model classes we just wrote.

    # No questions are in the system yet.
    >>> Question.objects.all()
    <QuerySet []>

    # Create a new Question.
    # Support for time zones is enabled in the default settings file, so
    # Django expects a datetime with tzinfo for pub_date. Use timezone.now()
    # instead of datetime.datetime.now() and it will do the right thing.
    >>> from django.utils import timezone
    >>> q = Question(question_text="What's new?", pub_date=timezone.now())

    # Save the object into the database. You have to call save() explicitly.
    >>> q.save()

    # Now it has an ID.
    >>> q.id
    1

    # Access model field values via Python attributes.
    >>> q.question_text
    "What's new?"
    >>> q.pub_date
    datetime.datetime(2012, 2, 26, 13, 0, 0, 775217, tzinfo=<UTC>)

    # Change values by changing the attributes, then calling save().
    >>> q.question_text = "What's up?"
    >>> q.save()

    # objects.all() displays all the questions in the database.
    >>> Question.objects.all()
    <QuerySet [<Question: Question object (1)>]>
    ```

    adding a `__str__()` method to both Question and Choice in the polls/models.py file可以讓回傳的QuerySet變的好讀取

    <QuerySet [<Question: What's up?>, <Question: What's up?>]>

    加一個客製化的method到Question
    ```

    import datetime

    from django.db import models
    from django.utils import timezone


    class Question(models.Model):
        # ...
        def was_published_recently(self):
            return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

    ```

    ```

    >>> from polls.models import Choice, Question

    # Make sure our __str__() addition worked.
    >>> Question.objects.all()
    <QuerySet [<Question: What's up?>]>

    # Django provides a rich database lookup API that's entirely driven by
    # keyword arguments.
    >>> Question.objects.filter(id=1)
    <QuerySet [<Question: What's up?>]>
    >>> Question.objects.filter(question_text__startswith='What')
    <QuerySet [<Question: What's up?>]>

    # Get the question that was published this year.
    >>> from django.utils import timezone
    >>> current_year = timezone.now().year
    >>> Question.objects.get(pub_date__year=current_year)
    <Question: What's up?>

    # Request an ID that doesn't exist, this will raise an exception.
    >>> Question.objects.get(id=2)
    Traceback (most recent call last):
        ...
    DoesNotExist: Question matching query does not exist.

    # Lookup by a primary key is the most common case, so Django provides a
    # shortcut for primary-key exact lookups.
    # The following is identical to Question.objects.get(id=1).
    >>> Question.objects.get(pk=1)
    <Question: What's up?>

    # Make sure our custom method worked.
    >>> q = Question.objects.get(pk=1)
    >>> q.was_published_recently()
    True

    # Give the Question a couple of Choices. The create call constructs a new
    # Choice object, does the INSERT statement, adds the choice to the set
    # of available choices and returns the new Choice object. Django creates
    # a set to hold the "other side" of a ForeignKey relation
    # (e.g. a question's choice) which can be accessed via the API.
    >>> q = Question.objects.get(pk=1)

    # Display any choices from the related object set -- none so far.
    >>> q.choice_set.all()
    <QuerySet []>

    # Create three choices.
    >>> q.choice_set.create(choice_text='Not much', votes=0)
    <Choice: Not much>
    >>> q.choice_set.create(choice_text='The sky', votes=0)
    <Choice: The sky>
    >>> c = q.choice_set.create(choice_text='Just hacking again', votes=0)

    # Choice objects have API access to their related Question objects.
    >>> c.question
    <Question: What's up?>

    # And vice versa: Question objects get access to Choice objects.
    >>> q.choice_set.all()
    <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
    >>> q.choice_set.count()
    3

    # The API automatically follows relationships as far as you need.
    # Use double underscores to separate relationships.
    # This works as many levels deep as you want; there's no limit.
    # Find all Choices for any question whose pub_date is in this year
    # (reusing the 'current_year' variable we created above).
    >>> Choice.objects.filter(question__pub_date__year=current_year)
    <QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>

    # Let's delete one of the choices. Use delete() for that.
    >>> c = q.choice_set.filter(choice_text__startswith='Just hacking')
    >>> c.delete()

    ```
    
    
    
    For more information on model relations, see [Accessing related objects](https://docs.djangoproject.com/en/2.2/ref/models/relations/). 
    
    For more on how to use double underscores to perform field lookups via the API, see [Field lookups](https://docs.djangoproject.com/en/2.2/topics/db/queries/#field-lookups-intro). For full details on the database API, see our [Database API reference](https://docs.djangoproject.com/en/2.2/topics/db/queries/).


* Introducing the Django Admin

    python manage.py createsuperuser

    然後啟動server

    python manage.py runserver

     http://127.0.0.1:8000/admin/

* Make the poll app modifiable in the admin
    
    讓admin頁面可以修改poos的app

    polls/admin.py

    ```
    from django.contrib import admin

    from .models import Question

    admin.site.register(Question)
    ```

* Explore the free admin functionality

    頁面可以做一些針對polls的動作，同API操作


* test

Reference:

https://docs.djangoproject.com/en/2.2/intro/tutorial01/

<https://openhome.cc/Gossip/CodeData/PythonTutorial/DjangoPy3.html>
