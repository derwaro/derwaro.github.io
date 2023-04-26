---
title: "Django Form Custom Validation"
date: 2023-04-25T12:22:15-06:00
lastmod: 
draft: false
tags: ["django", "flask"]
categories: ["tech"]
---
I could not find the the correct solution or path to custom form validation in Django, so I will quickly write up my findings (for me to check later on and maybe they help you too).
Keep on reading↡ if you want to follow my path of discovery, jump to the [working answer](#workinganswer) or to the [working snippet](#workinsnippet) right away.

# flask-wtforms-style
Initially i wanted to impose the validation flask-wtforms-style, since this is how i learned form validation first. Turns out i wasn't quite able to do that. The idea was to put all my validators in a `validators.py` file in my app. Import the used (would say all) validators into my `forms.py` and impose them onto form fields where needed. **This did not work for me!**

**validators.py**  
```python
from django.core.exceptions import ValidationError


def end_smaller_than_start(time_end, time_start):
    if time_start >= time_end:
        raise ValidationError(
            f"End time {time_end} should be after start time {time_start}"
        )
```

**forms.py**  
```python
from django import forms

from django.utils import timezone

from .validators import end_smaller_than_start


class FillPapelitoForm(forms.Form):
    equipment_type = forms.CharField(max_length=150, required=True)
    time_start = forms.TimeField(required=True)
    time_end = forms.TimeField(required=True, validators=[end_smaller_than_start])

```

**models.py**  
I did not put any specific constraints into my `models.py`, but for the sake of completeness:  
```python
from django.db import models
from django.utils import timezone

# Create your models here.
class PapelitoReport(models.Model):
    equipment_type = models.CharField(max_length=150)
    time_start = models.TimeField()
    time_end = models.TimeField()

```

*Note:* I left out some models, fields that are not important in proving my point here.

While this *did* in some way validate I couldn't get the code to do what I wanted in the validator. What I wanted was to make sure that `time_end` was bigger, i.e. after, `time_start`, since I want the duration of the usage for the equipment. So my logical approach would be to check in the validator:
```
if time_end smaller than time_start:
    throw an error
```

However for some reason I couldn't make out the validator validated the other way round. That is, I had `<=` (smaller or equal than) in my validator but it only validated in my expected way if I put `>=` (bigger or equal than) in the validator. 
some quick examples:
```
*first example*
time_start: 08:00
time_end: 16:00
--> error!

*second example*
time_start: 08:00
time_end: 07:00
--> **no** error
```

Since I wanted my code to clearly represent what it does and what I want from it, I had to come up with some other way to custom validate those two values. I tried and tried and searched and searched and finally came up with the:

# working answer {#workinganswer}
Based on [this answer](https://stackoverflow.com/a/44749336) on Stack Overflow by User [Igor](https://stackoverflow.com/users/6174024/igor-yudin) I was able to validate my values in the way I wanted. The answer states that one should `not describe fields in forms.py`, but nonetheless `some special ones` can be described in `forms.py`. The answer states the following solution in  
**forms.py**  
```python
from models import Student

class StudentForm(forms.ModelForm):
    (...)
    (fields are defined here)
    (I left them out since the answer uses a different approach to define fields)
    (...)
    
    # Now you could describe all validation methods
    def clean_first_name(self):
        first_name = self.cleaned_data['first_name']
        if not first_name.isalpha():
            return ValidationError('First name must contain only letters')
        return first_name
```

What this does is it it checks the `cleaned_data`[^1] returned on form submission, from the dictionary it grabs the values needed, validates them and *returns* a descriptive `ValidationError`. If no error is found (the question was asking for how to validate that the field `first_name` contains only alphanumerical values), then the same, unchanged value is returned.

This did work more or less. There was just a small error that prevented this from working for me. And I think also for the original question poster (there is a comment below Igor's solution stating: `It still isn't validating(...)`). In line 13 of the code part above Igor `return`s a `ValidationError`. This did not work for me. I changed the return to `raise` and ... it worked fine!

So now on to the  
# working snippet {#workingsnippet}
for my case based on the answer above:

**forms.py**
```python {hl_lines=[14]}
from django import forms

from django.core import validators

class FillPapelitoForm(forms.Form):
    equipment_type = forms.CharField(max_length=150, required=True)
    time_start = forms.TimeField(required=True)
    time_end = forms.TimeField(required=True)

    def clean_time_end(self):
        time_end = self.cleaned_data["time_end"]
        time_start = self.cleaned_data["time_start"]
        if time_end <= time_start:
            raise forms.ValidationError(
                f"Error: End Time {time_end} must be after Start time {time_start}"
            )
        return time_end
```

Here the validator does what is in the code and what I want it to check (that if end is smaller than start, there should be an error) and *`raise`s* the error (instead of *`return`ing* it).

*note:* I submitted an edit to the Stack Overflow answer, which is at submission of this post still in review. 




[^1]: cleaned_data is a dictionary-like object that stores the cleaned form data after validation. In Django, when a user submits a form, the data is automatically validated by the framework. During this process, Django checks if the data entered in the form is valid and in the correct format. If it's valid, Django stores the cleaned data in the cleaned_data dictionary.

The cleaned_data is passed to the view function as an argument and can be used to create or update models, generate reports or perform any other operations required. The cleaned_data dictionary is also used to re-populate the form fields in case there is any validation error, so the user doesn't have to re-enter the data again.
