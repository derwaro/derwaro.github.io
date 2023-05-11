---
title: "Django Toplevel Base Template"
date: 2023-05-11T10:54:34-06:00
draft: false
tags: ["django", "shortpost", "notes"]
categories: ["tech"]
---
Django organizes the parts of a project inside of more or less isolated apps. This makes it difficult to have a `base.html`, that can be easily applied to all `.html` pages from a centralized location. Luckily there is a way to do so:

1. open project_name/project_name/settings.py and find the TEMPLATES array and update the 'DIRS' : [] to 'DIRS': [os.path.join(BASE_DIR, 'templates')]
2. Create a directory at root level named templates (that is the root level for that project. Same location as `manage.py`). The location of this folder will be project_name/templates
3. Place all templates here in projects_name/templates that you want to access from all the apps. File: project_name/templates/base.html
4. Extend this base file from any application-specific template, that might be located at project_name/app_name/templates/app_name/template_file.html by using {% extends "base.html" %}.

based on Stack Overflow [answer](https://stackoverflow.com/a/60931532/14009697) by [Mukesh Kumar](https://stackoverflow.com/users/5022624/mukesh-kumar).
