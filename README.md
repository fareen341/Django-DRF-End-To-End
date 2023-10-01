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

<b>Difference between get() and filter()</b>
<pre>
filter() is used to fetch multiple records and get() is used to retrive single records.
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

# Django-models
<pre>
on_delete, parameters:
1. CASCADE: if parent obj is deleted its child obj will get deleted.
2. PROTECT: will not delete the related object, unless you delete the child obj first.
3. SET_NULL
4. SET_DEFAULT
5. DO_NOTHING
</pre>

<b>db_index in models</b>b>
<pre>
The db_index attribute is used to create database indexes on specific fields to improve query performance when filtering or ordering by those fields.
Eg:
product = models.CharField(max_length=13, db_index=True)

<b>Performing complex query:</b>
1. extra methor:
queryset = MyModel.objects.extra(select={"custom_field": "SELECT ... FROM ..."})

2. Using F() expressions:
from django.db.models import F
queryset = MyModel.objects.filter(field1=F('field2'))

3. Using Q()
from django.db.models import Q
queryset = MyModel.objects.filter(Q(field1=value1) | Q(field2=value2))

4. using() function
queryset = MyModel.objects.using('other_db').filter(...)

</pre>

# Pandas
In pandas we have series and dataframe, input/output, data cleaning like missing and null value, merging and joinig and plot.
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

<b>loc & iloc in pandas</b>
<pre>
Indexing:
<b>loc example:</b>

import pandas as pd

data = {'A': [1, 2, 3, 4, 5],
        'B': [10, 20, 30, 40, 50],
        'C': ['apple', 'banana', 'cherry', 'date', 'elderberry']}

df = pd.DataFrame(data, index=['row1', 'row2', 'row3', 'row4', 'row5'])

O/p:
        A   B          C
row1  1  10      apple
row2  2  20    banana
row3  3  30    cherry
row4  4  40    date
row5  5  50    elderberry

selected_data = df.loc[[row], [column]]
selected_data = df.loc[['row1', 'row3'], ['A', 'C']]

O/p:
      A   B      C
row1  1  10  apple

selected_column = df.loc[:, 'B']

<b>iloc: select by integer position.</b>
Example:
selected_column = df.iloc[:, 1]

O/p:
row1    10
row2    20
row3    30
row4    40
row5    50
Name: B, dtype: int64
</pre>

<b>Data Visualization</b>
<pre>
Built-in plotting methods like plot(), hist(), boxplot(), etc.

Step 1: Create or load data.
Step 2: Convert into dataframe/series.
Step 3: Plot, using df.plot(kind)

Example:
bar plot:
import pandas as pd
data = {'Category': ['A', 'B', 'C'], 'Value': [10, 20, 15]}
df = pd.DataFrame(data)
df.plot(x='Category', y='Value', kind='bar')

histogram:
import pandas as pd
series = pd.Series([1, 2, 2, 3, 3, 3, 4, 4, 5])
series.plot(kind='hist', bins=5)
</pre>

<b>Data Merging and Joining</b>
<pre>
1. We can perform sql like join of two dataframe:
2. The how parameter specifies the type of join (inner, outer, left, right).
3. You can also join multiple DataFrames by chaining .join().

import pandas as pd
left = pd.DataFrame({'key': ['K0', 'K1', 'K2'],
                    'value_left': [1, 2, 3]})
right = pd.DataFrame({'key': ['K1', 'K2', 'K3'],
                     'value_right': [4, 5, 6]})
result = pd.merge(left, right, on='key', how='left')  # Left join

O/p:
key	value_left	value_right
0	K0	1	NaN
1	K1	2	4.0
2	K2	3	5.0


Example 2: using .join()
  
import pandas as pd
left = pd.DataFrame({'A': ['A0', 'A1', 'A2']},
                    index=['K0', 'K1', 'K2'])
right = pd.DataFrame({'B': ['B0', 'B1', 'B2']},
                     index=['K1', 'K2', 'K3'])
result = left.join(right, how='inner')

O/p:
	A	B
K1	A1	B0
K2	A2	B1
</pre>

# Numpy
Numpy is all about numeric calculations, it works well with array, we get maths functions easily in numpy like sqrt, add, sin, power etc, generate range, generate random number.
<pre>
import numpy as np

# Creating an array from a Python list
my_list = [1, 2, 3, 4, 5]
arr_from_list = np.array(my_list)

# Creating an array filled with zeros
zeros_array = np.zeros(5)

# Creating a 2D array filled with ones
ones_array = np.ones((2, 3))

# Creating an array with a range of values
range_array = np.arange(0, 10, 2)

# Creating a linearly spaced array
linspace_array = np.linspace(0, 1, 5)

# Creating a random array with values between 0 and 1
random_array = np.random.rand(3, 4)
</pre>

# Management commad
<pre>
-makemigrations vs migrate
makemigrations is used to generate migration files that represent changes to your database schema. It doesn't apply any changes to the database itself. 
Where as migarte apply those changes.
</pre>

# Django Production/deployment
<pre>
In django settings file:
1. debug = False
Meaning the django application will run on production if debug is False. In production enviromnet it does:
i. Django runs in production mode.
ii. Detailed error pages are replaced with a generic error message to avoid exposing sensitive information to users.
iii. Automatic code and template reloading are disabled to improve performance.
iv. Static files are typically served by a separate web server (e.g., Nginx or Apache) or a CDN (Content Delivery Network) for better performance and security.
v. Security features like clickjacking protection are enabled.
vi. Performance optimizations are often turned on. 

<b>gunicorn vs web server</b>
gunicorn and nginx both are web server, but using both togther is best practice, we can just use gunicorn also for small scale project.
In practice, the most common and recommended approach for Django applications in a production environment is to use Nginx + Gunicorn. 
This combination provides a balance between security, performance, and scalability. Nginx handles security and static file serving, while Gunicorn executes Python code.
</pre>

# DRF
1. Serializers & Deserializers:
Serializers convert complex Python data types (such as Django model instances) into a format that can be easily rendered into JSON, XML, or other content types. Deserializers is vice versa.

2. ViewSet:
ViewSets automatically generate views for common CRUD operations (e.g., creating, retrieving, updating, deleting) based on the methods you define within the ViewSet class. We have following methods: list(), retrive(), update(), partial_update(), destroy().

3. Routers:
In Django Rest Framework (DRF), routers are a convenient way to automatically generate URL patterns for your API views, particularly for ViewSets. Routers simplify the process of defining and organizing URL patterns for CRUD (Create, Read, Update, Delete) operations on your API resources. DRF includes a built-in SimpleRouter class that you can use to create default URL patterns for your ViewSets. Here's how routers work in DRF.


# Decorators, kafka, ORM, class & methods, DRF
