# Django Users

## Settings

Django uses settings to set user authentication behavior and to keep track of the User model across apps.

### User Model

The `User` model that your project uses is stored using the `AUTH_USER_MODEL` setting. If you are using the default User, you don't have to set this.

[documentation](https://docs.djangoproject.com/en/2.0/ref/settings/#auth-user-model)

```python
AUTH_USER_MODEL = 'auth.User'
```

If you ever need to refer to the User model that your project is using, you can import the settings from django.conf. This is useful for declaring a foreign key dependency on a model.

```python
from django.conf import settings

class Dog(models.Model):
  
    name = models.CharField()
    human = models.ForeignKey(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE
    )
```

### Password Validators

The `AUTH_PASSWORD_VALIDATORS` settings is used to specify which password validators should be used when registering an account.


Django provides a number of password validators to set restrictions on password values. These are:      

* `UserAttributeSimilarityValidator` checks if the user's password is similar to their username, first name, last name, or email using difflib's SequenceMatcher.
* `MinimumLengthValidator` enforces a minimum password length.
* `CommonPasswordValidator` checks the password against a list of common passwords, rejecting the password if it matches any common ones.
* `NumericPasswordValidator` enforces that the password isn't made up entirely of numbers.

Any of the password validators that you want to use are provided as dicts to the `AUTH_PASSWORD_VALIDATORS` list.

[documentation](https://docs.djangoproject.com/en/2.0/topics/auth/passwords/#password-validation)

```python
AUTH_PASSWORD_VALIDATORS = [
    {
        # the path to the validator
        'NAME': 'django.contrib.auth.password_validation.MinimumLengthValidator',
        # any keyword args to be passed to the validator's __init__ method
        'OPTIONS': {
            'min_length': 9,
        }
    },
]
```

You can also write your own password validators and include them in the `AUTH_PASSWORD_VALIDATORS` list.

```python
# path/to/validators.py
class LowerAndUpperValidator(object):
    # the validate method is used to test the password
    def validate(self, password, user=None):
        lower = re.compile(r'[a-z]')
        upper = re.compile(r'[A-Z]')
        has_lower = lower.search(password)
        if has_lower is None:
            raise ValidationError(
                'The password must include at least one lowercase letter',
                code='password_no_lower'
            )
        has_upper = upper.search(password)
        if has_upper is None:
            raise ValidationError(
                'The password must include at least one uppercase letter',
                code='password_no_upper'
            )

    # the get_help_text method is used to add a help message to the
    # password field's help texts
    def get_help_text(self):
        return ('Your password must include at least one lowercase and one'
                'uppercase letter')

# settings.py
AUTH_PASSWORD_VALIDATORS = [
    # any other validators...,
    {
        'NAME': 'path.to.validators.LowerAndUpperValidator'
    }
]
```

## Built-in Users

Django comes with a built-in user model that is "good enough" for most cases.

### Built-in Settings
```python
"""
As mentioned above, the built-in auth.User model is set
by default as the User model, so you don't need to set it
if you are using it.
"""
AUTH_USER_MODEL = 'auth.User'
```
  
### Built-in User Model

The default User model extends the AbstractUser model. The AbstractUser model extends the AbstractBaseUser and PermissionsMixin models.

The default User has the following fields:

```python
from django.contrib.auth.models import User

"""
AbstractUser fields:
"""
# the user's username. This is what is used to login and so must be unique.
# valid characters are letters, numbers, @, ., +, -, and _
username = models.CharField('username', max_length=150, unique=True,
)
first_name = models.CharField('first name', max_length=30, blank=True)
last_name = models.CharField('last name', max_length=30, blank=True)
email = models.EmailField('email address', blank=True)
# if True, user can access admin site
is_staff = models.BooleanField('staff status', default=False)
# if True, account is active. Useful when authentication is required
# or fake deleting account
is_active = models.BooleanField('active', default=True)
date_joined = models.DateTimeField('date joined', default=timezone.now)

"""
PermissionsMixin fields:
"""
is_superuser = models.BooleanField(default=False,)
groups = models.ManyToManyField(Group, blank=True)
user_permissions = models.ManyToManyField(Permission, blank=True)

"""
AbstractBaseUser fields:
"""
password = models.CharField('password', max_length=128)
last_login = models.DateTimeField('last login', blank=True, null=True)

"""
The two most important fields are username and password, as they are what will be
used to authenticate the user during login.

The email field is also required for user verification. If the user doesn't
provide an email address, there will be no way to reset a forgotten
password (without leaving yourself vulnerable to phishing).
"""
```

### Built-in User Forms

Having a User model is great, but you also need a way to create, authenticate, and update Users.

#### Creating Users
```python
"""
The UserCreationForm provides three fields:
username, password1, and password2 (which should be the same as password1)
"""
from django.contrib.auth.forms import UserCreationForm

"""
When a user attempts to create a new User, the backend will validate
that password1 and password2 two are not empty and are both equal. Any
additional settings.AUTH_PASSWORD_VALIDATORS you have will also be run
against the password.

If you want any other fields included in the form, you can extend
the UserCreationForm
"""

class CustomUserCreationForm(UserCreationForm):

    class Meta(UserCreationForm.Meta):
        model = User
        fields = UserCreationForm.Meta.fields + ('email',)
```

#### Authenticating Users
```python
"""
The authentication form requires authentication fields, which for the
built-in User is a username and password.
"""
from django.contrib.auth.forms import AuthenticationForm
```
   
#### Updating Users
```python
"""
Django has built-in forms for resetting andupdating a User's
password, but for other fields, you should create your own forms.
"""
from django.fields import ModelForm

class UpdateUserForm(ModelForm):
    class Meta:
        fields = ['first_name', 'last_name']
```
   
#### Forms In Views
```python
"""
The forms should then be used in views just like any other form
"""

# class-based views
class RegistrationView(CreateView):

    model = User
    form_clas = UserCreationForm

# using built-in auth views
from django.contrib.auth.views import login
def login_user(request):
    return login(request, authentication_form=AuthenticationForm)

# or view functions
def update_user(request):
    if request.method == 'POST':
        form = UpdateUserForm(request.POST, instance=request.user)
        if form.is_valid():
          form.save()
          response = # create a response
          return response
    elif request.method == 'GET':
        form = UpdateUserForm()
    return TemplateResponse(request, template_path, {form: form})
```

## Custom Users

For a variety of reasons, you might want to create your own custom user model. Some reasons might include wanting to be more (or less) restrictive on username values or preferring to use a different field (such as email) as the user's "username".

If you plan on using your own User model, do so at the beginning of your project. makemigrations will not create migrations when you change the User model, forcing you to make any changes to the data and/or database yourself.

In this example a CustomUser will be created in an app named "custom". CustomUser will have the following fields: an email address (as the user identifier), a name, a join date, and booleans to identify if the user is active and is a staff member (allowed to access admin site).

For a full example of a custom user, check out this [example](https://docs.djangoproject.com/en/2.0/topics/auth/customizing/#a-full-example) in the customizing authentication documentation.


### Custom Settings
```python
"""
You need to identify your auth model in your settings.
"""
AUTH_USER_MODEL = 'custom.CustomUser'
    </code></pre>
  </div>
  <div id="custom-models">
    <h3>Custom User Model</h3>

    <div>
      <h4>Model</h4>
      <pre><code class="language-python">
from django.contrib.auth.models import AbstractBaseUser, PermissionsMixin
from django.db import models

"""
For our custom user model, we should start by inheriting from
the AbstractBaseUser and PermissionsMixin classes
"""
class CustomUser(AbstractBaseUser, PermissionsMixin):

    email = models.EmailField(unique=True)
    name = models.CharField(max_length=50)
    join_date = models.DateField(auto_now_add=True)

    is_active = models.BooleanField('active', default=True)
    is_staff = models.BooleanField(default=False)

    # specify the Manager for the User
    objects = CustomUserManager()

    """
    fields inherited from AbstractBaseUser:
    password = models.CharField(_('password'), max_length=128)
    last_login = models.DateTimeField(_('last login'), blank=True, null=True)
    """

    """
    fields inherited from PermissionsMixin:
    is_superuser = models.BooleanField(default=False)
    groups = models.ManyToManyField(Group, blank=True, related_name="user_set",
                                    related_query_name="user")
    user_permissions = models.ManyToManyField(Permission, blank=True,
                                              related_name="user_set",
                                              related_query_name="user")
    """

    # specify the 'username' field of the model, must be unique
    # this is required by AbstractBaseUser
    USERNAME_FIELD = 'email'
    # list of fields required when creating a superuser
    REQUIRED_FIELDS = ['name',]

    """
    AbstractBaseUser requires get_full_name and get_short_name
    to be implemented by the subclass
    """
    def get_full_name(self):
        return self.email

    def get_short_name(self):
        return self.get_full_name()

    """
    The model will have access to a number of methods inherited
    from AbstractBaseUser, including:
    """
    # Checks if the user is logged in, always returns True.
    # (an AnonymousUser's is_authenticated method returns False). 
    def is_authenticated(self):
        return True

    # use this to set the class's password attribute as the
    # salted and hashed version fo the raw password
    def set_password(self, raw_password):
        # ...

    # given a raw password, salt and hash it, checking the returned
    # value against the known salted and hashed output. The setter
    # function is only used when the stored password value needs to
    # be updated (for example, changing the hashing function)
    def check_password(self, raw_password):
        # ...

    """
    and methods from PermissionsMixin, including:
    """

    # check if a user has permissions
    def has_perm(self, perm, obj=None):
        # ...
```
   
#### Manager
```python
"""
You also need to specify the Manager that your custom user model uses.
The built-in UserManager can be used if your User model has the fields:
username, email, is_staff, is_active, is_superuser, last_login, and
date_joined.

If not, you need to create a Manager that inherits from BaseUserManager.
The custom manager will need to implement create_user and create_super_user
methods.
"""
from django.contrib.auth.models import BaseUserManager


class CustomUserManager(BaseUserManager):

    use_in_migrations = True
  
    """
    create_user will be used when creating a regular user
    """
    def create_user(self, email, name, password, **extras):
        if not email:
            raise ValueError('email is required')
        # BaseUserManager.normalize_email converts domain to lowercase
        user = self.model(
            email=self.normalize_email(email),
            name=name,
            **extras
        )
        # set_password will take take of the hashing
        user.set_password(password)
        user.save()
        return user

    """
    create_superuser will be used when creating a superuser
    """
    def create_superuser(self, email, name, password, **extras):
        if not email:
            raise ValueError('email is required')
        user = self.model(
            email=self.normalize_email(email),
            name=name,
            **extras
        )
        user.set_password(password)
        # make sure the user is staff and a superuser
        user.is_staff = True
        user.is_superuser = True
        user.save()
        return user
```

### Custom User Forms
```python
"""
the CustomUser should have forms for both creating a new CustomUser
and updating an existing CustomUser
"""
from django import forms
from django.contrib.auth.forms import UserCreationForm

from .models import CustomUser

class CreateCustomUserForm(UserCreationForm):

    class Meta:
        model = CustomUser
        fields = ('email', 'name')


class CustomUserChangeForm(forms.ModelForm):
    class Meta:
        model = CustomUser
        fields = ('email', 'name')
```
  
### Custom User Admin
```python
"""
In order to properly integrate with the admin site,
the custom user has to have forms for:
1. Creating a user
2. Changing (all fields for) a user

We can use the CreateCustomUser from our forms, but the
CustomUserChangeForm only allowed us to update the email and name
so we can't use that. Instead, we can use the auth.form's
UserChangeForm which will include all of our fields.
"""

from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from django.contrib.auth.forms import UserChangeForm

from .models import CustomUser
from .forms import CreateCustomUserForm


class CustomUserAdmin(BaseUserAdmin):


    """
    The following attributes may need to be overridden if your User's
    fields don't match up with the built-in User:
    form
    add_form
    change_password_form
    list_display
    list_filter
    fieldsets
    add_fieldsets
    search_fields
    ordering
    filter_horizontal
    """
    form = UserChangeForm
    add_form = CreateCustomUserForm
    # don't need to override change_password_form

    list_display = ('email', 'name', 'is_staff', 'is_superuser')
    list_filter = ('is_staff',)
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal info', {'fields': ('name',)}),
        ('Permissions', {'fields': ('is_staff', 'is_superuser')}),
    )
    # add_fieldsets is not a standard ModelAdmin attribute. UserAdmin
    # overrides get_fieldsets to use this attribute when creating a user.
    add_fieldsets = (
        (
            None,
            {
                'classes': ('wide',),
                'fields': ('email', 'name', 'password1', 'password2')
            }
        ),
    )
    search_fields = ('email',)
    ordering = ('email',)
    filter_horizontal = ()

admin.site.register(CustomUser, CustomUserAdmin)
```
