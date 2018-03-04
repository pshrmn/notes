# Django Basics

**Note:** This tutorial was originally written using Python 3.5 and Django 1.9.

## Setup

### Virtual Environment

  Often times when developing you will have multiple Django projects with different requirements. For example, when you start a new project, you will most likely be using the latest version of Django. For older projects, they might be using an older version of Django. Even for projects that use the same version of Django, they might use different versions of other packages. To avoid these conflicts, you should make use of virtual environments. Each environment has its own Python executable as well as packages.

```bash
pyvenv <venv_path>
// windows
python -m venv <venv_name>
```

Once you have created a virtual environment, you need to activate it. When the virtual environment is not activated, your global Python will be used and packages will be installed to it. When the virtual environment is activated, its Python executable will be used and any packages that you install (e.g. through pip) will be installed in the virtual environment.

```bash
<venv_name>/bin/activate
// windows
<venv_name>/Scripts/activate
```

### Pip Install

After you have your environment setup to how you would like, you are ready to install Django.

```bash
pip install django
```

## Django Projects

When you install Django, a `django-admin.py` file will be created (if you use a virtual environment, it will be located in the `<venv_name>/Scripts` directory). To create a new Django site, all you have to do is call the django-admin script with the startproject command.


```bash
django-admin startproject <project_name>
```

This will create a project directory, containing the base necessary files for your Django project, as well as a manage.py file which you will use to interact with your project.

```
<root>
| - manage.py
+ - <project_name>
    | - __init__.py
    | - settings.py
    | - urls.py
    + - wsgi.py
```

At this point you can run your server locally, and you will get a "Congratulations" page, but nothing particularly interesting. The site will be accessible at `http://localhost:8000`.

```bash
python manage.py runserver
```

## Settings

Django gets relevant information for a project from a settings module. By default, Django creates a `settings.py` file in your project directory when you create a project. The location of the settings module that Django uses is determined from the environment variable `DJANGO_SETTINGS_MODULE`. You can access the settings for a project by importing them from Django's `conf` module.

```python
from django.conf import settings
```

**Note:** In order for a variable in a settings module to be accessible by Django, the variable name must be in all capital letters. `FOO_BAR` will work, but `baz_quux` will not.

### Multiple Settings Modules

When you create a project using the `startproject` command, a `settings.py` file is created. This is good enough for some simple projects, but you will quickly run into issues once a project becomes complicated. For example, the database that you run in development is not (should not be) the same as the database that is run in production. Also, while you may want to leave debugging on in development, you never want it on in production because of the risk of it leaking information. For this reason, it is convenient to create a `settings` folder that contains a number of settings modules for different situations.

```
// default
settings.py

// better
settings
| - __init__.py
| - base.py
| - dev.py
+ - production.py
```

Any settings that should be shared across environments can be kept in the `base.py` module. Then, in your `dev/production` modules, you can import the base settings (<code class="language-python">from .base import *</code>) and specify other settings that are only relevant to that environment.

When developing locally, you can either specify an environment variable to indicate that the dev settings module should be used or edit `manage.py` to hardcode the dev settings module as the default settings module.

```python
os.environ.setdefault('DJANGO_SETTINGS_MODULE', '<project_name>.settings.dev')
```

In production, you should specify the correct settings module using the `DJANGO_SETTINGS_MODULE` environment variable.

## Django Apps

Django projects are made up of apps. Apps provide a way to group related objects, and are where you will define objects using models that will have corresponding tables in your database. An app's views are used to render responses to HTTP requests related to the app. Apps are created by calling `manage.py` with the startapp command.


```bash
python manage.py startapp <app_name>
```

This will create a directory for the app with some files that you might find useful for your app (but are not necessarily required).

```
<root>
| - manage.py
| - <project_name>
+ - <app_name>
    | - migrations
    | - __init__.py
    | - admin.py
    | - apps.py
    | - models.py
    | - tests.py
    + - views.py
```

Django does not automatically detect that an app exists. Instead it has to be registered by adding its name to the list of `INSTALLED_APPS` in your settings.

```python
INSTALLED_APPS = [
    # ...
    'app_name'
]
```

For more information on the various files in an app's directory, check out these tutorials on [models](./models.md), [views](./views.md), [urls](./urls.md), and [admin](./admin.md).
