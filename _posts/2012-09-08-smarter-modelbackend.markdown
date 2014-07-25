---
layout: post
title:  "Smarter ModelBackend: select_related your user profile"
date:   2012-09-08 17:27:00
tags:
- django
- django-auth
---

I have a [user profile model](https://docs.djangoproject.com/en/dev/topics/auth/#storing-additional-information-about-users) for every project I build in Django, so when [django-debug-toolbar](https://github.com/django-debug-toolbar/django-debug-toolbar) told me that the auth app was sloppily selecting just my `User` object, then making another trip back to the database when I called `get_profile()`, I knew this would not stand.  Here's a quick little patch to select your user and profile data in one query.

Somewhere in your project (utils or misc file would be good), add this snippet:

<script src="https://gist.github.com/3681395.js?file=smartmodelbackend.py"></script>

Then in your settings:

<script src="https://gist.github.com/3681395.js?file=settings.py"></script>

After adding this code, make sure to logout of your app and back in (Django caches your backend after logging in, so it will keep going back to the original `ModelBackend` if you don't).  You should see one less query in your debug tools.

If you frequently access a model even deeper than your profile, you can easily change up the `'profile'` to something like `'profile__company'` and fetch them all in a single query.