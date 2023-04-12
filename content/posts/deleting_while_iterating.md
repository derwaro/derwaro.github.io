---
title: "Deleting while iterating"
date: 2023-02-22T22:48:59-06:00
lastmod: 
draft: false
tags: ["python", "shortpost"]
categories: ["tech"]
---
Just found out that you can't `del` keys from a dict while iterating through it.

Say, we have the following dictionary:
```python
{
    "Alabama": {
        "USState": "Alabama",
        "Persons": 2,
    },
    "Missouri": {
        "USState": "Missouri",
        "Persons": 0,
    },
    "Idaho": {
        "USState": "Idaho",
        "Persons": 0,
    }
}

```

This doesn't work:

```python
for s in states.keys():
    if states[s]["Persons"] == 0:
        del states[s]
```

as it will through a `RuntimeError: dictionary changed size during iteration` at us!
(OK, it will actually delete the first found case and right afterwards through the error at us)

To delete while iterating over a dictionary, you have to iterate over a copy of the dictionary and delete from base dictionary:

```python
for s in states.copy().keys():
    if states[s]["Persons"] == 0:
        del states[s]
```
