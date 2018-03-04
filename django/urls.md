# Django URLs

**Note:** This tutorial was originally written using Python 3.5 and Django 1.9.

## `urls.py`

When Django receives a request, it will iterate over its `urlpatterns` of your URLconf (URL configuation) to match the URL to a view.

When you call `startproject`, a `urls.py` file will be put in your project directory with some default values. However, the urls file can go anywhere, you just need to specify the location using the `ROOT_URLCONF` variable in your settings module.

### `urlpatterns`

`urlpatterns` is a list of `url` instances. You need to have a list named `urlpatterns` in your urls file.

```python
urlpatterns = [
  url('^about$', views.about),
  url('^contact$', views.contact)
]
```

Multiple lists of urls can be joined together.

```python
urlspatterns = [
    url('^about$', views.about),
]
urlpatterns += [
    url('^profile/', include('profile.urls'))
]
```
  
### `url`

A `url` is an instance of `django.conf.urls.url`. It requires two arguments, a regular expression and a view function. When a url is matched by the regular expression, its view function will be called.

```python
url(regex, view)
```

#### `regex`

The first argument to the url function is a regular expression to describe the URL.

```python
url('^profile/', view)
```

The regular expression can include variables that will be used by the view. These are captured using named groups in the regular expression. For example, a profile might include a username. The view function will be passed any matched variables as `kwargs`.

```python
# the url /profile/lee will call view(request, username="lee")
urlpatterns = [
    url('^profile/(?P<username>[a-zA-Z0-9_]+)$', view)
]
```

#### `view`

The view function will be called when the url is matched against. It will be passed the request and any extra kwargs from the regular expression or options from the url function.

#### `include`

Instead of a view function, you can call the `include` function. It takes either a string, which is the location of a urls module, or a list of url instances. This is used for nesting urls.

```python
from django.conf.urls import url, include

#import views
urlpatterns = [
    url('^profile/', include('profile.urls'))
]

# nest views
urlpatterns = [
    url('^(?P<name>[a-zA-Z0-9_]+)/', include([
      url('^$', views.index, name='profile-index'),
      url('^settings', views.settings, name='profile-settings')
    ]))
]
```

#### `kwargs`

You can pass additional options to your view with the kwargs dict. Each key in the kwargs will be used as a keyword argument when the view is called.

```python
url('^contact$', views.contact, {foo: 'bar'})
# when the url /contact is requested,
# views.contact(request, foo='bar') will be called
```

#### `name`

Naming a view makes it convenient to generate a related url by reversing the name (using the `urlresolvers.reverse` function). For example, if you have a url with a name 'contact', in your template you can reference it and the correct url to your contact view will be inserted. This name can also be used in the `get_absolute_url` method of a Model.

```python
# urls.py
urlpatterns = [
    url('^contact$', views.contact, name='contact')
]
```
```html
<!-- index.html -->
<a href="{% url 'contact' %}">Contact</a>
<!-- this will resolve to -->
<a href="/contact">Contact</a>
```

#### Namespaces

If you use simple names for your views in your `url` instances, you risk running into name conflicts. In order to help avoid that, views can be namespaced.

```python
# vegetable/urls.py

"""
In order to add a namespace for a set of views, add an
app_name variable to the urls.py module
"""
app_name = 'vegetable'
urlpatterns = [
  url(r'...', detail_view, name='detail')
]
```

Now, instead of specifying the name of a view, you will specify the namespace and the view name separated by a colon. `reverse('vegetable:detail')` instead of `reverse('detail')`.

#### 404

If no urls match the request's URL, a 404 response is returned.  

## App URLs

It can be useful to define your URLs on a per-app basis. The easiest way to do this is to add an `urls.py` file to your app's directory.

```
<root>
| - manage.py
| - <project_name>
+ - <app_name>
    | - __init__.py
    | - models.py
    + - urls.py
```

In your app's urls module, you define urls similarly to the way your project's urls are organized: inside of a list called `urlpatterns`. Doing this also lets you group all of the app's urls under one base pattern.


```python
# project/urls.py
from django.conf.urls import url, include

urlpatterns = [
  url('^app/', include('app.urls'))
]
```

```python
# app/urls.py
from django.conf.urls import url
from .views import *

urlpatterns = [
  # /app/
  url('^$', index_view, name='app_index'),
  # /app/favorites
  url('^favorites', favorites_view, name='app_favorites')
]
```

## Admin URLs

It is simple to add the admin related urls to your urlpatterns. You just need to make sure that `django.contrib.admin` is included in your settings' `INSTALLED_APPS`.

```python
from django.contrib import admin

urlpatterns += [
    url(r'^admin/', admin.site.urls),
]
```

## Class-based View URLs

For class based views, call the view's `as_view` method.

```python
from your_app.views import YourClassBasedView

urlpatterns = [
    url('^some_pattern$', YourClassBasedView.as_view()),
]
```

You can also use static templates with the `TemplateView`.

```python
from django.views.generic import TemplateView

urlpatterns = [
    url('^$' TemplateView.as_view(template_name='index.html'))
]
```

## `urlresolvers`

[documentation](https://docs.djangoproject.com/en/2.0/ref/urlresolvers/)

Django provides some url resolving functions that help you work with urls in your project. The one that you're most likely to find useful is reverse.

#### `reverse`

`reverse` returns the URL that corresponds to a view. The first argument is be either name of the view (as defined in the `url` instance). It can take any args or kwargs that add any additional variables that are used in the URL. The first argument can also be the view function, but this does not work with namespaced views.

```python
from django.core.urlresolvers import reverse
reverse('vegetable', pks={pk: self.pk})
```
