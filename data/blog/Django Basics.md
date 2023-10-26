---
title: Basics of Django Part 1
date: '2023-08-29'
tags: ['python', 'Programming', 'Django','backend','development','web development','python programming']
draft: false
summary: Django is a python based web framework used to build web applications. It is a high level framework that provides a lot of features out of the box. It is a Model View Template framework.

---


# Django Notes
### Reference : Code With Mosh
---

### Setting up a development environment
First we can setup a virtual environment where we can install latest version of django.

```bash
╭─asif221b@Mac ~/Documents/storefront
╰─$ python3 -m virtualenv env
created virtual environment CPython3.9.13.final.0-64 in 2060ms
  creator CPython3Posix(dest=/Users/asif221b/Documents/storefront/env, clear=False, no_vcs_ignore=False, global=False)
  seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/Users/asif221b/Library/Application Support/virtualenv)
    added seed packages: pip==22.1.2, setuptools==62.6.0, wheel==0.37.1
  activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
╭─asif221b@Mac ~/Documents/storefront
╰─$ source env/bin/activate
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$ pip3 install django
Collecting django
  Using cached Django-4.0.6-py3-none-any.whl (8.0 MB)
Collecting sqlparse>=0.2.2
  Using cached sqlparse-0.4.2-py3-none-any.whl (42 kB)
Collecting asgiref<4,>=3.4.1
```

Now we have to start a django project.

```bash
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$ django-admin startproject storefront .
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$
```

> The . at the end means django will not create a directory storefront but use the current directory for the project instead.


Once a django project is initialized, django creates a few files for us.The following is the file hierarchy in this case.

![[Pasted image 20220718231026.png]]

- The `manage.py` file is used to manage the entire project and running the server.
- The `setting.py` file is where all the apps for the project are listed.

#### Running Django Server
Simply use the following command to run the server.Make sure the virtual environment is activate to avoid any errors.By default django runs the server on port `8000`.

```bash
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$ python3 manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
July 18, 2022 - 18:15:13
Django version 4.0.6, using settings 'storefront.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.

```

Now navigate to the given address on browser.

### Creating a Django App within a Django Project
A single django project is made by combining different django apps inside it.Django comes with a few pre-installed applications.We can see them in our `settings.py` file in the project. There will be the following list present in the file.

```python
INSTALLED_APPS = [

'django.contrib.admin',
'django.contrib.auth',
'django.contrib.contenttypes',
'django.contrib.sessions',
'django.contrib.messages',
'django.contrib.staticfiles',

]
```

These apps include the admin panel provided by django as well.So we can start our own app according to our own project needs.In this case we will be starting an app called playground.

```bash
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$ python3 manage.py startapp playground
(env) ╭─asif221b@Mac ~/Documents/storefront
╰─$
```

This will create a folder with the name of the app with a few files inside.

![[Pasted image 20220718232315.png]]

### Writing views
A view simply takes a request and return a particular http response.Lets write a simple few that returns hello world string.

```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.

def say_hello(req):
	return HttpResponse('Hello World!')
```

Now we have to add it to a particular `url` to which this response would be sent.For example in our case we want the page to show hello world when the user goes to the url `/playground/hello`.To map a view to a url first create a file called `urls.py` in the same app folder.The following content is entered in that file.

```python
from django.urls import path
from . import views

urlpatterns=[
	path('hello/', views.say_hello),

]
```

Now we need to import this url configuration into the main `urls.py` file for the project.Now the following content is entered in the `urls.py` file of the storefront folder.

```python
from django.contrib import admin
from django.urls import path,include

urlpatterns = [
	path('admin/', admin.site.urls),
	path('playground/', include('playground.urls')),

]
```

### Templates 
In order to use views to return html templates we can simply use the `render()` function in the views file.First create a folder called templates in the playground app and make a file called `index.html` with some html code inside.


```python
from django.shortcuts import render
from django.http import HttpResponse

# Create your views here.

def say_hello(req):
	return render(req, 'index.html')
```


### Models
To create models for any app open the `models.py` file.All the model classes are inherited from `model.Models` class.

```python 
from django.db import models

  

# Create your models here.
class Product(models.Model):
	title=models.CharField(max_length=255)
	description=models.TextField(blank=True)
	price=models.DecimalField(max_digits=10, decimal_places=2)
```

We make migrations every-time we apply any changes to the database.

### Django ORM (Object Relational Mapper)
Its maps objects to relational data.