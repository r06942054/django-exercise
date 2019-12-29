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
安裝及運行簡單app

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
熟悉建model與DM及必備的相關設定

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


# Part 3


Django的URLconfs maps URL patterns to views.
以及一些較快速及較好的的寫法

* Writing more views

    polls/views.py.

    ```
    def detail(request, question_id):
        return HttpResponse("You're looking at question %s." % question_id)

    def results(request, question_id):
        response = "You're looking at the results of question %s."
        return HttpResponse(response % question_id)

    def vote(request, question_id):
        return HttpResponse("You're voting on question %s." % question_id)
    ```

    polls.urls 讓url可以對應到views.py內的view


    ```
    from django.urls import path

    from . import views

    urlpatterns = [
        # ex: /polls/
        path('', views.index, name='index'),
        # ex: /polls/5/
        path('<int:question_id>/', views.detail, name='detail'),
        # ex: /polls/5/results/
        path('<int:question_id>/results/', views.results, name='results'),
        # ex: /polls/5/vote/
        path('<int:question_id>/vote/', views.vote, name='vote'),
    ]
    ```


* Write views that actually do something

Each view is responsible for doing one of two things: returning an HttpResponse object containing the content for the requested page, or raising an exception such as Http404. The rest is up to you.

polls/views.py
```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# Leave the rest of the views (detail, results, vote) unchanged
```

但上面的結果很難看，因此若要設計版面，可透過template設計

polls/templates/polls/index.html

```
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

polls/views.py

```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))
```

* A shortcut: render()

render會直接回傳HttpResponse

```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

* Raising a 404 error

如果question_id不存在，就raise 404

```
from django.http import Http404
from django.shortcuts import render

from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, 'polls/detail.html', {'question': question})
```

polls/templates/polls/detail.html

```{{ question }}```

* A shortcut: get_object_or_404()

polls/views.py¶

```
from django.shortcuts import get_object_or_404, render

from .models import Question
# ...
def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/detail.html', {'question': question})
```


* Use the template system

原本url在polls/index.html的寫法

```
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

若是使用原本的url寫法，當要修改url的格式時，可能會需要修改大量的template檔案，因此改為

```
<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
```

其中'detail'是定義在polls.urls module，可直接修改polls.urls內的格式為polls/specifics/12/

```
# added the word 'specifics'
path('specifics/<int:question_id>/', views.detail, name='detail'),
```


* Namespacing URL names

當有多數個app時，DJ會無法分辨url，因此必須在urls.py新增app_name = 'polls'在urlpatterns之前

並將template的html檔(polls/templates/polls/index.html)的url寫法改為

```
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```

# Part 4

* Write a simple form

polls/templates/polls/detail.html

```htmlmixed=
<h1>{{ question.question_text }}</h1>

{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

<form action="{% url 'polls:vote' question.id %}" method="post">
{% csrf_token %}
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br>
{% endfor %}
<input type="submit" value="Vote">
</form>
```

Whenever you create a form that alters data server-side, use method="post"

[GET vs. POST](https://blog.toright.com/posts/1203/%E6%B7%BA%E8%AB%87-http-method%EF%BC%9A%E8%A1%A8%E5%96%AE%E4%B8%AD%E7%9A%84-get-%E8%88%87-post-%E6%9C%89%E4%BB%80%E9%BA%BC%E5%B7%AE%E5%88%A5%EF%BC%9F.html)

當在post一個form的時候，要小心[Cross Site Request Forgeries](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0)，should use the {% csrf_token %} template tag

polls/views.py

```python=
from django.http import HttpResponse, HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse

from .models import Choice, Question
# ...
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

使用reverse function回傳url string`/polls/3/results/`，方便coding

polls/views.py

```
from django.shortcuts import get_object_or_404, render


def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, 'polls/results.html', {'question': question})
```

polls/templates/polls/results.html

```
<h1>{{ question.question_text }}</h1>

<ul>
{% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
{% endfor %}
</ul>

<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```

pluralize 似乎可以根據votes的數量，自動切換單複數
https://apidock.com/rails/ActionView/Helpers/TextHelper/pluralize



* Use generic views: Less code is better

part 3的index() detail() result()相當簡單但冗贅

These views represent a common case of basic Web development: getting data from the database according to a parameter passed in the URL, loading a template and returning the rendered template. Because this is so common, Django provides a shortcut, called the “generic views” system.

Let’s convert our poll app to use the generic views system, so we can delete a bunch of our own code. We’ll just have to take a few steps to make the conversion. We will:

1. Convert the URLconf.

2. Delete some of the old, unneeded views.

3. Introduce new views based on Django’s generic views.



* Amend URLconf

polls/urls.py
```python
from django.urls import path

from . import views

app_name = 'polls'
urlpatterns = [
    path('', views.IndexView.as_view(), name='index'),
    path('<int:pk>/', views.DetailView.as_view(), name='detail'),
    path('<int:pk>/results/', views.ResultsView.as_view(), name='results'),
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```

Note that the name of the matched pattern in the path strings of the second and third patterns has changed from <question_id> to <pk>

* Amend views

remove our old index, detail, and results views

use Django’s generic views instead.

polls/views.py

```python
from django.http import HttpResponseRedirect
from django.shortcuts import get_object_or_404, render
from django.urls import reverse
from django.views import generic

from .models import Choice, Question


class IndexView(generic.ListView):
    template_name = 'polls/index.html'
    context_object_name = 'latest_question_list'

    def get_queryset(self):
        """Return the last five published questions."""
        return Question.objects.order_by('-pub_date')[:5]


class DetailView(generic.DetailView):
    model = Question
    template_name = 'polls/detail.html'


class ResultsView(generic.DetailView):
    model = Question
    template_name = 'polls/results.html'


def vote(request, question_id):
    ... # same as above, no changes needed.
```

ListView: display a list of objects

DetailView: display a detail page for a particular type of object

Each generic view needs to know what model it will be acting upon. This is provided using the model attribute.

The DetailView generic view expects the primary key value captured from the URL to be called "pk", so we’ve changed question_id to pk for the generic views.

可用template_name指定template name，否則是用預設，例如<app name>/<model name>_detail.html

For DetailView the question variable is provided automatically – since we’re using a Django model (Question), Django is able to determine an appropriate name for the context variable.

However, for ListView, the automatically generated context variable is question_list. To override this we provide the context_object_name attribute, specifying that we want to use latest_question_list instead.

# Reference:

https://docs.djangoproject.com/en/2.2/intro/tutorial01/

<https://openhome.cc/Gossip/CodeData/PythonTutorial/DjangoPy3.html>


