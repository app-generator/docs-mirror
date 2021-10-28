Flask does not provide any ready-made support for password reset, so we have to build our own password reset flow.

## Password reset flow (how it works)
 
- requesting password change by providing an email
- send an email containing a link to change the password
- click on the link and change the password

As mentioned above, since there is no ready-made support, so we have to take care of routing, form handling, html templates, etc by ourselves, to create the password reset flow.

## Creating forms

We will create two forms, one for taking email as input from user at start, and another for taking new password at end.

```python
class ForgotPasswordForm(FlaskForm):
    email = TextField('Email', id='email_create', validators=[DataRequired(), Email()])

class ResetPasswordForm(FlaskForm):
    email_token_key = HiddenField()
    email = HiddenField()
    password = PasswordField('New Password', id='pwd_create', validators=[DataRequired()])
    confirm_password = PasswordField(u'Confirm Password again', [EqualTo('password', message="Passwords don't match")])
```


## Creating route urls and password reset flow logic

In `authentication/routes.py` file we will create two urls as follows:

- Using `forgot-password` url we will accept email as input from user using `ForgotPasswordForm` form, and check if user with given email exists, and generate a unique `email_token_key`. This would be used to create a unique password reset url link, which would be emailed to the user.

- Using `reset-password` url, when a user clicks on above link received by email, `ResetPasswordForm` will be presented, where user can set new password. We will then check email against our database, and update password accordingly.


## UI templates (forms)

Flask gives simple `render_template` feature, which we can use to render required `.html` files such as `forgot_password.html` and `reset_password.html`

In respective route's .html file, relevant `form` can be rendered for example, `ForgotPasswordForm` is shown here:

```html
<form method="post" action="">
	{{ form.hidden_tag() }}
	<!-- Form -->
	<div class="mb-4">
		<label for="email">Your Email</label>
		<div class="input-group">
			{{ form.email(placeholder="Your Registered Email", class="input form-control", type="email") }}
		</div>
	</div>
	<!-- End of Form -->
	<div class="d-grid">
		<button type="submit" name="forgot-password" class="btn btn-gray-800">Recover password</button>
	</div>
</form>
```

This is an example of the password reset in flow flask. You can find more examples [here](https://github.com/app-generator/docs-mirror).