---
title: "Notes Upgrade SQLAlchemy2.0"
date: 2023-09-13T11:10:50-06:00
lastmod: 
draft: false
tags: ["sqlalchemy", "notes", "db", "python"]
categories: ["tech"]
---

Wild collection of notes and changes while upgrading from SQLAlchemy1.4 to SQLAlchemy2.0
Before indicates the style for version 1.4, after indicates 2.0.

This post will get updated when I encounter more changes in my code.

**note on running pytest with warnings enabled: to run a pytest testsuite with all warnings enabled, instead of running `pytest`, run this: `SQLALCHEMY_WARN_20=1 python -W always -m pytest`[^1]

1. `declarative_base` import

Before:
```
from sqlalchemy.ext.declarative import declarative_base
```

After:
```
from sqlalchemy.orm import declarative_base
```

2. Querying a table by a primary key/id:

Before:
```
obj = db.query(MyTable).get(id)

```

After:
```
obj = db.get(MyTable, id)

```

Source: https://docs.sqlalchemy.org/en/20/orm/session_api.html#sqlalchemy.orm.Session.get

[^1]: https://stackoverflow.com/a/68082035/14009697