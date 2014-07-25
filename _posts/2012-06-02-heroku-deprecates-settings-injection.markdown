---
layout: post
title:  "Heroku Deprecates Settings Injection & New collectstatic Hook"
date:   2012-06-02 16:40:00
tags:
- heroku
- django
---

[Heroku has deprecated their settings injection policy](https://devcenter.heroku.com/articles/django-injection) for Django apps.  If you aren't familiar, Heroku currently appends a `DATABASES` variable to your setting.py file when you deploy your app, which contains the information needed to connect to your database.

Obviously this can be confusing at best and dangerous at worst, so I'm glad to see they're removing it.  Apps that want to turn it off right now rather than waiting until it's removed (which is highly advisable :), can follow the instructions [here](https://devcenter.heroku.com/articles/django-injection).

Heroku has also added a hook that allows you to call a `collectstatic` on your app right after it is deployed, which should be pretty handy as I find myself doing that after every push anyway.  This feature is on by default, but you can [disable it](https://devcenter.heroku.com/articles/django-assets) if you'd like.  Now if they could also add a hook that runs your south migrations, I would be set.