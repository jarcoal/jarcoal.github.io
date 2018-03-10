---
layout: post
title:  "Say Goodbye to Virtualenv"
date:   2018-03-10 15:00:00
---

These days it's hard to imagine developing in Python without [virtualenv](https://virtualenv.pypa.io/).  Having isolated environments prevents a whole class of subtle (and not so subtle) bugs, and makes it much easier to manage dependencies.

But virtualenv isn't without it's pain points.  Having to `activate` and `deactivate` is a nuissance ([virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/) helps a bit) that makes jumping into, out of, and between environments harder than it needs to be.  Ideally we should be able to `pip install django` and `./manage.py runserver` without thinking about anything else.

The solution is [pipenv](https://pipenv.readthedocs.io/).  pipenv is a replacement for both pip and virtualenv, and is going to let us say goodbye to `activate`/`deactivate` forever.

I'm not going to cover it in detail for this post, but in addition to abstracting away virtual environments, pipenv also gives us a replacement for `requirements.txt` that ensures we get deterministic builds.  Learn more about what they look like [here](https://docs.pipenv.org/basics/#example-pipfile-pipfile-lock).

## Installing pipenv

For macOS users, the easiest way to get started is with [homebrew](https://brew.sh/):

```sh
$ brew install pipenv
```

But you can always use `pip`:

```sh
$ pip install pipenv
```

Now that we have `pipenv` installed, create a directory for your project:

```sh
$ mkdir -p ~/projects/pipenv_test/ && cd ~/projects/pipenv_test/
``` 

## Using pipenv

Here's where the magic starts.  Let's install a dependency:

```sh
$ pipenv install django
```

That's it.  Behind the scenes pipenv has created a virtual environment and installed django into it.  It has also recorded django as a dependency in `Pipfile` and `Pipfile.lock`.  Again, we won't go into those files in depth here, but like `requirements.txt` they should go into version control.

Continuing with our django example, let's create a project:

```sh
$ pipenv run django-admin.py startproject pipenvtest
```

So what exactly is happening here?  By prefixing our normal `django-admin.py` command with `pipenv run`, it will be run within the context of the virtual environment that we are no longer thinking about.

Now we can start the dev server:

```sh
$ cd pipenvtest
$ pipenv run ./manage.py runserver
```

Silky smooth.  What's a virtual environment?

## Environment Variables

If you're doing things by the book then you're probably storing your various secrets/tokens/etc in environment variables.  The problem is when you execute `pipenv run ...` it's going to lose those environment variables.  Fixing that is as easy as adding a `.env` file to the root of your project (aka next to `Pipfile`) and populating it with your vars:

```sh
$ echo 'DATABASE_PASSWORD=supersecretstuff' >> .env
```

Now when you `pipenv run ...` those environment variables will be in context.

That's it!  Enjoy never thinking about virtualenv again.