---
title: Code snippets in blog posts
published: false
---

Trying to decide which code blocks would be best for posting blogs.

----

### Jekyll/Liquid highlight block, for sql

**Source:**

{% raw %}
```liquid
{% highlight sql %}
SELECT *, OBJECT_NAME(o.[object_id])
FROM sys.objects o
WHERE o.[name] = 'foobar'
	AND o.[type] = 'P'
ORDER BY o.[object_id];
{% endhighlight %}
```
{% endraw %}
