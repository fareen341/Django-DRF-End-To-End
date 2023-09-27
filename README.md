# Django ORM
1. Group By: Get count of views by views, meaning group by using views:
<pre>
  #group by only views:
  queryset = Blog.objects.values('views').annotate(view_count=Count('views'))

  #group by name and views, whatever we put in values() that becomes group by
  queryset = Blog.objects.values('views', 'name').annotate(view_count=Count('views'))
</pre>

2. Join: all(), select_related(), prefetch_related()
<pre>
class Author(models.Model):
    name = models.CharField(max_length=100)
    bio = models.TextField()

    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    # publication_date = models.DateField()
    version = models.CharField(max_length=13)

    def __str__(self):
        return self.title

class Price(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE, default=0.0)
    price = models.IntegerField()
</pre>

<pre>
So the difference in prefetch_related is select_related is:
1. prefetch_related is used when you have to get data from reverse foreign key eg: author and book, book has foreign key and 
  i want to fetch records from author table then i can use prefetch_related. Eg: Book.objects.prefetch_related('price_set')
2. select_related is used when we wants to fetch records from the foreign key table i.e from book table. Eg: Book.objects.select_related('author')
3. consider below query:
    x = Book.objects.all().values('title', 'author__name')
    x = Book.objects.select_related('author').values('title', 'author__name')

both will print the same query:
SELECT "BlogApp_book"."title", "BlogApp_author"."name" FROM "BlogApp_book" INNER JOIN "BlogApp_author" ON ("BlogApp_book"."author_id" = "BlogApp_author"."id")

But the main difference is, when we hit first query it'll run the query again and again even tho it is join it'll do the join again and again, 
  but in second query it'll do the join once and store the result in the memory

4. use of both select_related & prefetch_related
Eg: for this eg consider the below models.
Book.objects.select_related('author').prefetch_related('price_set')
1. In above example, i can go from price to book but not vice versa, if i want vise versa then i can use prefetch_related. So in above example 
  author is part of Book, but i cant see price field name in book itself thats is why here comes the reverse foreign key.
2. in prefetch_related i can use _set or i can give the name in model field itself, which is related_name = "price
</pre>

# Django-Celery:

Redis stuff:
1. Pull redis and run:
    $ docker pull redis
    $ docker run --name redis -d redis

2. Check on which ip it is running:
    $ docker inspect container_id | grep IPAddress

3. create celery.py file: this file should be inside the app, where our views.py file is
<pre>
# celery.py

from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# Set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'Blog.settings')

app = Celery('Blog')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()
</pre>

4. Edit settings.py file.
<pre>
CELERY_BROKER_URL = 'redis://172.17.0.2:6379'
CELERY_RESULT_BACKEND = 'redis://172.17.0.2:6379'
CELERY_ACCEPT_CONTENT = ['json']
CELERY_TASK_SERIALIZER = 'json'
CELERY_RESULT_SERIALIZER = 'json'
</pre>

5. Pip install necessary stuff.
<pre>
pip install celery
pip install redis
</pre>

6. Create tasks.py file: this file should be in the app, where views.py file is
<pre>
# BlogApp/tasks.py

from celery import shared_task
import time

@shared_task
def send_email_with_delay(email, message):
    # Simulate sending an email
    time.sleep(5)  # Simulate a 5-second delay
    return f"Email sent to {email}: {message}"
</pre>

7. views.py
<pre>
# BlogApp/views.py

from django.shortcuts import render
from django.http import HttpResponse
from BlogApp.tasks import send_email_with_delay

def send_email_view(request):
    # Trigger the Celery task asynchronously
    result = send_email_with_delay.delay("fareenansari33@gmail.com", "Hello, Celery!")

    # You can return a response to the user
    return HttpResponse(f"Email task started with task ID: {result.id}")
</pre>

# Djangi-models
<pre>
on_delete, parameters:
1. CASCADE: if parent obj is deleted its child obj will get deleted.
2. PROTECT: will not delete the related object, unless you delete the child obj first.
3. SET_NULL
4. SET_DEFAULT
5. DO_NOTHING
</pre>

# Pandas
<b>1. DataFrame</b>
<pre>

import pandas as pd
data = {
    'Name': ['Alice', 'Bob', 'Charlie'],
    'Age': [25, 30, 35],
    'City': ['New York', 'San Francisco', 'Los Angeles']
}

df = pd.DataFrame(data)

O/p:
      Name  Age           City
0    Alice   25       New York
1      Bob   30  San Francisco
2  Charlie   35    Los Angeles
</pre>

<b>2. Series</b>
<pre>
import pandas as pd
# Creating a Series with custom index labels
data = {'Alice': 25, 'Bob': 30, 'Charlie': 35}
my_series = pd.Series(data)

# Print the Series
print(my_series)

O/p:
Alice      25
Bob        30
Charlie    35
dtype: int64
</pre>

<b> Input/Output in pandas</b>
<pre>
Reading:
1. read_csv:
eg: df = pd.read_csv('data.csv')

2. read_excel
3. read_json
4. read_sql

Writing:
same functions like above but instead of read it is to_csv, to_sql etc

Example: to_csv
import pandas as pd
df = {
        'alpha' : ['a', 'b', 'c', 'd'],
        'data' : [10, 20, 30, 40]
     }
my_series = pd.DataFrame(df)
my_series.to_csv('output.csv', index=False)
</pre>

# Decorators, kafka, ORM, class & methods, DRF
