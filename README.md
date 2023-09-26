# Django
Get count of views by views, meaning group by using views:
<pre>
  #group by only views:
  queryset = Blog.objects.values('views').annotate(view_count=Count('views'))

  #group by name and views, whatever we put in values() that becomes group by
  queryset = Blog.objects.values('views', 'name').annotate(view_count=Count('views'))
</pre>

# Decorators, kafka, ORM, class & methods
