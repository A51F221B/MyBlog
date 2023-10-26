---
title: Basics of Django Part 2
date: '2023-09-10'
tags: ['python', 'Programming', 'Django','backend','development','web development','python programming']
draft: false
summary: In this post we explain models in Django and how to use them.

---


## Starting A Project


## Models 
Models are simply where you describe the layout of your database. All model classes are inherited from `django.db.models` class, so make sure to add its import in the `models.py` file in the project.
```python
from django.db import models

# all the classes are inherited from models.Model
class Question(models.Model): # question will be the name of table
    question_text=models.CharField(max_length=200)
    pub_date=models.DateTimeField('date published')
    
    def __str__(self):
        return self.question_text
    

class Choice(models.Model):  # choice will be the name of the table
    questions=models.ForeignKey(Question,on_delete=models.CASCADE)
    choice_text=models.CharField(max_length=200)
    votes=models.IntegerField(default=0)
    
    def __str__(self):
        return self.choice_text

```

Our database will use `question_text,pub_date,choice_text` as **column names** and we will use these entities in our code as well.

```ad-note
 Note : Whenever we make changes to models.py i.e. changes to databases; we have to make migrations.
```

**Quick example**
This example model defines a `Person`, which has a `first_name` and `last_name`:

```python
from django.db import models

class Person(models.Model):
	first_name = models.CharField(max_length=30)
	last_name = models.CharField(max_length=30)
```

`first_name` and `last_name` are fields of the model. Each field is specified as a class attribute, and each attribute maps to a database column.
The above Person model would create a database table like this:

```sql
CREATE TABLE myapp_person (
	"id" serial NOT NULL PRIMARY KEY,
	"first_name" varchar(30) NOT NULL,
	"last_name" varchar(30) NOT NULL
);
```
### Migrations 
```bash
$ python manage.py makemigrations
$ python manage.py migrate
```
When we enter these commands all the new tables in the database will be created.The `migrate` command will take care of our database but in case if we want to see what changes it makes at the SQL level, we can use this command :
`python manage.py sqlmigrate [app name] 0001`
```ad-note
We can also use `python manage.py check` to check for any problems in our project without making any migrations or changes.

We can also use `python manage.py shell` to have a advance python-django shell where we can play with the APIs.
```

## Django Admin Interface
To create a super user type  `python manage.py createsuperuser`

```python
from django.contrib import admin

# Register your models here.
from .models import Question

admin.site.register(Question)
```

We we want to register any of our models in the admin interface we have to mention them in the `admin.py` file.
```python
from django.contrib import admin
from .models import Question  # class Question
# Register your models here.
admin.site.register(Question) # Question is the model name i.e. class name in 				     					  #models.py file
```

## URLs 
Django uses are module called `URLconf` to map all the urls.

```ad-note
Now the important thing to note is there is a `urls.py` file made by default in Django which points to our project's URL and there is also another `urls.py` that we create in our project that points to all the `views.py`.
```

The `urls.py` file in the app directory contains all the views mapped to it.

```python
from django.urls import path
from . import views
urlpatterns = [
	path('articles/<int:year>/', views.year_archive),
	path('articles/<int:year>/<int:month>/', views.month_archive),
	path('articles/<int:year>/<int:month>/<int:pk>/', views.article_detail),
]
```

The above code maps urls to [[#Views]] which are `call back functions`.
The `urls.py` file in the project directory  maps all the urls to the apps.`include()`
is always used when mapping to a app , only admin site is the exception.

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('polls/',include('polls.urls')),
]
```

`path(route,view,kwargs,name)` : where kwargs and name are optional parameters.

## Views
Views handle all the requests from the user. A certain view is shown to the type of request that user has made.**To call a view we have to map it to a url**.
```python
# example code 
from django.shortcuts import render
# Create your views here.
from django.http import HttpResponse

def index(request):
    return HttpResponse("This is the index page")
```
A view does 3 basic things
- Taking a `request` parameter.
- Returning `HTTPResponse` object 
- Or raising an exception i.e. `Http404`

## Creation of Project
- Creating project
`django-admin start project [project name]`
- Run server
`python manage.py runserver`
```ad-note
If server does not run due to the port already in use:
sudo lsof -t -i tcp:8000 | xargs kill -9 
```
- Start app
`python manage.py startapp [app name]`
- Write [[#Views]] 
- Create a URLconf in the app directory with the name `urls.py`
- Add the views in the url file.
- Define [[#Models]]
- Include app in the `settings.py` list of installed apps.
- Make [[#Migrations]]
- Add models to [[#Django Admin Interface]]
