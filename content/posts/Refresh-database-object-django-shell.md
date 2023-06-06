---
title: "Refresh Database Object Django Shell"
date: 2023-06-05T23:14:11-06:00
lastmod: 
draft: true
tags: ["django", "shell", "database"]
categories: ["tech"]
---

Django provides a function through it's API to refresh a database object: `refresh_from_db()`. It is documented [here on the official Django docs](https://docs.djangoproject.com/en/4.2/ref/models/instances/#django.db.models.Model.refresh_from_db) and treated for example [here on StackOverflow](https://stackoverflow.com/questions/4377861/reload-django-object-from-database). Still I want to document it's usage.

# Usage
It is necessary to use `refresh_from_db()`, when a model is changed using an external program or - as happened in my case - when new fields are added to a model through `manage.py makemigrations` followed by `manage.py migrate`, while the same model is loaded in a Django shell (`manage.py shell`).  
It is necessary to call the function **upon an explicit object**, it is not possible to call it upon a whole model!

**models.py** at the time when the Django shell is started and the model imported:
```python
class ClientSetting(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    company_name = models.CharField(max_length=150, unique=True)
    company_name_slug = models.SlugField(
        max_length=150, default=slugify(company_name), editable=False, unique=True
    )
    address = models.CharField(max_length=250)
    servable_client = models.PositiveIntegerField()
```

**models.py** after adding a new field, followed by the necessary steps to migrate the database:
```python
class ClientSetting(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    company_name = models.CharField(max_length=150, unique=True)
    company_name_slug = models.SlugField(
        max_length=150, default=slugify(company_name), editable=False, unique=True
    )
    address = models.CharField(max_length=250)
    servable_client = models.PositiveIntegerField()
    calendar_id = models.CharField(max_length=90)
```

Since I never closed the shell nor refreshed the database, I get `AttributeError: 'ClientSetting' object has no attribute 'calendar_id'`, when I try to access an objects `calendar_id` field through an exemplary query like `ClientSetting.objects.get(pk=1).calendar_id`. Updating the whole model using `refresh_from_db()` fails, since no `self` is provided, but mandatory. To update the models using the same shell session, one has to call this command first:
```python
ClientSetting.objects.get(pk=1).refresh_from_db()
```

The real advantage obviously is, when the object was beforehand assigned to a variable as in:
```python
# assign object to variable o
o = ClientSetting.objects.get(pk=1)

# modify model as described above, migrate etc.
(...)

# refresh object
o.refresh_from_db()
```

Now, it is possible to access the updated models details in the same shell session (no closing, reopening, importing, setup required).
