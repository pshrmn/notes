# Django Forms

Forms turn the one way street of information flow from your site to the user into a two way street where the information flows between the user and your site.

## Creating Forms

### Form Definitions

Django's forms are similar to its models. They inherit from a base Form class and you define fields on them. If you have a model that you want a user to be able to create an instance of, you should create a form that corresponds to that model.      

In our Model examples, we had vegetables and gardens. While we may not want our users to do any genetic engineering creating new types of vegetables, they should be able to make their own garden. To do this, we can create a GardenForm which has fields corresponding to our Garden model (ie. the names of the Form's fields should be the same as the Model's)

```python
from django import forms

class GardenForm(forms.Form):
    # forms have fields that correspond to model fields
    name = forms.CharField(max_length=100)
```

Forms don't have to have related models, though. For example, you might have a contact form that will trigger sending an email. If that information never hits your database, you don't need a model for it, but you do need a form to get the data from the user.

```python
class ContactForm(forms.Form):
    name = forms.CharField(max_length=100, label='Your Name')
    email = forms.EmailField(label='Contact Email')
    subject = forms.CharField(max_length=100)
    message = forms.CharField(max_length=10000)
```

### Form Fields

Django forms are made up of fields. Each Field on your model has a number of optional arguments. Some of the options exist on every field, while others are field dependent.

[documentation](https://docs.djangoproject.com/en/2.0/ref/forms/fields/)

#### Arguments

```python
# Arguments on every field:

"""
required (default=True)
By default, all of the fields in your form are required. If you have
a field that isn't required (eg the model's field has null=True), pass
the required=False argument.
"""
description = forms.CharField(required=False)

"""
label
When your field is displayed to the user in a form, it should have
a label so that the user knows what data the field wants. By default,
django will create a label by capitalizing the first letter and replacing
any underscores with spaces (your_field becomes 'Your field'). If you
want a different label, you can supply it in the label argument.
"""
description = forms.CharField(label='Desc.')

"""
label_suffix (default=':')
The label suffix is placed between the label and the input. If you don't
want it to be a colon, you can pass in a different value.
"""
description = forms.CharField(label_suffix='')

"""
initial
If you want your field to have an initial value, provide it using the
initial argument. The corresponding input element will have its value
set to the initial value.
"""
description = forms.CharField(initial='...')

"""
help_text
Help text is useful for providing a user clues about what type of data
if valid for a field.
"""
temperature = forms.FloatField(help_text='in Celsius')

"""
disabled
If you want a form field to be present, but not editable by the User,
make it disabled. Even if the user alters the data, the value that they
submit will be ignored. Disabled fields should have an initial value
set otherwise they aren't particularly useful.
"""
favorite_color = forms.CharField(disabled=True, initial='blue')

"""
validators
While each Field type validates data in its own way, you might want
to add custom validators. This allows you to more strictly validate
the data than what is done by default. Validators is a list so that
you can do multiple validation tests.
"""
from django.core.exceptions import ValidationError

def validate_color(color):
    if color == 'brown':
        raise ValidationError('No one\'s favorite color is brown')

favorite_color = forms.CharField(validators=[validate_color])


"""
widget
Each of Django's built-in form fields has a widget associated
with it. For example CharFields use <input type="text">
and IntegerFields use <input type="number">. Generally speaking
you won't have to specify a field's widget, but if you want to
use something other than the default widget, you can.

https://docs.djangoproject.com/en/2.0/ref/forms/widgets/
"""
BREEDS = (
  (0, 'Pembroke Welsh Corgi'),
  # ...
)
# the default widget for a ChoiceField is a forms.Select
dog_breed = forms.ChoiceField(choices=BREEDS, widget=forms.RadioSelect)
```

#### ChoiceField

One field type that doesn't directly correspond to a model field is the `ChoiceField`. When a model field has a limited number of possible choices, you don't want your form to accept anything but the possible choices. A `ChoiceField` requires you to pass it a `choices` argument, which is an iterable of (database_value, user_value) tuples, the same as it is for model fields.

```python
CHOICES = (
    ('C', 'Celsius'),
    ('F', 'Fahrenheit'),
    ('K', 'Kelvin')
)
temperature = forms.ChoiceField(choices=CHOICES)
```

The default widget used by a ChoiceField is Select, which will create a `<select>` element with `<option>` choices. If you want to use `<input type="radio">` elements instead, you should use the `RadioSelect` widget.

```python
temperature = forms.ChoiceField(choices=CHOICES, widget=forms.RadioSelect)
```

If you want to be able to select multiple choices, use a `MultipleChoiceField`. This uses a `SelectMultiple` by default, but you can also use a `CheckboxSelectMultiple`.

```python
FLAVORS = (
    ('V', 'Vanilla'),
    ('C', 'Chocolate'),
    ('S', 'Strawberry')   
)
scoops = forms.MultipleChoiceField(choices=FLAVORS)
```

## Form Data

Forms exist to receive and validate data.

### Form Instances

An instance of a form is created by calling the form. It can take a number of arguments, but you can also create an empty form by instantiating the form class with no arguments.

```python
# empty form
gf = GardenForm()

"""
data
a dict containing the data for the form. The keys
in the dict should correspond to the field attributes
of the form.
"""
gf = GardenForm(data={
  "name": "Green Valley"
})
"""
This is the first keyword argument of the form, so it can
easily be passed without providing the keyword
"""
gf = GardenForm({
  "name": "Green Valley"
})

"""
files
Sometiems a form will include a field for which data has
to be uploaded. That uploaded data will be included in the
request.FILES attribute and can be included in the form
with the files option
"""
uf = UploadedForm(files=request.FILES)

"""
initial
initial data can be provided for fields. This data will
only be used if the field is disabled.
"""
gf = GardenForm(initial={
  "name": "Garden No. 1"
})
```

### Validating Form Data

You should never blindly trust the user's input and this is especially true with forms that will insert data into a database. Django attempts to help you with this by cleaning the form's data and checking it for any errors. You can check that a form's data is valid by calling its `is_valid` method.

```python
good = form.is_valid()
```

The `is_valid` method does two things:

1. Check if the form is bound (i.e., the form has been given data)     
2. Verify that the form has no errors by checking the form's `errors` attribute. Internally, the form maintains an `_errors` attribute which is `None` when the form hasn't been cleaned and validated and a dict when it has. The form's `errors` attribute is actually a property, so when accessing it, it looks at at the `_errors` attribute. If it is `None`, that means that the form hasn't been cleaned, so it calls its `full_clean` method to do so before returning the `_errors` attribute.

So what is a full clean? A full clean coerces and validates the data passed to the form by following these steps:

1. Clean Fields:

For every field in the form:        

  1. Call its `clean` method. The clean method does the following:
    1. Its data is coerced to its desired data type by calling the field's `to_python` method (eg. `IntegerField` coerced to int) and raise a `ValidationError` if data cannot be coerced.
    2. Calls its `validate` method. What this does will depend on the field's type, but if it fails, it will raise a `ValidationError`.
    3. Finally, the field calls its `run_validators` method. This iterates over the fields list of validators (made up of default validators and those passed in using the field's `validators` option) and creates a list of all validation errors. If, after all of the validator functions have been run, the list of errors is not empty, a `ValidationError` is raised, which is passed the list of errors.
    4. The coerced data is returned.
  2. The value returned by the field's `clean` method is added to the form's `cleaned_data` dict using the attribute name of the field.
  3. If the form has a `clean_<field name>` method where `<field_name>`; is the name of the current field attribute, call it. If that method raises a `ValidationError` it is added to the form's errors, otherwise the `cleaned_data` value for that field is updated with the return value of the `clean_<field_name>` method.
  4. If any of the above steps raised a `ValidationError`, when that error is caught its value is added to the form's `_errors` dict.

2. Clean Form:

During this step extra cleaning on the form can be completed by calling the form's `clean` method. In the default case, that method does nothing.

3. Post Clean:

Perform even more cleaning if necessary after fields and form have been cleaned. In the default case, this step does nothing.

Once the `full_clean` method has returned, any errors in the forms data should have been caught. The `errors` property will return the `_errors` value.

If any errors were set during the `full_clean` method call, `is_valid` will return `False`. Otherwise, it returns `True` and the form's data should be considered clean, valid, and ready to be put into your database (or have any other desired action done on it).

### Handling Requests

Typically forms instances will be created using data that comes from a request.

```python
from .forms import GardenForm
from .models import Garden

@login_required
def form_view(request):
    if request.method == 'POST':
        # on the post request, fill in the form with the post data
        form = GardenForm(request.POST)

        # if the form is valid, create a new instance and redirect
        # otherwise the invalid form will be returned in the response
        if form.is_valid():
            # create the garden using the cleaned data
            g = Garden(**form.cleaned_data)
            g.gardener = request.user
            g.save()
            # redirect to the new garden using the url from
            # the Garden model's get_absolute_url method
            return redirect(g)

    if request.method == 'GET':
        # on the get request, serve an empty form
        form = GardenForm()
    # our form is provided to the template in the context object
    return render(request, '/gardens/template.html', {'form': form})
```

## `ModelForms`

[documentation](https://docs.djangoproject.com/es/2.0/topics/forms/modelforms/)

Creating a Form class based on a Model class where you have to match all of the fields might seem like unnecessary work, and it is. Instead Django provides the ModelForm class, which allows you to map a Model class to a Form.


### `ModelForm` Class

```python
from django import forms

from .models import Garden

# Your ModelForm will inherit from ModelForm
class GardenForm(forms.ModelForm):

    # you can include any additional fields that are not part of
    # the model
    find_out = forms.ChoiceField(
        choices=CHOICES,
        label='Source',
        help_text='How did you find out about this site?'
    )

    # attributes are set in the Meta class
    class Meta:
        # you need to specify which model the form is based on
        model = Garden

        # you can pick which of the models fields to include
        fields = ['name']
        # or specify __all__ to include all fields
        fields = '__all__'
        # likewise you can select which fields to exclude
        exclude = ['gardener']

        # labels can be provided in a dict with keys as field names
        labels = {
            'name': 'Garden Name'
        }

        # as can help texts
        help_texts = {
            'name': 'The name of the garden'
        }

        # and also error messages. Each value is also a dict where its
        # keys are the code of the error.
        error_messages = {
            'name': {
                'max_length': 'How posh are you? That name is way too long.'
            }
        }

        # if you want to change a field's widget, do so in the widgets dict
        widgets = {
            'name': forms.Textarea()
        }
```

### Working With a `ModelForm` Instance

ModelForm instances are created the same way that Form instances are.

```python
# from a dict
form = GardenForm({'name': 'Babylon'})
# or a QueryDict
form = GardenForm(request.POST)
```

When you create a `ModelForm` instance, you can include an `instance` argument whose type is `ModelForm`'s Model. If you do this, when you save the `ModelForm`, it will update the instance instead of creating a new one.

```python
g = Garden.objects.get(pk=0)
# g.name = 'Bill\'s Garden'
form = GardenForm({'name': 'William\'s Garden', instance=g)
# g.name = 'Bill\'s Garden'
g.save()
# g.name = 'William\'s Garden'
```

If you don't pass in an instance, an empty instance of the `ModelForm`'s Model class will be created. Then, when the form is validated, the instance will be populated with the form data.

```python
form = GardenForm(request.POST)
# form.instance is empty Garden instance
good = form.is_valid()
# form.instance now has values set from data
```
  
#### `save`

`ModelForms` have a `save` method which validates the form data. If the validated form has no errors, the ModelForm creates an instance of the `ModelForm`'s Model using the form data, saves it to the database, and returns the instance.

```python
form = GardenForm(request.POST)
garden = form.save()
```

`save` can take a `commit` keyword argument. By default it is `True`, but if `commit=False`, the instance is not saved to the database. This can be useful if you want to do something to the instance of the Model before it hits the database.


```python
form = GardenForm(request.POST)
garden = form.save(commit=False)
# do something with the garden...
garden.save()
```

### Handling Requests

Working with requests using ModelForms is similar to working with requests using Forms. Below is a replication of the Form request handling example above, only done using ModelForms.

```python
from .forms import GardenForm
from .models import Garden

@login_required
def form_view(request):
    if request.method == 'POST':
        form = GardenForm(request.POST)
        # if the form is valid, save it
        if form.is_valid():
            # here, the ModelForm can handle creating and saving
            # our Garden instance for us
            form.save()
            return redirect(form.instance)
    if request.method == 'GET':
        form = GardenForm()
    return render(request, '/gardens/template.html', {'form': form})
```
