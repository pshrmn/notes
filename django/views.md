# Django Views

**Note:** This tutorial was originally written using Python 3.5 and Django 1.9.

**Note:** If you want to test any of the code out in a Python REPL, make sure to call `python manage.py shell` instead of opening a regular Python REPL so that you know your project's settings have been configured.

## About Views

A view takes an HTTP request and returns a response.

```python
from django.http import HttpResponse

"""
the first argument to a view function is a request object
"""
def view(request):
    return HttpResponse("Roger.")

"""
other arguments can also be passed. This is useful for
any keyword args that are added from the matching url
in your URLconf
"""

# urls.py
urlpatterns = [
    url('^/car/(?P<pk>[0-9]{4,6}$', car_view),
    url('^/cars$', cars_view, {'foo': 'bar'})
]

# views.py

"""
You can match additional arguments directly by name
"""
def car_view(request, pk):
    # handle request

"""
or generically by kwargs
"""
def cars_view(request, **kwargs):
    # handle request
```

## Requests

[documentation](https://docs.djangoproject.com/en/2.0/ref/request-response/#httprequest-objects")

A request object has a variety data available as attributes that are useful in creating a response.

```python
def view(request):
    """
    You can respond to requests based on its method.
    The method will always be in all capital letters.
    """
    if request.method == 'GET':
        # handle the get request
    elif request.method == 'POST':
        # handle the post request

    """
    Any query parameters will be in a GET QueryDict
    https://docs.djangoproject.com/en/2.0/ref/request-response/#querydict-objects
    """
    get_params = request.GET

    """
    And any POST parameters (from a POST request)
    will be in a POST QueryDict
    """
    post_params = request.POST
    # the post data can be used to fill in forms
    form_data = YourForm(request.POST)

    """
    If the request sent any cookies, they will be
    available as a dict. Modifying any cookies has
    to be done using the response's set_cookie method
    """
    cookies = request.COOKIES

    """
    The headers sent with the request are also
    available as a dict
    """
    headers = request.META

    """
    Middleware will also add attributes to the request.
    """

    """
    If you are using the SessionMiddleware,
    there will be a session attribute dict, which
    you can modify to update.
    """
    session_id = request.session['id']
    request.session['hakuna'] = 'matata'

    """
    If you are using the AuthenticationMiddleware,
    there will be a user attribute. The type of the user
    is determined by settings.AUTH_USER_MODEL
    """
    user = request.user
    # check if the user is logged in using is_authenticated
    logged_in = user.is_authenticated()
```

## Responses

Your view should return a response to its request.

```python
from django.http import HttpResponse

def view(request):
    """
    The most basic way of creating a response is to create an
    HttpResponse, passing it a string of the content of the response.
    """ 
    response = HttpResponse("This is my response.")  

  
    # Headers can be set on a response like keys on a dict.
    response['header-field'] = 'header-value'

    """
    Cookies are set using the response's set_cookie method
    set_cookie's main arguments are a key and value
    https://docs.djangoproject.com/en/2.0/ref/request-response/#django.http.HttpResponse.set_cookie
    """
    response.set_cookie('best_cookie', 'orange-cranberry')
    # you can also delete any cookies by key
    response.delete_cookie('worst_cookie')

    return response
```

### Classes

Django provides a variety of response classes for handling different types of requests.

```python
"""
The most common response is an HttpResponse. It has a status code
of 200 and a content-type derived from settings.DEFAULT_CONTENT_TYPE
It takes a string argument which it will encode to bytes.
"""
http_resp = HttpResponse(';)')

"""
If you want to respond with JSON, use a JsonResponse
"""
resp = {
    'foo': 'bar' 
}
json_resp = JsonResponse(resp)
# the content-type of of the response will be set to application/json
json_resp._headers['content-type'] == ('Content-Type', 'application/json')
```

### Status Codes

There are a number of HttpResponse subclasses that are used to handle different response statuses.

```python
"""
A regular HttpResponse has a status code of 200.
"""
http_resp = HttpResponse('...')
http_resp.status_code == 200

"""
Subclasses handle other situations
"""
redirect_resp = HttpResponseRedirect('looks like it\'s over there')
redirect_resp.status_code == 302

four_oh_four = HttpResponseNotFound('something went wrong')
four_oh_four.status_code == 404
internal_error = HttpResponseServerError('uh oh')
internal_error.status_code == 500

"""
If something goes wrong in your view and you need to send a 404 response,
you can raise Http404 and Django will handle returning a 404 response for you.
"""
from django.http import Http404

def failure_view(request):
    raise Http404
```

## Template Responses

While writing the text to output to an `HttpResponse` is fine for simple responses, it quickly becomes painful when rendering more complex responses. To deal with this, Django provides two classes: `SimpleTemplateResponse` and `TemplateResponse`.

#### `SimpleTemplateResponse`

```python
"""
A SimpleTemplateResponse is a subclass of HttpResponse
"""
from django.template.response import SimpleTemplateResponse

def view(request)
    """
    The first argument is a template object or a string of the
    path to the template.
    """
    template_path = "path/to/template.html"
    """
    It can also take a context dict with the data to be used in
    rendering the template. The context is where you would put
    any model objects that you have fetched from your database.
    """
    context = {
      'method': request.method
    }
    resp = SimpleTemplateResponse(template_path, context)
    return resp.render()
```
```html
<!-- path/to/template.html -->
<html>
  <body>
    You made a {{ method }} request.
  </body>
</html>
```
  
#### `TemplateResponse`
```python
"""
TemplateResponse is a subclass of SimpleTemplateResponse
that takes a request as its first argument. This will add
request related properties to the context (as configured
by the context_processors of the TEMPLATES variable in
your settings module)
"""

from django.template.response import TemplateResponse

def view(request)
    return TemplateResponse(request, "path/to/template.html", {})
```


## Shortcuts

[documentation](https://docs.djangoproject.com/en/2.0/topics/http/shortcuts/)

Django provides shortcut functions for common tasks related to requests and responses.

```python
"""
render takes the request, the location of the template, and an optional
context dict and returns an HttpResponse.
"""
from django.shortcuts import render

def view(request):
    return render(request, 'path/to/template.html', {'context': 'variables'})


"""
redirect returns an HttpResponseRedirect to a different url.
The different url is determined based on the to argument.
"""
from django.shortcuts import redirect

def from_view(request):
    # if to is a model, it uses the model's get_absolute_url method to
    # determine the path to redirect to
    model_redirect_resp = redirect(to=FooModel)
    # if to is a view name, path determined by urlresolver.reverse function
    view_redirect_resp = redirect(to='to-view')
    # and if to is a string, it simply uses that
    str_redirect_resp = redirect('/some/path')

    # passing permanent=True will return an HttpResponsePermanentRedirect
    # this response will have a status code of 301
    perm_redirect = redirect('/perm/path', permanent=True)
 
"""
get_object_or_404 will raise a 404 error if the object you attempt to
get from the database doesn't exist.
"""
from django.shortcuts import get_object_or_404

from .models import Car

def car_view(request, car_id):
    car = get_object_or_404(Car, car_id)
    # an error was raised if a Car with car_id didn't exist
    # if you get to here, the car does exist, so proceed
    return render(request, 'cars.html', {'car': car})

"""
get_list_or_404 is the list equivalent of get_object_or_404
It filters the model based on field lookups and raises an Http404 error
if the returned list is empty
"""
def cars_view(request):
    fast_cars = get_list_or_404(Car, top_speed__gte=120)
    return render(request, 'cars/cars.html', {'fast_cars': fast_cars})
```
