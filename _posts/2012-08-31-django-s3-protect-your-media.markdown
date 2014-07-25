---
layout: post
title:  "Django & S3 - Protect Your Media"
date:   2012-08-31 14:32:00
tags:
- django
- s3
- aws
---

So you found the wonderful [django-storages](http://django-storages.readthedocs.org/) and now you're using S3 to store your static and media files.  Perfect.  All those 9's should keep your content safe from datacenter drama (hopefully).

But what about fat fingers or malicious users?  One bad `DELETE` request issued to S3 is all it takes to wipe out your data.  Your static files can easily be pulled back out of version control, but your media files are gone.

Enter [S3 versioning](http://docs.amazonwebservices.com/AmazonS3/latest/dev/Versioning.html).  Versioning is a great feature of S3 that gets very little press.  The idea is simple: when enabled, S3 will archive and version any file that gets uploaded to your bucket and the newest version of a file gets served when a request comes in.  In addition, if you issue a `DELETE` request against that same file, it will insert a marker at the top of the version stack indicating the file is deleted, so it knows that the next time it's requested, a 404 should be served.  The end result is that your files can never be deleted with a simple `DELETE` request.

So how do you enable versioning on a bucket?  Simple: issue an authorized `PUT` request against the bucket like so (note  `?versioning` query string):

<script src="https://gist.github.com/3558666.js?file=s3-versioning.txt"></script>

Now we could probably just leave it at that.  After all, your files are now safe from an accidental or rogue `DELETE`.  But your static files that are already in version control and can easily be restored are now being re-versioned by S3 with every `collectstatic` you issue.  This is obviously redundant and costly, so let's fix that.

If you're anything like me, somewhere in your `settings.py` you have something like this:

<script src="https://gist.github.com/3558835.js?file=django-static-file-config-s3.py"></script>

This is great, but unfortunately it binds both your media and static to the same bucket.  This isn't going to work if we want to keep our static files off of the versioned bucket we created for our media.  Let's separate that out into two buckets:  one versioned bucket for our media, and one regular bucket for our static.  Somewhere in your project, create a `s3config.py` that looks like this:

<script src="https://gist.github.com/3558907.js?file=django-multi-bucket-config.py"></script>

Now back in your `settings.py`, let's separate out those storage backend configs:

<script src="https://gist.github.com/3558971.js?file=django-multi-bucket-settings.py"></script>

We're looking good now.  Our media files will get uploaded to the versioned bucket, and our replaceable static files are uploaded to a regular bucket.

That just about wraps things up.  There are a couple of issues I didn't tackle in this post, such as how to actually recover from an old version if you do accidentally delete your files.  There are [good docs](http://docs.amazonwebservices.com/AmazonS3/latest/dev/RestoringPreviousVersions.html) on AWS that explain this process.

Also, what if a malicious user issues the `DELETE` request?  Won't they just delete all of the versions as well?  If you're using AWS's [multi-factor authentication](http://aws.amazon.com/mfa/) (and you should be), then you can [setup your versioned bucket to require a MFA code](http://docs.amazonwebservices.com/AmazonS3/latest/dev/ConfiguringaBucketwithMFADelete.html) to delete or suspend versions. 