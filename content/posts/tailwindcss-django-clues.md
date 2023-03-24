---
title: "TailwindCSS Django Clues"
date: 2023-03-23T22:44:09-06:00
lastmod: 
draft: true
tags: ["Django", "TailwindCSS"]
---

These are some clues to correctly set up the TailwindCSS NPM method with Django.
I'm closely following [this Stackoverflow Post](https://stackoverflow.com/a/63392427) by [David Dahan](https://stackoverflow.com/users/2255491/david-dahan), but for the clarity and to have some resource to solve this problem in my future I want to do a quick writeup with additions and points of divergence. But still **Thanks David!**

I'm using the following versions:
- Django 4.1.7
- tailwindCSS 3.2.7
- npm 9.6.2

So assume we have the following initial directory tree (not in full detail, I tried to include the necessary files and those that help locating everything):
```
.
└── django-project/
    ├── django-project/
    │   └── ...
    ├── my-app/
    │   ├── static/
    │   │   ├── css/
    │   │   │   └── custom.css
    │   │   └── js/
    │   │       └── custom.js
    │   ├── templates/
    │   │   └── my-app/
    │   │       ├── base.html
    │   │       └── my-view.html
    │   ├── urls.py
    │   ├── views.py
    │   ├── models.py
    │   └── apps.py
    ├── manage.py
    └── requirements.txt
```

- The first step is to run the following commands in our `django-project` folder (the first one that is):
1. `mkdir jstoolchain`
2. `cd jstoolchain`
3. `npm init -y`
4. `npm install -D tailwindcss`
5. `npx tailwindcss init`

- Now on to configure everything (this is where i adapted Davids solution). Being still in the folder `jstoolchain` use your favorite text editor to edit `tailwind.config.js` to:
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["../**/templates/**/*.{html,js}"],
  theme: {
    extend: {},
  },
  plugins: [],
}
```
*Notice:* the path for content in Davids solution is slightly different. His path didn't work with the standard django directory layout!

- Create an `input.css` file in the upper `django-project` folder. The file should contain at least the following tailwindcss components:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Our directories now look like this (notice the added `input.css` file!):
```
.
└── django-project/
    ├── django-project/
    │   └── ...
    ├── my-app/
    │   ├── static/
    │   │   ├── css/
    │   │   │   └── custom.css
    │   │   └── js/
    │   │       └── custom.js
    │   ├── templates/
    │   │   └── my-app/
    │   │       ├── base.html
    │   │       └── my-view.html
    │   ├── urls.py
    │   ├── views.py
    │   ├── models.py
    │   └── apps.py
    ├── manage.py
    ├── requirements.txt
    └── input.css
```

- create shortcuts/commands to run tailwind in `package.json` located in the `jstoolchain` folder:
```json
{
  "name": "jstoolchain",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "tailwind-watch" : "tailwindcss -i ../input.css -o ../my-app/static/css/output.css --watch",
    "tailwind-build" : "tailwindcss -i ../input.css -o ../my-app/static/css/output.css --minify"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "tailwindcss": "^3.2.7"
  }
}
```

*Notice:* the `-o flag` of the commands in `scripts` have to contain the path to the actual app. In our case: `my-app`. Substitute as necessary. Or alternatively, but not yet tested: pass `**` instead of the app name. E.g.: `"tailwind-watch" : "tailwindcss -i ../input.css -o ../**/static/css/output.css --watch"`

- in `jstoolchains` keep running `npm run tailwind-watch` to ensure the `output.css` is created.

- if `npm run tailwind-watch` runs without errors, out `output.css` should be populated by the tailwindCSS classes we use in our files.

- the last step is to include the generated `output.css` in our templates. I included it in my `base.html` header section like this:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>{% block title %}Default Title{% endblock %}</title>
    {% load static %}
 
<link href="{% static 'css/output.css' %}" type="text/css" rel="stylesheet" /> 
</head>
<body>

(...)

```

**Conclusion:** I slightly modified the path parameters in `tailwind.config.js` and `package.json` and the link in the base template's header to make this solution work for the standard directory layout as created by django itself. Again: the biggest thanks go to [David Dahan](https://stackoverflow.com/users/2255491/david-dahan)!