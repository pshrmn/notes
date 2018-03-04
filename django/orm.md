# Django ORM

To simplify the flow of data between your project and the database, Django provides an ORM (Object-Relational Mapper) that can handle the getting and setting of data from the database.
## Basics

To add a row to a table, begin by creating an instance of a Model

```python
from vegetable.models import Vegetable

# an instance of the Vegetable model
cuke = Vegetable(name='cucumber', ...)
```

This has not been synced with the database, so it has no associated row in the database and therefore its primary key has not been created. This can be rectified by calling the instance's `save` method. When we call the save method, the data for the instance will be used to insert a row into the associated table in the database.

```python
cuke.save()
```

When we update data in the instance, the `save` method will need to be called again in order to update the row in the database.

```python
cuke.weight = 10
cuke.save()
```

## QuerySets

In order to get data from the database, we need to use QuerySets. A QuerySets is a set of instances that correspond to rows in a table in your database. A `QuerySet` has methods that are used to interact with the database in order to select the desired rows.


### Manager

Each Model has at least one associated Manager. A Manager is in charge of making a base selection to the database, which will return a QuerySet. For the default Manager, this will select all of the rows in the associated table. You can create more specific Managers whose base selection is more specific. The default Manager is accessed through the `objects` attribute of the Model.

```python
from vegetable.models import Vegetable
from django.db import models

isinstance(Vegetable.objects, models.Manager) == True
```

Whenever you want to access the database with the ORM, you will do it through a Manager. They are further explained in the [Managers]("#managers) section.

### Evaluating

`QuerySet`s are not evaluated immediately, so you don't have to worry about hitting the database unnecessarily when chaining methods together.

```python
"""
database hits are triggered by:
"""

# iterating over a QuerySet
for row in Vegetable.objects.all():
  print(row.name)

# slicing a QuerySet with a step parameter
odd_vegetables = Vegetable.objects.all()[1::2]

# pickling a QuerySet
import pickle
pickled = pickle.dumps(Vegetable.objects.all())

# or calling repr, len, list, or bool on the QuerySet
qs = Vegetable.objects.all()
as_list= list(qs)
# NOTE: don't actually do this, use the .count method
count = len(qs)
```

### Methods

`QuerySet`s have two types of methods: those that return another QuerySet and those that return something else.


#### Methods that Return QuerySets

[documentation](https://docs.djangoproject.com/en/2.0/ref/models/querysets/#methods-that-return-new-querysets)

These `QuerySet` methods take keyword arguments which Django refers to as field lookups. Their return values are also QuerySets so they can be chained together to build complex queries. 

##### `filter`
```python
"""
filter selects rows that match the field lookups. Multiple
keyword arguments can be passed to the filter. Doing this
is the same as adding an AND statement in SQL
"""
greenies = Vegetable.objects.filter(color__eq='green')
big_greenies = Vegetable.objects.filter(
    color__eq='green',
    weight__gte=15
)
```

##### `exclude`
```python
"""
exclude does the opposite of filter. Any rows that match the
lookup conditions are excluded from the QuerySet.
"""
no_purple = Vegetable.objects.exclude(color__eq='purple')
```
      
##### `all`
```python
"""
all makes a copy of the QuerySet.
"""
veggies = Vegetable.objects.all()
```

These methods can be chained together to form complex queries.

```python
Vegetable.objects.filter(color__eq='green').exclude(weight__gt=15)
```

#### Methods That Return Other Values

There are also some `QuerySet` methods which return something other than a QuerySet.

[documentation](https://docs.djangoproject.com/en/2.0/ref/models/querysets/#methods-that-do-not-return-querysets)

##### `get`
```python
"""
get returns the instance of the one row in the table that
matches the conditions. If more than one row is selected by
the query, a MultipleObjectsReturned exception is raised.
"""
v = Vegetable.objects.get(pk=1)
```

##### `create`
```python
"""
You can create an instance and save it in the same method call
using create. The method returns the instance.
"""
potato = Vegetable.objects.create(name="potato")
```

##### `get_or_create`
```python
"""
Sometimes you may not know if a row already exists in your database.
If it does exist, you want an instance of it and if it doesn't you want
to create a row for it and return that instance.

get_or_create lets you do just that. The method returns a tuple where the
first value is the instance and the second is a boolean indicating if
the accompany row in the database was just created. 
"""
cucumber, is_new = Vegetable.objects.get_or_create(name="cucumber")
```
   
##### `count`
```python
"""
count returns the number of rows that the QuerySet selects.
All of the work for this is done on the database, so it is more
efficient than calling len on a QuerySet that selects all of
the rows in a table.
"""
veggie_count = Vegetable.objects.count()
```

##### `aggregate`
```python
"""
aggregate lets you use aggregating SQL functions such as Avg, Max, Min, and Sum
"""
from django.db.models import Avg, Max, Min, Sum

avg_weight = Vegetable.objects.all().aggregate(Avg('weight'))
"""
The aggregate function returns a dict with the aggregate values. The keys
are the field name, two underscores, and the aggregate function

avg_weight = {'weight__avg': 15}
"""

big_and_small = Vegetable.objects.all().aggregate(Min('weight'), Max('weight'))
"""
multiple aggregate functions can be included in one call

big_and_small = {'weight__min': 2, 'weight__max': 30}
"""

total_weight = Vegetable.objects.all().aggregate(total_weight=Sum('weight'))
"""
The aggregate function can be provided as a keywork argument and the keyword
will be used as the key in the output dict.

total_weight = {'total_weight': 234}
"""
```

### Chaining Methods
```python
"""
QuerySet methods that return QuerySets can be chained together.
"""
greenies = Vegetable.objects.filter(color__eq='green')
heavy_greenies = greenies.filter(weight__gte=15)

"""
but you can also include multiple field lookups in one method
if you just want to have an AND clause in your query
"""
green_and_heavy = Vegetable.objects.filter(color__eq='green', weigth__gte=15)
```

### Field Lookups

[documentation](https://docs.djangoproject.com/en/2.0/topics/db/queries/#field-lookups)

Field lookups are done using keyword arguments to specify the table column and the match type. The keyword is a concatenation of these values, in the form `<column_name>__<condition>`. You can also do lookups using just the column name, which is interpreted as an exact match condition.


##### `exact` and `iexact`
```python
"""
Specify that the value should be matched exactly in the table.
"""
cukes = Vegetable.objects.filter(name__exact="cucumber")

"""
A lookup using just the field name is also interpreted as an exact lookup
"""
taters = Vegeatble.objects.filter(name="potato")

"""
iexact is the case insensitive version of exact
"""
cukes = Vegetable.objects.filter(name__iexact="cUcUmBeR")
```

##### `lt`, `lte`, `=`, `gte`, `gt`
```python
"""
numerical comparisons can be made with:
"""

# lt for less than
small_veg = Vegetable.objects.filter(weight__lt=5)

# lte for less than or equals
smallish_veg = Vegetable.objects.filter(weight__lte=7)

# = for equals
exact_veg = Vegetable.objects.filter(weight=10)

# gte for greater than or equals
biggish_veg = Vegetable.objects.filter(weight__gte=15)

# gt for greater than
big_veg = Vegetable.objects.filter(weight__gt=20)
```

## Managers

```python
"""
Managers are the middle men between your model's class and the database.
Django automatically creates a Manager for every model which can be accessed
as the "objects" attribute of the model.
"""
class Vegetable(models.Model):
    name = models.CharField(max_length=100)
# Manager = Vegetable.objects

"""
The Manager class shares a number of attributes with a QuerySet and any
queries you make to a model will start with the manager. For example,
if you want to get all of the vegetables that are red, you would call
filter on the manager:
"""
red_veggies = Vegetables.objects.filter(color='red')

"""
The Manager can be renamed. This is useful if you want to have
an objects field on your model, or just don't like the "objects" name.
"""
class Vegetable(models.Model):
    name = models.CharField(max_length=100)
    gardeners = models.Manager()
# Manager = Vegetable.gardeners

"""
The default Manager creates a QuerySet that selects every row
in the table. You can create your own Manager that selects a more
specific set of rows by overriding the get_queryset method
of the Manager.

Using a customer manager will prevent the default manager from being
added. If you want the default manager in addition to your custom
one, add it manually.
"""
class GreenManager(models.Manager):
    # add the use_in_migrations attribute so that the manager can
    # be used in your models migrations
    use_in_migrations = True

    def get_queryset(self):
        return super().get_queryset().filter(color='green')

class Vegetable(models.Model):
    name= models.Charfield(max_length=100)

    # the order the managers are provided in determines
    # which one is the base manager (the first one)
    objects = models.Manager()
    greenies = GreenManager()
```