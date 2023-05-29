---
title: "Group Permissions After Renaming Model"
date: 2023-05-25T11:03:20-06:00
lastmod: 
draft: true
tags: ["django", "database", "migrations"]
categories: ["tech"]
---
# Introduction
Django offers a really easy way to rename classes in it's `models.py` file, since it automatically recognizes that the class name changed and documents the necessary changes in it's migrations file and then applies them.

# Rename a class in models.py
The steps to rename a class in your `models.py` therefore are:
1. rename all imports ect. in other files to the new class name (Tip: in vscode use `ctrl+shift+f` to search in all files in your workspace)
1. rename the class in your `models.py`
2. run `manage.py makemigrations`
3. answer "y" when asked if you renamed `OldClass` to `NewClass`  
    3.1. verify that the migrations file was created correctly
4. run `manage.py migrate`

That's it.

One would think that using this workflow Django is now completely up to date and on par with the new model name. Well, yes and no. The database and all imports/occurrences of `OldClass` **are** effectively updated. **But**, Django does not update `auth_permissions` nor `auth_group_permissions` tables. I stumbled upon this, when I changed a class in my `models.py`, renamed all occurrences of OldClass, but suddenly my users could see their NewClass in their admin interface, although it was correctly registered in `admin.py`.

# Example case
The layout of my project is more or less like this:
Each `User` has a `ClientSettings` (take notice of the "s", plural of setting) table assigned for further details provided by each user. Each `User` upon registration is assigned my custom group `client`[^1] and is a member of staff (`is_staff=True`). I noticed that Django's admin interface adds a plural "s" to each registered model. So my class `ClientSettings` was represented as `Client Settingss` there. I couldn't live with that and changed the class name to not include the plural "s": `ClientSetting`. I followed all the steps I outlined above and called it a day. I went ahead and logged myself into the admin interface with a test account to see if Django really managed to rewire the pre-existing rows to the renamed rows. As far as I could tell I had my `accounts/admin.py` rewritten just fine to the new model name:

```python
from django.contrib import admin
from .models import ClientSetting


class ClientSettingAdmin(admin.ModelAdmin):
    def get_queryset(self, request):
        qs = super(ClientSettingAdmin, self).get_queryset(request)
        if request.user.is_superuser:
            return qs
        return qs.filter(user=request.user)

    def save_model(self, request, obj, form, change):
        if not obj.user:
            obj.user = request.user
        obj.save()


admin.site.register(ClientSetting, ClientSettingAdmin)
```

## Problem
But still I could not see a section for the test user's `ClientSetting`. I poked around, made double sure that I substituted the old model name with the new one, checked through the Django shell, manually created users and assigned them a `ClientSetting` row,... to no avail, I just could not get the section for `ClientSetting` to show up in the admin interface. Then I tried to access the url[^2] directly, which returned `403 Access Forbidden`! Aha, I was up to something. Seems like the section *does actually exist*, but the logged in user does not have the right permissions to access it[^3]. Then it hit me: After I intitially created my custom group `client`, I set up it's permissions. Maybe Django after all, did not update *every* occurrence of the old model name?

## Solution: add permission to group (again)
T=Django does, as a matter of fact, not update model names in it's automagically generated and maintained tables `auth_permissions` and `auth_group_permissions`. Still it does add new entries to it for the new model name! As is shown in the picture below, where I already added the new permissions to the `Chosen permissions` column:
![Django Group Permissions Selection](/img/django-group-permissions.png#center)
Interestingly Django added the old name and the new name with in the exact same way in this view. Both permissions are filed under `accounts | client setting | ...`, with no plural "s" no matter what. This makes it a bit difficult to distinguish. For my case, I simply added all occurrences. You can however for completeness sake, delete the older permissions.

## Find and delete old permissions
If you want the admin interface to only show those permissions you really are applying and that are in existence, hop into a Django shell (`manage.py shell`) and run the following commands, after making a backup of your database:
```python
# import necessary Django module
from django.contrib.auth.models import Permission

# query and filter the Permission table
# substitute 'client setting' as needed
p = Permission.objects.filter(name__contains='client setting').all()

# sort the result by it's id's
p = sorted(p, key=lambda Permission: Permission.id)

# by now our list should contain 8 items. 4 old ones and 4 new ones.
len(p)
# output: 8

# delete the first four entries. They have a lower id and therefore reference the old model name
for i in p[:4]:
    i.delete()

# Query Permission again. Expect to have 4 entries returned
len(Permission.objects.filter(name__contains="client setting").all())
```

This is also reflected in the `Chose permissions` column from earlier:

![Updated Chosen Permissions Selection](/img/django-group-permissions2.png#center/)

The problem is present for some years now (see [Stack Overflow question from 2009](https://stackoverflow.com/questions/593810/fixing-the-auth-permission-table-after-renaming-a-model-in-django)) , but it seems that there still is no automated solution to it. 

# Summary
Django really does make it easy to change your mind on model names and captures the intentions really well in it's migrations. If, however, the model is used as a permission setting, there is some extra work involved!

[^1]: this is done with the following lines in **accounts/views.py**:
    ```python
    (...)
    client_group = Group.objects.get(name="client")
    newuser.groups.add(client_group)
    (...)
    ```

[^2]: `localhost:8000/admin/accounts/clientsetting`

[^3]: This was made even more obvious when I tried to access `localhost:8000/admin/accounts', which listed the path for `localhost:8000/admin/accounts/clientsetting` as a valid one. 