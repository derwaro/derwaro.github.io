--- 
title: "Don't forget the errors"
date: 2022-06-02T23:19:25+02:00 
lastmod: 
tags : ["python", "flask", "wtforms", "flask-bootstrap", "bootstrap"] 
draft: true
categories: ["tech"]
---

## Looping over radio-buttons
For a project I had the need to manually loop over a radiobutton group. 
This problem arose, since I was unable to assign a class to the automagically created buttons when using the `render_field` macro from flask-bootstrap:
```html
<div class="col-md-4">
    {{ form.geschlecht.label }}
    {{ wtf.form_field(form.geschlecht, class='form-control') }}
</div>
```
This is the snippet from another field, where I was able to add the bootstrap necessary class `form-control`. However for the actual radiobutton group in question `art`, I had to have two classes `form-control` and `rd_art`. I needed it to be able to make js talk to it.

After some searching I found out that I can manually loop over the radiobutton group and while doing so add classes to each element.
According to the [wtforms documentation](https://wtforms.readthedocs.io/en/2.3.x/_modules/wtforms/fields/core/#RadioField):

>class wtforms.fields.RadioField(default field arguments, choices=[], coerce=unicode)[source]
>
>    Like a SelectField, except displays a list of radio buttons.
>
>    Iterating the field will produce subfields (each containing a label as well) in order to allow custom rendering of the individual radio fields.
>```html
>    {% for subfield in form.radio %}
>        <tr>
>            <td>{{ subfield }}</td>
>            <td>{{ subfield.label }}</td>
>        </tr>
>    {% endfor %}
>```
>    Simply outputting the field without iterating its subfields will result in a <ul> list of radio choices.

*Perfect!* Let's get going:
```html
<div class="col-md-4">
    {{ form.art.label }}
    <br>
    {% for subfield in form.art %}
    <tr>
    <td>{{ subfield(class="rd_art") }} {{ subfield.label }}</td>
    </tr><br>
    {% endfor %}
</div>
```

It's really that simple!

BUT! What is not mentioned in the documentation (and obviously not thought of by me), are the errors! Using the template from the documentation, errors are not rendered anywhere. Pair that with a missing `required` argument AND with a missing `default` in the `forms.py`, and your're bound for ~~error~~ trouble.
So when submitting the form, the user is presented the same form with all the details that where just filled in, but no error message. One may think, that it was submitted just fine, but nope, it was not.

### remembering the errors



