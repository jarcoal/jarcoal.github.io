---
layout: post
title:  "Versioning Static Assets w/ Heroku Config Vars"
date:   2012-10-17 15:41:00
---

Cache-related HTTP headers are a great tool in our web dev toolbox, but some browsers still find ways to ignore them, requiring us to fallback to our old methods of static versioning.

Here is a quick and easy way of managing your static versions using Heroku's config vars.  This example uses Django, but the idea can be applied to any backend.

**1) Create a template context processor like this:**

<script src="https://gist.github.com/3908746.js?file=template_context_processor.py"></script>

**2) Add it to your [TEMPLATE\_CONTEXT\_PROCESSORS](https://docs.djangoproject.com/en/dev/ref/settings/#std:setting-TEMPLATE_CONTEXT_PROCESSORS)**

<script src="https://gist.github.com/3908746.js?file=settings.py"></script>

**3) Apply the template variables to your link/script tags:**

<script src="https://gist.github.com/3908746.js?file=template.html"></script>

**4) After pushing out your code, setup your config vars:**

<script src="https://gist.github.com/3908746.js?file=heroku-config.txt"></script>

So from now on, when you make some changes to your static content and push it out to Heroku, you can easily version them by incrementing the value of your static version config vars.  