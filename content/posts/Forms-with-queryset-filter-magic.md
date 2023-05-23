---
title: "Forms With Queryset Filter Magic"
date: 2023-05-20T15:57:34-06:00
lastmod: 
draft: false
tags: ["django", "form", "howto", ""]
categories: ["tech"]
---
This is a post about some of the *magic* that Django provides out of the box. What I wanted to accomplish is to filter the option of a `forms.ModelChoiceField` based on a ForeignKey of a ForeignKey.


# Introduction
My Project has two apps: `Appointment` and `Accounts`.
`Appointment` provides views to book an appointment for certain `Treatment`.
My clients (intended for small business owners/service providers) can create accounts on my platform, that are stored in the Django-provided `django.contrib.auth.models User` models. In addition to the basic account details I extend certain logics and User details in the `Accounts` app. Most importantly for this post, each user is assigned a `ClientSettings` entry, where the user can provide a name for their business, i.e. `company_name`. This value also provides unique url on my platform. It should be obvious that if you wanted to book an appointment at `testcompany`, you should only see those `Treatment` options that are affiliated with that company. So calling url `example.com/testcompany/choose_treatment`, should only show those `Treatment`s that where created by the user who has `testcompany` set in their `ClientSettings` entry.

To accomplish this it was clear to me that I had to create a `queryset` for my form. Initially I wanted to pass the filtering right in the `forms.ModelChoiceField` definition:  
# **not working forms.py**  
```python
from django import forms
from .models import Treatment
from phonenumber_field.formfields import PhoneNumberField


class ChooseTreatmentsForm(forms.Form):
    name = forms.ModelChoiceField(
        queryset=Treatment.objects.filter(active=True).filter(company_name=Treatment.user.clientsettings.company_name).all()
    )
    client_name = forms.CharField(
        max_length=150,
    )
    client_surname = forms.CharField(
        max_length=150,
    )
    client_mail = forms.EmailField(
        required=True,
    )
    client_phone = PhoneNumberField()
```

This did however not work. It didn't matter if I imported the `User` nor the `ClientSettings` model. I was able to filter it with the same query in the Django Shell, but not in my actual `forms.py`. Strange.

Then I found out various things:
- I should put the `queryset` filtering into the form's `__init__` function: This makes sure that the options are not *cached* and generated at each call of the form.
- I have to actually pass the value of `company_name` to the form: To be honest, this was obvious, but I worked on other ways to do so, like retrieving it from the session value, where I stored it in the view, which did not work. 
- Something was off with the dotted values. I was missing some *magic*, since it worked in the shell, but not in `forms.py`.

# My rewritten **forms.py**
```python {hl_lines=[2,18]}
class ChooseTreatmentsForm(forms.Form):
    name = forms.ModelChoiceField(queryset=Treatment.objects.none())
    client_name = forms.CharField(
        max_length=150,
    )
    client_surname = forms.CharField(
        max_length=150,
    )
    client_mail = forms.EmailField(
        required=True,
    )
    client_phone = PhoneNumberField()

    def __init__(self, *args, **kwargs):
        company_name = kwargs.pop("company_name")
        super().__init__(*args, **kwargs)
        self.fields["name"].queryset = Treatment.objects.filter(
            user__clientsettings__company_name=company_name, active=True
        )
```
First off, I had to define the `name` field (i.e. the name of the `Treatment`) with an empty `queryset`. This makes sure that the field is generated correctly and showing no options, when there are no `Treatment`s affiliated with the `company_name`.  
To generate the choices for the field, it is necessary to plug into the forms `__init__` function. The function receives the parameter of `company_name` from the instantiation in the view (see further down) and stores it in a variable with the same name, through the `pop` method (reminder: `pop` removes a key form a dict and returns it's value). Next the function calls upon the regular form instantiation through `super()` and then sets the queryset for field `name` with my desired filter on line 18.  
Django does not, in this case anyways, use dotted properties. It uses double underscores: *magic*. To better understand this, I should show you my models:
**appointment/models.py**
```python {hl_lines=[18]}
from django.db import models

from datetime import timedelta
from django.contrib.auth.models import User


class Treatment(models.Model):
    name = models.CharField(
        max_length=200, unique=True, blank=False, default="New Treatment"
    )
    price = models.IntegerField(default=0, blank=False)
    duration = models.DurationField(default="00:15:00", blank=False)
    description = models.CharField(
        max_length=500, default="This is the description of the treatment."
    )
    client_count = models.IntegerField(default=1)
    active = models.BooleanField(default=True)
    user = models.ForeignKey(User, on_delete=models.CASCADE)

    def __str__(self) -> str:
        return f"{self.name}, {self.duration} minutes, Price: {self.price} MXN"
```
links each `Treatment` to a `User`.

**accounts/models.py**
```python {hl_lines=[7]}
from django.db import models

from django.contrib.auth.models import User


class ClientSettings(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    company_name = models.CharField(max_length=150)
    address = models.CharField(max_length=250)
    servable_client = models.PositiveIntegerField()

```
links each `User` to a `ClientSettings`.

So to reach the `ClientSettings` for each `User` for each `Treatment` I have to filter `user__clientsettings__company_name`, using double underscores to successfully access the linked models. 

Now to the last still open point: How to pass the parameter to the form upon instantiation? Here's are the relevant parts of my
**views.py**
```python {hl_lines=[5,6,9,14]}
(...)
def choose_treatments(request, company_name):
    # get company name from url and save to session
    # e.g. https://example.com/COMPANY/choose_treatments
    company_name = request.build_absolute_uri().split("/")[-3]
    request.session["company_name"] = company_name

    if request.method == "POST":
        form = ChooseTreatmentsForm(request.POST, company_name=company_name)
        if form.is_valid():
            (...)
            return redirect("calendarview", company_name=company_name)
    else:
        form = ChooseTreatmentsForm(company_name=company_name)
    return render(request, "appointment/choose_treatments.html", {"form": form})
(...)
```

Since this is the first view a user will call on each company's (my clients) page, the `company_name` is extracted[^bignote] from the url and stored in session on lines 5 and 6. Line 9 and 14 show how a form is instantiated with a parameter, which is then received in `forms.py` as shown above.

This is how I successfully passed a dynamic parameter to a form and filtered the form's output based, namely the `ModelChoiceField`, on it.

[^bignote]: the relevant part of the projects `urls.py`:  
    **project/urls.py**
    ```python
    (...)
    urlpatterns = [
        (...)
        path("<company_name>/", include("appointment.urls")),
        (...)
    ]
    ```