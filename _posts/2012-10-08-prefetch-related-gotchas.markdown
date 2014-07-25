---
layout: post
title:  "prefetch_related gotchas"
date:   2012-10-08 18:57:47
---

One of the cool new features rolled out in Django 1.4 is the `prefetch_related()` method added to querysets.  The purpose for this method is to improve performance for situations like this:

<script src="https://gist.github.com/3855957.js?file=bad-template-loop.html"></script>

See that inner loop for the product features?  That thing is banging against your database with each iteration.  If you have 100 products, that means 100 extra queries just to fetch their features.  No good.

Calling `prefetch_related('feature_set')` on the initial products queryset let's us get all of the features in one query to the database, then Django does some magic behind the scenes "joining" them to their parent product.

<script src="https://gist.github.com/3855957.js?file=prefetch-queryset.py"></script>

That's cool, now we're down to just two queries.  All is well?  Not quite.

To find the limitations with prefetch_related, we need to take a peek at the SQL that it's generating.  If you haven't figured this out already, you can see the SQL that Django's ORM is building by printing out the `query` attribute of any queryset like so:

<script src="https://gist.github.com/3855957.js?file=print-sql-example.py"></script>

That said, I highly recommend you look into [django-debug-toolbar](https://github.com/django-debug-toolbar/django-debug-toolbar).  It makes analyzing your queries/SQL a lot easier.

The first query Django makes to retrieve the products is the same as any other query you'd run.  It's just a quick select with any filters you might have expressed as `where` clauses:

<script src="https://gist.github.com/3855957.js?file=product-sql-example.sql"></script>

Looking good so far.  Now that Django has found your products, it's going to prefetch the features for those products.  For the sake of this example, let's say the previous query returned 5 products, with IDs 1-5.  Here is what your next query will look like:

<script src="https://gist.github.com/3855957.js?file=prefetch.sql"></script>

A bit different from the first query.  The `IN` portion of the where clause is the key thing to note here.  This tells your database: "find all of the product features associated with products 1-5".

Sounds good, and in this particular case it is likely a win for performance.  But can you imagine what this query will look like if your first query returned 500 products? 5000 products?  Not only will it become absolutely giant, requiring tons of data to be sent over the wire to your DB, but it's also not cache-able by your database.

Most databases perform simple query caching by looking at the incoming statement and checking to see if it has been parsed before.  If it has, the execution plan comes out of the cache and your DB saves a bunch of cycles that it would have spent parsing/explaining.  This is one reason why bind variables are such a clear win.  Not only do they protect your query from injection, they make it very easy on your database to identify previously run queries because rather than the values in your statement changing, they remain consistent thanks to statically named variables.

Besides the `IN` statement bloating up in size, it also changes constantly depending on the result set of the previous query (in this case the products query), so your database can't get a good cache of the feature statements you're sending it.

The real take-away here is that `prefetch_related()` works very well for queries that you expect to return a small amount of results, but as your result-set grows, you will see that performance benefit turn into a loss quickly.