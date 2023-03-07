---
title: "-1 as a url parameter in Django"
date: 2023-03-07T14:28:08-06:00
lastmod: 
draft: false
tags: ["django", "python"]
---

Django only accepts positive integers as url parameters.
Invalid `urls.py`:
```python
urlpatterns = [
path("<str:chart_name>/step/<int:step>", views.chart_step, name="chart_step"),
]
```
It doesn't really matter how we structure our view, since we receive an error nonetheless:
`views.py`:
```python
def chart_step(request, chart_name, step):

output = (

Chart.objects.filter(chart_name=chart_name)

.get()

.persona_set.filter(step=step)

)

return HttpResponse(output)
```

But say, we have a `step` parameter which receives values from -9999 to 0 to +9999?
We have to receive the parameter in `urls.py` in a different format, a different type to be exact. I chose to receive it as `str`:
valid `urls.py`:
```python
urlpatterns = [
path("<str:chart_name>/step/<str:step>", views.chart_step, name="chart_step"),
]
```
Since I defined the column in `models.py` to be an `IntegerField`, I can't pass this string-parameter directly to the view. I will have to typecast it into an `int`, by adapting my view function.
valid `views.py`:
```python
def chart_step(request, chart_name, step):

output = (

Chart.objects.filter(chart_name=chart_name)

.get()

.persona_set.filter(step=int(step))

)

return HttpResponse(output)
```

Now Django accepts the negative value and returns the corresponding database entries.