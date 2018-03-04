# Django Templates
**Note:** This tutorial was originally written using Python 3.5 and Django 1.9.

**Note:** If you want to test any of the code out in a Python REPL, make sure to call `python manage.py shell` instead of opening a regular Python REPL so that you know your project's settings have been configured.
  
## About Templates

Django comes with support for its own template syntax as well as Jinja2. This tutorial will focus on the Django Template language. Templates are largely made up of HTML, but also incorporate tags and variables for dynamic content.

When a template is rendered, it will have access to a number of context variables that can be used to customize the HTML output. These variables can either be passed from the view function to the template or be automatically included by context processors.

## Settings
```python
TEMPLATES = [
    {
        # the template engine
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        # which directories to look for templates in
        'DIRS': [],
        # if True, check for templates in app directories
        'APP_DIRS': True,
        'OPTIONS': {
            """
            context_processors are used to add extra data to the context,
            which can be used when rendering a template
            """
            'context_processors': [
                # add the request to the context dict
                'django.template.context_processors.request',
            ],
            """
            When True, debug will return a debug page when template
            errors occur. This should always be False in production.
            """
            'debug': DEBUG
        },
    },
    {
        """
        When using jinja2 templates, you will have to do extra work
        to get the context support similar to Django templates.
        However, request, csrf_input, and csrf_token variables will
        be automatically added to the context in jinja2 templates.
        """
        'BACKEND': 'django.template.backends.jinja2.Jinja2',
    }
]
```

## Basics

Besides HTML, Django templates are composed of tags, variables, and filters.

### Tags

[documentation](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/)

```python
"""
Tags are enclosed by curly braces and percentage signs
"""
{% tag %}

"""
When tags describe a block of code, they required an end tag
which is a combination of end and the tag's name (eg for and endfor)
"""
{% for person in team.player_set.all %}
  <li>{{person.name}}</li>
{% endfor %}
```

#### Common Tags
```python
"""
block
Describes a block of content. These can be overriden in
inherited templates.
"""
{% block main %}
  Block Content
{% endblock main %}

"""
extends
import a base template to use. This can either be the string
name of a template or a Template object.
The base template will then replace (unless the block.super variable
is included) any blocks in the base template with the ones
included in the current template
"""
{% extends 'base.html' %}

"""
include
import content from a different template and include it at
the current location. The included template will have access
to context variables depending on where it is imported. For example
including a template in a for loop will give it access to the
variable for the current iteration
"""
{% for row in table %}
  {% include 'row_template.html' %}
{% endfor %}

"""
csrf_token
include this in any forms to prevent cross site request forgeries
https://docs.djangoproject.com/en/2.0/ref/csrf/
"""
<form>
{% csrf_token %}
</form>

"""
url
let django create url paths for you. You can include arg
or kwarg variables, but not both.
"""
{% url 'profile' user.pk %}

"""
for
loop over iterable objects in for loops
"""
{% for thing in list %}
  {{ thing }}
{% endfor %}

"""
empty
Django provides the empty tag for when an iterable
is empty (contains no values)
"""
{% for thing in list %}
  {{ thing }}
{% empty %}
  There were no things.
{% endfor %}

"""
if, elif, else
conditionally set content
"""
{% if color == 'blue' %}
  Blue is the best!
{% elif color == 'brown' %}
  Brown is the worst!
{% else %}
  {{ color|capfirst }} is okay!
{% endif %}

"""
firstof
Returns the first variable that isn't False
"""
{% firstof foo bar baz 'default' %}

"""
autoescape
Autoescape lets you control whether or not content
is automatically escaped. Turning off auto-escape can be
dangerous if the content isn't trusted.
"""
{% autoescape off %}
  <script>alert("muahahaha!")</script>
{% endautoescape}

"""
with
If you have a variable that you want to cache (expensive
to generate) do so with the with tag.
"""
{% with foo=bar.baz.foo %}
  {{ foo }}
{% endwith %}
```

### Variables
```python
"""
Variables are placed between doubly curly braces
"""
{{ variable }}

"""
Use dot attribute name to access dict values,
instance attributes, and iterable indices
"""
{{ user.username }}
```

### Filters

Often times you will want to modify a variable value before it is inserted into the page. This is done through filters, which is specified inside of a variable tag, by listing the variable, then a pipe `|`, and then the filter function.

[documentation](https://docs.djangoproject.com/en/2.0/ref/templates/builtins/#built-in-filter-reference)

```python
{{ variable|capfirst }}

"""
Multiple filters can be chained together
"""
{{ variable|lower|capfirst }}

"""
Some filters take arguments, which are defined using a colon
then specifying the argument value. Only one argument is
allowed for a filter.
"""
{{ number|divisibleby:7 }}
```

#### Common Filters
```python
"""
date
format a date for output
"""
{{ variable|date:"Y-M-D" }}

"""
default
specify a default value in case the variable is False
"""
{{ variable|default:"Default Value" }}

"""
first/last
return the first or last item in a list
"""
{{ a_list|first }}
{{ a_list|last }}

"""
join
join the item in a list together
"""
{{ a_list|join:", " }}

"""
urlencode
escape characters to be used in a URL
"""
{{ url_variable|urlencode }}
```

### Calling Functions
```python
"""
Methods/functions are referenced, but not called in
tags and variables
"""

{% for player in team.player_set.all %}
  {{ player.get_full_name }}
{% endfor %}
```

## Extending Templates

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
  <title>
    {% block title %}
    Your Site!
    {% endblock title %}
  </title>
</head>
<body>
  <main>
    {% block main %}
      Default Content
    {% endblock main %}
  </main>
</body>
</html>

<!--
page.html

When you extend a base template, you only need to
override the blocks that need to change
-->
{% extends 'base.html' %}

<!--
include the block.super variable to include the
content from the parent version of the block
-->
{% block title %}
  Special Page | {{ block.super }}
{% endblock title %}

{% block main %}
Special Content
{% endblock main %}
```

```html
<!--
The output of rendering the page.html will be:
-->
<!DOCTYPE html>
<html>
<head>
  <title>
    Special Page |Your Site!
  </title>
</head>
<body>
  <main>
    Special Content
  </main>
</body>
</html>
```

## Rendering Forms

Django templates understand how to render forms, you just need to include the `form` variable inside of an HTML `<form>`. These widget for each form field will be used to determine which input (or other tag) type will be used when rendering the field.

```html
<form>
  {{ form }}
</form>
```

Each field in the form will render an `<label>` and the `<input>` (or other tag) for the field. The text for the `<`label`>` is configured using the `label` and `label_suffix` options of a field.

The form has three methods that will alter how the form fields are rendered.

* `form.as_p` wraps the field tags in a `<p>` tag.
```html
{{ form.as_p }}
```
* `form.as_table` wraps the field tags in a `<tr>` tag. This does not provide the `<table>` and other relevant tags. You will need to include those in the template.
```html
<table>
  {{ form.as_table }}
</table>
```
* `form.as_ul` wraps the field tags in an `<li>` tag. This does not provide the `<ul>` tag. You will need to include it in your template. While this is named `as_ul`, because the `<ul>` tag isn't provided, you can also use an `<ol>`.
```html
<ul>
  {{ form.as_ul }}
</ul>
```

**Note:** Don't forget to include the CSRF token in the form.
```html
<form>
  {% csrf_token %}
  {{ form }}
</form>
```

You will also need to provide the mechanism for submitting the form (most likely a button).

```html
<form>
  {% csrf_token %}
  {{ form }}
  <button>Submit!</button>
</form>
```