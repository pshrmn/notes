# Django Models

Models are a link between your database and your app. Each model class corresponds to a table in your database and each field of the model class is a column in that table.

For most examples, two related models will be used: a vegetable and a garden in which vegetables grow.

**Note:** This tutorial was originally written using Python 3.5 and Django 1.9.

**Note:** If you want to test any of the code out in a Python REPL, make sure to call `python manage.py shell` instead of opening a regular Python REPL so that you know your project's settings have been configured.

## Creating Models

### Model Definitions

```python
"""
The models module contains the classes and other variables
necessary for creating a model.
"""
from django.db import models

"""
A Django model is defined using a class that inherits from the 
models.Model base class.
"""
class Vegetable(models.Model):
    pass
```


A table is created in your database for each model. The name of the table is derived from the name of the app your model is in and the name of the model.

The Vegetable class is in the garden app, so in the database a `garden_vegetable` table will be created.

```python
# garden/models.py
class Vegetable(models.Model):
    pass
```

### Fields

Django models are made up of fields. Each field in a model will map to a column in the accompanying database table. Django provides a large number of fields. The table column for a field will be configured based on the field type and the options that you provide to it. For a full list of built-in fields, you can check out Django's [Model field reference](https://docs.djangoproject.com/en/2.0/ref/models/fields/).


```python
class Vegetable(models.Model):
    """
    Fields are added to the class to specify the columns of
    the model's table in the database.
    """
    name = models.CharField(max_length=100)
    color = models.CharField(max_length=100)
    weight = models.FloatField()
```

In your code, you can directly access fields of a model. This can be done for getting and setting. When you set a value, it **will not** automatically be pushed to the database. You will need to manually save changes. Check out the [ORM](./orm.md) notes for more information on this.

```python
cuke = Vegetable(name="cucumber", color="green", weight=7)
color = cuke.color
cuke.weight = 10
```

#### Primary Keys

Every table should have a primary key, so every model should have a primary key field. However, you do not have to do this manually if you do not want. By default, Django adds an id field to each model, which is used as the primary key for that model. You can create your own primary key field by adding the keyword arg `primary_key=True` to a field. If you add your own primary key field, the automatic one will not be added.


```python
class Garden(models.Model):
    garden_id = models.IntegerField(primary_key=True)
```

The value of a model's primary key can be accessed either through the name of the primary key field (eg. `v.id`) or by referring to the primary key directly (`v.pk`).

```python
cuke = Vegetable.objects.create(name="cucumber")
cuke.id == cuke.pk  # True
```

#### Field Options

Each field on a model can be passed different options. Some options are available to [all fields](https://docs.djangoproject.com/en/2.0/ref/models/fields/#field-options) and others are relevant to only some fields.

Below are some commonly used options.

```python
"""
null (default False)
Specify whether or not the field can be null. This is especially useful
when adding a new field to a model.
"""
age = models.IntegerField(null=True)

"""
default
Specify a default value for the field when none is provided.
"""
favorite_color = models.CharField(default='blue')

"""
primary_key
When primary_key=True, that field will be used as the primary key
of the model. Setting this will prevent the automatic id primary
key field from being used.
"""
ssn = models.CharField(max_length=11, primary_key=True)

"""
choices
An iterable of values used to specify the set of valid options
for a field. Each value should be a tuple of two values, the first
being the field's value and the second being what is shown to the user.

eg. ('C', 'Celsius') will store the value 'C' in the database, but
the in HTML it will output 'Celsius'.
"""
CELSIUS = 'C'
FAHRENHEIT = 'F'
KELVIN = 'K'
TEMP_CHOICES = (
  (CELSIUS, 'Celsius'),
  (FAHRENHEIT, 'Fahrenheit'),
  (KELVIN, 'Kelvin')
)
temperature_scale = models.CharField(max_length=1,
                                     choices=TEMP_CHOICES,
                                     default=CELSIUS)


"""
max_length (default varies by Field type)
The maximum number of characters of the value for a field.
"""
name = models.CharField(max_length=100)

"""      
auto_now (default False)
Used for dates and times, will use a datetime class (time, date, or
datetime) to automatically set the value of the field upon creation.
"""
date = models.DateField(auto_now=True)
date_time = models.DateTimeField(auto_now=True)
time = models.TimeField(auto_now=True)

"""
validators
Sometimes you will want the data for a field to be more strict than
the field type enforces on itself. You can add a list of validators
to a field that will raise a ValidationError when the conditions
are not met.
"""
from django.core.exceptions import ValidationError

def validate_color(color):
    if color == 'brown':
        raise ValidationError('No one\'s favorite color is brown')

favorite_color = models.CharField(validators=[validate_color])
```

### Instance Creation

You can create an isntance for a model by calling the model's class as a function and passing it the associated data. The arguments to the function can either be index-based (args) or keyword-based (kwargs).

**Note:** If you are using an automatic primary key (id), when creating an instance using args the id primary key is the 0th argument to the class, so you need to provide it. You can pass None to let the database handle assigning this value for you.

If you need to programatically check which fields a model has, and in what order, a model class has, call `<model_name>._meta.fields`.

```python
# kwargs with no included primary key
# automatically uses auto-increment
v = Vegetable(name="cucumber", color="green", weight=11)

# args requires the primary key to be included
# use None to allow auto-increment
v2 = Vegetable(None, "cucumber", "green", 11)

"""
In the above example, we haven't done anything with the
primary key, id, of the vegetable. The default id added by
Django is automatically set when we save the instance.
"""

v.pk == None  # True
v.save()
v.pk = 1  # or some other int
```

The `Model`'s `Manager` also provides the create method to create an instance and save it to the database in one command.

```python
v3 = Vegetable.objects.create(name="Tomato", color="red", weight=8)
v3.pk = 3  # or some other int
```

## Relationships

Models become a lot more useful when they relate to one another. Django provides fields to describe many-to-one, one-to-one, and many-to-many relationships between models.

### Many-to-One Relationships

A many-to-one relationship exists when a number of rows in one table relate to a single row in another table.

[documentation](https://docs.djangoproject.com/es/2.0/topics/db/examples/many_to_one/)

```python
"""
A many-to-one relationship is created using a foreign key.
"""
class Garden(models.Model):
    name = models.CharField(max_length=200)
    # ...

class Vegetable(models.Model):
    name = models.CharField(max_length=100)
    # ...
    garden = models.ForeignKey(Garden)

```

To set the foreign key, pass the related instance as the foreign key argument. That instance's primary key will be used as the foreign key.      

Because the primary key of the Garden is required by the Vegetable, the Garden has to be saved before the Vegetable references it so that it actually has a key to be referenced.

```python
bills = Garden(name='Bill\'s Garden')
bills.save()
z = Vegetable(name='zucchini', garden=bills)

"""
In this relationship each vegetable is only grown in one garden,
but the garden can have many vegetables. To reference the garden
from a vegetable instance, you just have to reference the garden
property of the vegetable.
"""
farmers_haven = Garden(name="Farmer's Haven")
farmers_haven.save()
cuke = Vegetable(name="cucumber", ..., garden=farmers_haven)
cuke.save()
tomato = Vegetable(name="tomato", ..., garden=farmers_haven)
tomato.save()

cuke.garden == farmers_haven  # True
```

A garden can have multiple vegetables related to it. You can access them through the `vegetable_set` of the garden. The `vegetable_set` field name is created by concatenating the name of the model and the word "set".

```python
farmers_haven.vegetable_set == [cuke, tomato]
```
  
### One-to-One Relationships<

One-to-one relationships exist when elements in two tables are related, but the relationship is exclusive between one element in the first table and one element in the second table.

[documentation](https://docs.djangoproject.com/es/2.0/topics/db/examples/one_to_one/)

```python
"""
Sometimes a relationship between rows in two tables is exclusive.
In those cases, you can use a models.OneToOneField to specify this.

If our vegetable and garden from above had a one to one relationship,
then each vegetable could only be grown at one garden and each
garden could only grow one vegetable.
"""

class Garden(models.Model):
    name = models.CharField(max_length=200)
    # ...

class Vegetable(models.Model):
    name = models.CharField(max_length=100)
    # ...
    garden = models.OneToOneField(Garden)

"""
In this scenario, because the garden now only can have one vegetable,
its vegetable would be referred to directly using the vegetable property.
"""

farmers_haven = Garden(name="Farmer's Haven")
farmers_haven.save()
cuke = Vegetable(name="cucumber", ..., garden=farmers_haven)
cuke.save()

# reference the Garden from the Vegetable using the garden property
cuke.garden == farmers_haven

# reference the Vegetable from the Garden using the vegetable property.
farmers_haven.vegetable == cuke
```
  
### Many-to-Many Relationships

While multiple vegetables that only grow in one garden or only growing one vegetable per garden may make for convenient examples, in reality many vegetables can be grown in many different gardens. This many-to-many relationship can be modeled in a couple of different ways.

[documentation](https://docs.djangoproject.com/es/2.0/topics/db/examples/many_to_many/)

The first way will create a table linking two models for you. This is convenient when the linking table only has columns for the foreign keys of each instance. When the table linking the models requires additional fields, you will need to create this linking table yourself.

```python
"""
Similar to the ForeignKey and OneToOneField fields, Django provides a
ManyToManyField that can be used to describe a many-to-many relationship.
"""

class Garden(models.Model):
    name = models.CharField(max_length=200)
    # ...

class Vegetable(models.Model):
    name = models.CharField(max_length=100)
    # ...
    gardens = models.ManyToManyField(Garden)

"""
This is useful when the relationship between the two models
is all of the relevant information about the relationship. A
table will automatically be created in the database that links
the two models together, requiring no extra code from you.
"""

farmers_haven = Garden(name="Farmer's Haven")
bills = Garden(name='Bill\'s Garden')

farmers_haven.save()
bills.save()

cuke = Vegetable(name='cucumber')
zuke = Vegetable(name='zucchini')
tomato = Vegetable(name='tomato')

cuke.save()
zuke.save()
tomato.save()
```

Our `Vegetable` model explicitly links itself to the `Garden` model using the `gardens` field. This allows us to access all of the gardens that grow a vegetable by calling `vegetable_instance.gardens`. The `Garden` model does not have an explicit reference to the `Vegetable` model. In order to access all of the vegetables that are grown in a garden, we access a property whose name is `vegetable_set` (similar to many-to-one relationships).

```python
farmers_haven.vegetable_set
cuke.gardens
```

Two instances in models that have a many-to-many relationship can be linked using the `add` method of the related field.

```python
tomato.gardens.add(farmers_haven, bills)
farmers_have.vegetable_set.add(cuke, zuke)
```
  
If the relationship between two models requires more information, a model to describe the relationship should be used. For instance, if we wanted to keep track of how many seeds of a vegetable were planted in a garden, we could add that field to the relationship model.

```python
class Garden(models.Model):
    name = models.CharField(max_length=200)
    # ...

class Vegetable(models.Model):
    name = models.CharField(max_length=100)
    # ...
    """
    Adding a through option to the field will let Django know
    that it should use that model instead of creating one
    for you.
    """
    gardens = models.ManyToManyField(Garden, through='Crop')

"""
The model referenced in the through option needs to have
foreign key fields for each model in the many-to-many
relationship.
"""
class Crop(models.Model):
    garden = models.ForeignKey(Garden)
    vegetable = models.ForeignKey(Vegetable)
    seeds = models.IntegerField()
```

### Deleting Instances of Models
    
[documentation](https://docs.djangoproject.com/en/2.0/ref/models/fields/#django.db.models.ForeignKey.on_delete)

Relationships can be tricky. When an instance of a model has a relationship with an instance of another model, we need to specify what happens when one of those instances is deleted. If we do not, then an instance might contain references to another instance that no longer exists, which can cause errors in your project.

This deletion issue can come up frequently with `ForeignKey` and `OneToOneField` relationships. Django provides these fields an `on_delete` keyword argument which allows you to specify what should be done to the instances's row in the database when the related instance is deleted from the database.

```python
"""
models.CASCADE
When the related ForeignKey model's instance is deleted,
also delete this instance.
"""
garden = models.ForeignKey(Garden, on_delete=models.CASCADE)

"""
models.PROTECT
Prevent this instance from being delete when the related
ForeignKey model's instance is deleted.
"""
garden = models.ForeignKey(Garden, on_delete=models.PROTECT)

"""
models.SET_NULL
Set the ForeignKey to null (requires null=True on ForeignKey).
"""
garden = models.ForeignKey(Garden, null=True, on_delete=models.SET_NULL)

"""
models.SET_DEFAULT
Set the ForeignKey to the default value (requires a default
value to be defined).
"""
garden = models.ForeignKey(Garden, default=0, on_delete=models.SET_DEFAULT)

"""
models.SET()
Takes either a value or a function. If passed a value, set the
ForeignKey to the value. If a function, set the ForeignKey
to the result of the function.
"""
garden = models.ForeignKey(Garden, on_delete=models.SET(0))

# or

def default_garden():
    obj, _ = Garden.objects.get_or_create(name='Babylon')
    return obj

garden = models.ForeignKey(Garden, on_delete=models.SET(default_garden))


"""
models.DO_NOTHING
Self-explanatory.
"""
garden = models.ForeignKey(Garden, on_delete=models.DO_NOTHING)
```

### Users

When a user creates an instance of a model, you may want to save which user created it. This will allow you to restrict updates to that instance to only be done by the user who created it. You can do so by setting a ForeignKey field on the Model which references your project's user model.

```python
"""
The user model can be imported as the AUTH_USER_MODEL from your settings
"""
from django.conf import settings

class Garden(models.Model):
    name = models.CharField(max_length=100)
    gardener = models.ForeignKey(settings.AUTH_USER_MODEL)
```

## Syncing to the Database

Once we have created our models, we need a way to make sure that our database contains the required tables. To do this, Django uses migrations. First, we use `makemigrations` to create files (one per app) describing what the table should look like. Once we have done that, we use `migrate` to actually create and alter tables.

```bash
python manage.py makemigrations
python manage.py migrate
```

Any time that a model is altered, makemigrations and migrate should be re-run.

Whenever a new field is added to a model, we have to take into consideration how this will affect rows that already exist in the related table. To deal with this, new fields should either be provided a default value or be allowed to be null (null=True).

## Model Methods

There are a number of class methods that you can implement on your Models that Django will look for when calling certain functions. While these aren't required, they are useful to add to your models.

```python
class Vegetable(models.Model):
    name = models.CharField(max_length=100)

    def __str__(self):
        """
        You might want to add a __str__ method to your model to give a
        prettier representation of your model when str is called on it.
        This is most useful when you are working in the REPL and
        in admin pages.
        """
        return '{}'.format(self.name)

    def get_absolute_url(self):
        """
        get_absolute_url should return a string of the url that is
        associated with that particular model instance. This is
        especially useful in templates and redirect responses.
        """
        return reverse('vegetable', pks={pk: self.pk})
```

## Meta Class

The Meta class of a Model allows you to set properties of the class that aren't a Field of the Model.

[documentation](https://docs.djangoproject.com/en/2.0/ref/models/options/)

```python
class Vegetable(models.Model):
    name = models.CharField()
    weight = models.IntegerField()

    class Meta:
        """
        Use an app_label in lieu of the model's app
        being included in your INSTALLED_APPS
        """
        app_label = 'misc_app'
        """
        Specify the name of the table instead of using
        the default app_model pattern
        """
        db_table = 'veggies'
        """
        Specify the order of the instances returned from the database
        by field name. The default order is ascending, a '-' at the
        beginning of the string is descending, and a '?' is random
        ordering.
        Ordering can be done on multiple fields, which will be
        applied in order.
        """
        ordering = ['-weight']
        """
        The singular name of the model can be set. If not set,
        uses lower case and inserts spaces between words (as
        determined by upper case letters)
        """
        verbose_name = 'veg'
        """
        The plural name can also be set. The default plural verbose
        just adds an 's' to the end of the verbose_name
        """
        verbose_name_plural = 'veggies'
```
