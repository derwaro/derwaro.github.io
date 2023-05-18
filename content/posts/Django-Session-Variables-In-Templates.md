---
title: "Django Session Variables in Templates"
date: 2023-05-17T22:46:03-06:00
lastmod: 
draft: false
tags: ["django", "shortpost", "notes"]
categories: ["tech"]
---

To use a session variable in a Django template, you don't have to put it inside curly braces like this `{{ request.session.variable_name }}`.  
It is enough to simply write out `request.session.variable_name`.

Example:
You want to use a (custom) session variable inside of an `<a>` tag using `{% url %}` to create the correct, well, url. The endpoint however needs some parameters/variables, like for example, `company_name`.
```html
<a href="{% url 'session_writer' chosen_slot=slot.start|date:"Y-m-d\TH-i" endpoint="book_treatment" company_name=request.session.company_name %}"  id="{{ slot.start|date:"Y-m-d" }}T{{ slot.start|date:"H-i" }}">{{slot.start|date:"H:i"}} - available</a>
```
As you can see the variable inside of `{% url %}` does not need `{`curly`}` braces . Variables for other `html` tags, like `id` **do** need `{`curly`}` braces.