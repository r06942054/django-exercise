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


Reference:

https://docs.djangoproject.com/en/2.2/intro/tutorial01/

<https://openhome.cc/Gossip/CodeData/PythonTutorial/DjangoPy3.html>
