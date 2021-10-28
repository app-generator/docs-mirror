This page explains how to associate a `Profiles` model with additional information like address, phone number .. etc., with a `Users` model in Flask.

In Flask, we usually keep a `Users` model with basic details like name, email, etc, however to add other related fields it is recommended to keep another `Profiles` model.

## Steps Involved

- creating `Profiles` model
- creating `ProfileForm` form
- creating routes like `settings` for editing/adding profile settings

## `Profiles` model

```python
class Profiles(db.Model):

    __tablename__ = 'profiles'

    id = db.Column(db.Integer, primary_key=True)

    firstname = db.Column(db.String(64))
    lastname  = db.Column(db.String(64))
    birthday  = db.Column(db.DateTime)
    gender    = db.Column(db.Integer)
    phone     = db.Column(db.String(32))
    address   = db.Column(db.String(128))
    number    = db.Column(db.String(64))
    city      = db.Column(db.String(64))
    state     = db.Column(db.String(64))
    country   = db.Column(db.String(64))
    zipcode   = db.Column(db.String(16))
    photo     = db.Column(db.String(512))

    users_id = db.Column(db.Integer, db.ForeignKey("Users.id"))
    user = db.relationship("Users", uselist=False, backref="profiles")

    def __repr__(self):
        return str(self.id)
```

`users_id` field is to create a foreign key associated with `id` column of `Users` model


## `ProfileForm` model

```python
class ProfileForm(FlaskForm):
    firstname  = TextField('First Name', id='first_name', validators=[DataRequired()])
    lastname   = TextField('Last Name', id='last_name', validators=[DataRequired()])
    birthday   = TextField('Birthday', id='birthday', validators=[DataRequired()])
    gender     = SelectField('Gender', id='gender', choices=[("1", "Male"), ("2", "Female")])
    email      = TextField('Email', id='email', validators=[DataRequired(), Email()])
    phone      = TextField('Phone', id='phone', validators=[DataRequired()])
    address    = TextField('Address', id='address', validators=[DataRequired()])
    number     = TextField('Number', id='number', validators=[DataRequired()])
    city       = TextField('City', id='city', validators=[DataRequired()])
    state      = TextField('State', id='state', validators=[DataRequired()])
    country    = TextField('Country', id='country', validators=[DataRequired()])
    zipcode    = TextField('Zip Code', id='zipcode', validators=[DataRequired()])
    photo      = FileField('Profile Photo', id='photo', validators=[DataRequired()])
```

## `settings` route and utils

This is used to handle profile data. Few useful things to take a note are:

- at start, it is checked if profile is already created/exists, or a new object in `Profile` model is required (if user is accessing this route and saving settings for the first time)
- form has date as `str` type, which is converted into a python `datetime` object
- a utility method `allowed_image_file` is used to check user profile photo extension type, so that only valid ones like .png or .jpg files are allowed to upload
- `settings.html` template is rendered in this route with `ProfileForm` details


This is an example of the user profile in flask. You can find more examples [here](https://github.com/app-generator/docs-mirror).