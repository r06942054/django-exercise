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


Reference:

https://docs.djangoproject.com/en/2.2/intro/tutorial01/

<https://openhome.cc/Gossip/CodeData/PythonTutorial/DjangoPy3.html>
