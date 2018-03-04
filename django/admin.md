# Django Admin

Django provides an admin interface that allows authorizing users to add/edit/remove content.

## Admin Views

For each app that is registered with the admin site, a few views will be available by default.

* **List Views** list the rows that exist in the table.
* **Changes Views** show the contents of a row in an editable form.
* **Add Views** provide a form to create a new row in a table.
* **Delete Views** make you confirm that you want to delete a row (or rows) before actually doing so.

## Settings

In order to use the admin site, you need to make sure that the admin app and its dependencies are included in the installed apps

```python
INSTALLED_APPS = [
    # ...,
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.messages',
    'django.contrib.sessions',
    'django.contrib.admin'
]
```

In order to make sure all of the necessary data is available to the templates when rendering the admin site, make the auth and messages context processor available.

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [#...],
        'APP_DIRS': True
        'OPTIONS': {
            'context_processors': [
                # ..., 
                'django.contrib.auth.context_processors.auth',
                'django.contrib.messages.context_processors.messages',
            ]
        }
    }
]
```

The authentication and messages middleware also need to be included.

```python
MIDDLEWARE_CLASSES = [
    # ...,
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
]
```

## URLs

To make sure that you can navigate to the admin site, all that you need to do is to include the admin urls in your urlpatterns.

```python
from django.contrib import admin

urlpatterns += [
    url(r'^admin/', admin.site.urls),
]
```

Typically the path to these urls will begin with /admin, but you can specify anything you want. If you want to "hide" the admin site, you can use an obscure path. This won't prevent someone from navigating to it, but it won't be as obvious.

```python
urlpatterns += [
    url(r'^4dm1n/', admin.site.urls),
]
```

## Users

Only a few users should have access to the admin site. Using the built-in User model, these will be users that are either staff or superusers.

```bash
python manage.py createsuperuser
```

The user that `createsuperuser` created will be able to log in to the admin site. The superuser will also be able to edit other users to give them access to the admin site. When giving another user access to the admin site, you should almost always give them staff access (or whatever equivalent your user model has) instead of superuser access. 

Superuser access allows the user to view/edit/delete everything, which most (if not all) users should not be able to do. By using the `PermissionsMixin` with your User model (included with default user model), you will have baked in support for giving staff members view/edit/delete permissions for each model instead of across everything like a superuser has. One precaution you should take in giving a staff member access to edit the user model or any permissions is that they could give themselves and other more access than is intended and remove access for people that should have it.

## admin.py

Each of your apps that you wish to be available in the admin site need to be registered with it. This is most commonly done by adding an admin.py file to each app directory.

[documentation](https://docs.djangoproject.com/en/2.0/ref/contrib/admin/)

```python
from django.contrib import admin

# import any models that you want to register
from .models import Foo, Bar, Baz

"""
Models need to be registered with the admin site
"""
admin.site.register(Foo)
```

When you register a model with the admin site, you can include a ModelAdmin class that describes what functionality the admin site should provide and what fields of the model should be shown.

There are a number of options that you can define on the ModelAdmin.

```python
class BarAdmin(admin.ModelAdmin):
    
    """
    actions
    an array of actions that are available in a select element
    on the list page for a model. All of the list elements
    that have been selected will have the action peformed on them.
    There is a default delete_selected action to remove any
    select elements from the table. You can write other action
    functions yourself.
    """
    actions = []

    """
    exclude
    a list of fields that should not be shown on the form page
    for the model
    """
    exclude = ('date_joined',)

    """
    fields
    the fields list defines the layout of the forms in the
    add and change views

    the fields list can be nested to provide structure within
    the view
    """
    fields = ('name', 'height', 'weight')
    fields = ('name', ('height', weight'))

    """
    fieldsets
    fieldsets provide a more fine tuned way to control layout
    than the fields option. Each element in the fieldsets tuple
    defines a section of the view. The elements are tuples where
    the first value is the name of the section (or None) and the
    second is an options dict.
    """
    fieldsets = (
        (
            None,
            {
                'fields': ('name',)
            }
        ),
        (
            'Attributes',
            {
                # include any relevant fields in the fields tuple
                'fields': ('height', 'weight'),
                # any classes that should be set on the fieldset
                'classes': ('collapse', 'wide'),
                # a description of the fieldset
                'description': 'These things are likely to change.'
            }
        )
    )

    """
    form
    specify the form to use in the add and change views
    """
    form = BarForm

    """
    list_display
    specify which fields to show in the list view. These values can
    also be attributes or methods of the model.
    """
    list_display = ('pk', 'name')

    """
    list_filter
    fields that the rows in the list view can be filtered on. All of
    the fields should have one of these types: BooleanField, CharField,
    DateField, DateTimeField, IntegerField, ForeignKey or ManyToManyField
    """
    list_filter = ('is_superuser',)

    """
    ordering
    specify the order of the rows in the list view
    """
    ordering = ['-height']

    """
    readonly_fields
    fields that should be displayed but not editable
    """
    readonly_fields = ['pk',]


admin.site.register(Bar, BarAdmin)
```

Django also provides a decorator as a shortcut to registering a model

```python
@admin.register(Baz)
class BazAdmin(admin.ModelAdmin):
    # ...
```
