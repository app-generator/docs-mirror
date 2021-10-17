A simple way to code a reset password mechanism in Django

## How Django assist developers on this feature

Django comes with a password reset feature. You can extend these features to modify the default behavior.

For example, the views `from django.contrib.auth` comes with support for login, logout, and all the password reset flow.

The password reset flow usually looks like this : 
- requesting password change by providing an email;
- send a mail containing a link to change the password;
- click on the link and change the password;

## Configuration

As I said earlier, Django provides views you can use for password resetting flow.

In your `urls.py` file, add these lines: 

```python
from django.contrib.auth import views

urlpatterns = [
    path('password_reset/', views.PasswordResetView.as_view(), name="password_reset"),
    path('reset/<uidb64>/<token>/', views.PasswordResetConfirmView.as_view(), name="password_reset_confirm"),
    path('password_reset/done/', views.PasswordResetDoneView.as_view(), name="password_reset_done"),
    path('reset/done/', views.PasswordResetCompleteView.as_view(), name="password_reset_complete"),
    path('activate/<uidb64>/<token>/', activate_account, name='activate')
]
```

By default, these views come with their own HTML templates. We can even add some options, like the `form_class`. For example, we want to return an error if the email the user has entered doesn't exist. This will be done on the `password_reset` URL.

```python
class EmailValidationOnForgotPassword(PasswordResetForm):
    def clean_email(self):
        email = self.cleaned_data['email']
        try:
            Profile.objects.get(email=email)
        except ObjectDoesNotExist:
            raise ValidationError("No user exists with this email")

        return email
```

And then accordingly modify the `urls.py` file.

```python
from django.contrib.auth import views
from .forms import EmailValidationOnForgotPassword

urlpatterns = [
    path('password_reset/', views.PasswordResetView.as_view(form_class=EmailValidationOnForgotPassword),
         name="password_reset"),    ...
]
```

## Customize the UI templates

But how do you use your own templates?

Well, just create a directory named `registration` in your `templates` directory.

Let's create a custom template for the `views.PasswordResetView` class-view. When we give a look at the code of the view, the name of the template is used.

```python
class PasswordResetView(PasswordContextMixin, FormView):
    ...
    template_name = 'registration/password_reset_form.html'
    ...
```

Great! Inside the registration directory, we've created earlier, create a file named `password_reset_form.html`. 

```html
{% extends "layouts/base-fullscreen.html" %}

{% block title %} Forgot Password {% endblock %}

<!-- Specific Page CSS goes HERE  -->
{% block stylesheets %}{% endblock stylesheets %}

{% block content %}

    <main>
        <section class="vh-lg-100 mt-5 mt-lg-0 bg-soft d-flex align-items-center">
            <div class="container">
                <div class="row justify-content-center form-bg-image">
                    <p class="text-center"><a href="./sign-in.html" class="d-flex align-items-center justify-content-center">
                        <svg class="icon icon-xs me-2" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg"><path fill-rule="evenodd" d="M7.707 14.707a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414l4-4a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l2.293 2.293a1 1 0 010 1.414z" clip-rule="evenodd"></path></svg>
                        Back to log in
                        </a>
                    </p>
                    <div class="col-12 d-flex align-items-center justify-content-center">
                        <div class="signin-inner my-3 my-lg-0 bg-white shadow border-0 rounded p-4 p-lg-5 w-100 fmxw-500">
                            <h1 class="h3">Forgot your password?</h1>
                            <p class="mb-4">Don't fret! Just type in your email and we will send you a code to reset your password!</p>
                        {% if form.errors %}
                                <div>
                                    {% for key, value in form.errors.items %}
                                        <p class="mb-4">{{ value }}</p>
                                    {% endfor %}
                                </div>
                        {% endif %}
                            <form action="#" method="POST">
                                {% csrf_token %}
                                {% for field in form %}
                                    <!-- Form -->
                                <div class="mb-4">
                                    <label for="email">{{ field.name|capfirst }}</label>
                                    <div class="input-group">
                                        <input type="email" class="form-control" id="id_{{ field.name }}"
                                               required
                                               autofocus
                                               name="{{ field.name }}"
                                        >
                                    </div>
                                </div>
                                {% endfor %}
                                <!-- End of Form -->
                                <div class="d-grid">
                                    <input type="submit" value="Recover password" class="btn btn-gray-800" />
                                </div>
                            </form>
                        </div>
                    </div>
                </div>
            </div>
        </section>
    </main>

{% endblock content %}

<!-- Specific Page JS goes HERE  -->
{% block javascripts %}{% endblock javascripts %}
```

This is an example of the `password_reset` endpoint. You can see more examples of custom template this [directory](https://github.com/app-generator/boilerplate-code-django-dashboard/tree/master/apps/templates/registration). 
Also, you can find the `urls.py` of the project [here](https://github.com/app-generator/boilerplate-code-django-dashboard/blob/master/apps/authentication/urls.py).
