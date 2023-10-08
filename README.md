
# This repository contains the things to learn for python developer job, read all below stuff is enpugh, foo python, django and DRF.

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

<b>Performing complex query.</b>
1. extra methor:
<pre>queryset = MyModel.objects.extra(select={"custom_field": "SELECT ... FROM ..."})</pre>

2. Using F() expressions:
<pre>
from django.db.models import F
queryset = MyModel.objects.filter(field1=F('field2'))
</pre>

3. Using Q()
<pre>
from django.db.models import Q
queryset = MyModel.objects.filter(Q(field1=value1) | Q(field2=value2))
</pre>

4. using() function
<pre>queryset = MyModel.objects.using('other_db').filter(...)</pre>

<b>Using exclude vs Q</b>
<pre>
# If we want to exclude and filter togther:
Book.objects.exclude(title="ecs").filter(title="s3")

# we can also use Q
Book.objects.filter(~Q(title="ecs"), Q(title="s3"))
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
For this to work we need matplotlab module.

Step 1: Create or load data.
Step 2: Convert into dataframe/series.
Step 3: Plot, using df.plot(kind)

Example:
bar plot:
import pandas as pd
import matplotlib.pyplot as plt
data = {'Category': ['A', 'B', 'C'], 'Value': [10, 20, 15]}
df = pd.DataFrame(data)
df.plot(x='Category', y='Value', kind='bar')
OR
plt.show()

histogram:
import pandas as pd
import matplotlib.pyplot as plt
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

# Management commad (makemigrations vs migrate)
1. Makemigrations is used to generate migration files that represent changes to your database schema. It doesn't apply any changes to the database itself. 
2. Where as migarte apply those changes. For example we have added default price as $ 200, so makemigration will created a database schema with default values, but does not change the values of the db. Like i have data in database which has no price, so after migrate changes will get applied and that price will become $ 200 automatically.
3. It is very important to review migration when working on production db.
4. Here in models first i dont have default value, second time i give default values in models from that migrations onwards the default values will be apply.
5. Do migratation of your app only and always do show migrations and review migrate:
<pre>
$ python manage.py makemigrations myapp
$ python3 manage.py showmigrations

Review recent migrations and apply:
$ python manage.py migrate myapp

Warning: this will do the changes on the actual database.
</pre>
6. If we want to undo the migrations
<pre>
First do show migrations:
$ python3 manage.py show migrations

 [X] 0001_initial
 [X] 0002_author_book
 [X] 0003_rename_name_author_auther_name
 [X] 0004_rename_auther_name_author_name
 [X] 0005_price
 [X] 0006_price_book_alter_price_price
 [X] 0007_alter_book_title
 [X] 0008_alter_book_title
 [X] 0009_alter_book_title
 [X] 0010_person
 [X] 0011_alter_person_location
 [X] 0012_alter_person_location
 [X] 0013_alter_person_location


To undo the migrate, this will not change the models file
$ python3 manage.py migrate BlogApp 0010_person

 [X] 0001_initial
 [X] 0002_author_book
 [X] 0003_rename_name_author_auther_name
 [X] 0004_rename_auther_name_author_name
 [X] 0005_price
 [X] 0006_price_book_alter_price_price
 [X] 0007_alter_book_title
 [X] 0008_alter_book_title
 [X] 0009_alter_book_title
 [X] 0010_person
 [ ] 0011_alter_person_location
 [ ] 0012_alter_person_location
 [ ] 0013_alter_person_location

The remove of X meaning the unmigrations in done.

Instead do the changes in the models file itself and do makemigrations and migrate
</pre>

7. Important Points: </br>
   i. Always do migration on your app only.</br>
   ii. Always take backup of your database before doing the migrate cuz migrate will apply the changes on database.</br>
   iii. Review with team members before applying migrate on production db. </br>

# Inheritance in django models.
1. Abstract base class inheritance.
<pre>
1. We use this when we want a common class as in, in school db name and age will be common.
2. It'll become abstract base class when we write abstract = True.
3. No table will be created for abstract base class.

Example:
Parent class

class CommonInfo(models.Model):
	name = models.CharField(max_length = 70)
	age = models.IntegerField
	
	class Meta:
		abstract = True

Child class

class Student(CommonInfo):
	fees = models.IntegerField()
</pre>

2. Proxy Model.</br>
i. Proxy Model and Abstract class both are same, but the main difference is in there concept. Use abstract class when we want to have common fields for multiple tables. Use proxy model when we want to inherit functionality of parent to child, it does not follow concept like use it for common fields, instead it is basically inheritance.  </br>
ii. Both the parent and child class share the same database table.  </br>
iii. It become proxy when we give proxy = True. </br>
iv. The class which have proxy = True or abstract = True, there table won't be created.</br>

3. Multi-table Inheritance
<pre>
1. The structure of this is same as abstract, here both will have seperate database table.
2. The parent and child will maintain one to one relation.
3. Passport table and person table both will have one to one relation.
Example:


Parent class 

class PersonDetail(models.Model):
	name = models.CharField(max_length = 70)
	age = models.InetegerField()


Child class

class PassportDetail(PersonDetails):
	passport_no = models.IntegerField()
</pre>

# Django Generic View
Django generic view provides some common functionality for CRUD operations. Like ListView, CreateView, DetailView, UpdateView, DeleteView.
Example:
<pre>
from django.views.generic.edit import UpdateView
from .models import MyModel

class MyModelUpdateView(UpdateView):
    model = MyModel
    template_name = 'myapp/mymodel_form.html'
    fields = ['field1', 'field2']
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
Github: https://github.com/encode/django-rest-framework/tree/master/rest_framework

# Serializers & Deserializers:
Serializers convert complex Python data types (such as Django model instances) into a format that can be easily rendered into JSON, XML, or other content types. Deserializers is vice versa.

# Routers:
1. In Django Rest Framework (DRF), routers are a convenient way to automatically generate URL patterns for your API views, particularly for ViewSets. Routers simplify the process of defining and organizing URL patterns for CRUD (Create, Read, Update, Delete) operations on your API resources. DRF includes a built-in SimpleRouter class that you can use to create default URL patterns for your ViewSets. Here's how routers work in DRF.
2. We have DefaultRouter 7 SimpleRouter, both have same functionality.

# Authenticatios - DRF provides:
1) BasicAuthentication: uname and pswd(not for prod env).
2) SessionAuthentication: Django's built-in session framework to authenticate users.
3) TokenAuthentication:  clients obtain an authentication token to use with API requests.
4) RemoteUserAuthentication: 
5) CustomAuthentication.
6) Thirt party: JWT Token, many packages are available for this authentication.

Example:
<pre>
<b>class based view:</b>

class MyApiView(APIView):
    authentication_classes = [TokenAuthentication]
    permission_classes = [IsAuthenticated]

<b>function based view: for function based view we need to use decorators:</b>

from rest_framework.decorators import authentication_classes, permission_classes

@authentication_classes([TokenAuthentication, SessionAuthentication])
@permission_classes([IsAuthenticated])
def my_function_based_view(request):
    # Your view logic here
    return Response({'message': 'Authenticated user can access this endpoint.'})

</pre>

# Permissions
Permissions are used to grant or deny access for different classes of users to different parts of the API.
Default permission class is IsAuthenticatedOrReadOnly.
1. AllowAny: everyone without passwd.
2. isAuthenticated: Every users including superuser, normal user, staff user can loging with there username and pswd, and deny if he's unauthenticated. 
3. IsAdminUser
4. IsAuthenticationOrReadOnly - authenticated user can perform write permission and other all users(anoymous) can read only.


# Paginator: usefull when we have large amount of data, so reduce the amount of data into chunks.
<pre>
pagination_class = PageNumberPagination

/blogposts/?page=1 will return the first page of blog posts.
/blogposts/?page=2 will return the second page.
And so on...
</pre>

# api_view() decorator:

The @api_view decorator in Django Rest Framework (DRF) is used to define and configure views for your API endpoints in a simple and concise manner. It allows you to create views using function-based views (FBVs) rather than class-based views (CBVs). The primary purpose of the @api_view decorator is to convert a regular Django view function into a DRF-compatible view that can handle HTTP requests and return appropriate responses.

<pre>
Example:

@api_view(['GET'])  # Decorate the view and specify allowed HTTP methods
def item_list(request):
    items = ['item1', 'item2', 'item3']  # Replace with actual data retrieval logic
    return Response({'items': items})
</pre>


# File upload in DRF.
In Django Rest Framework (DRF), handling file uploads is relatively straightforward. You can handle file uploads by using DRF's built-in FileUploadParser or MultiPartParser, combined with serializers and views designed to handle file fields. Here are the steps to handle file uploads in DRF:

<pre>
Example:
from rest_framework import serializers

class FileUploadSerializer(serializers.Serializer):
    file = serializers.FileField()


views.py
from rest_framework.parsers import MultiPartParser
from rest_framework.response import Response
from rest_framework.views import APIView

class FileUploadView(APIView):
    parser_classes = [MultiPartParser]

    def post(self, request, *args, **kwargs):
        serializer = FileUploadSerializer(data=request.data)

        if serializer.is_valid():
            # Handle the uploaded file, e.g., save it to a specific location
            uploaded_file = serializer.validated_data['file']

            # You can access file properties like name, size, and content
            file_name = uploaded_file.name
            file_size = uploaded_file.size

            # Process the file or perform any desired operations
            # ...

            return Response({'message': 'File uploaded successfully'})
        else:
            return Response(serializer.errors, status=400)

</pre>

# ViewSet:
ViewSets automatically generate views for common CRUD operations (e.g., creating, retrieving, updating, deleting) based on the methods you define within the ViewSet class. We have following methods: list(), retrive(), update(), partial_update(), destroy().


# GenericViewSet:
In Django Rest Framework (DRF), a GenericViewSet is a class-based view that provides a flexible way to define API views for working with Django models. It combines the functionality of several mixins and allows you to create custom views for your models.

<pre>
from rest_framework import viewsets
from .models import MyModel
from .serializers import MyModelSerializer

class MyModelViewSet(viewsets.GenericViewSet, 
                     viewsets.mixins.ListModelMixin, 
                     viewsets.mixins.CreateModelMixin,
                     viewsets.mixins.RetrieveModelMixin,
                     viewsets.mixins.UpdateModelMixin,
                     viewsets.mixins.DestroyModelMixin):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer

and use router for urls
</pre>

# ModelViewSet:
In Django Rest Framework (DRF), a ModelViewSet is a class-based view that combines the functionality of several other classes to simplify the creation of views for Django models. It is a convenient way to create views that perform common CRUD (Create, Read, Update, Delete) operations on a model.

<pre>
from rest_framework import viewsets
from .models import MyModel
from .serializers import MyModelSerializer

class MyModelViewSet(viewsets.ModelViewSet):
    queryset = MyModel.objects.all()
    serializer_class = MyModelSerializer


and use router to define urls.

<b>We also have ReadOnlyModelViewSet: only allow list and retrive methods</b>
</pre>


# APIView
The APIView class in Django Rest Framework (DRF) is a foundational component that allows you to create custom API views in a flexible and powerful way. It serves as the base class for creating views that handle HTTP requests and return HTTP responses in a RESTful API. Here are the key aspects and uses of the APIView class. Provides methods for handling different HTTP methods such as GET, POST, PUT, PATCH, DELETE

Example:
<pre>
from rest_framework.views import APIView

class ItemListView(APIView):
    def get(self, request):
        # Retrieve a list of items (replace with your data retrieval logic)
        items = ['item1', 'item2', 'item3']
        return Response({'items': items})
</pre>

# GenericAPIView.
1. Like it provides the generic functionality unlike in APIView in which we need to provide the custom logic for all methods
2. GenericAPIView is designed to work with models and provides a set of predefined views for common CRUD (Create, Read, Update, Delete) operations on model instances.
3. It is often used in combination with DRF mixins like ListAPIView, CreateAPIView, RetrieveAPIView, UpdateAPIView, and DestroyAPIView to create views for standard operations.

<b>Method 1: using seperate APIViews.</b>
<pre>
from rest_framework.generics import GenericAPIView, ListAPIView

class YourModelListView(ListAPIView):
    queryset = YourModel.objects.all()
    serializer_class = YourModelSerializer
</pre>

<b>Method 2: Grouping the views</b>
<pre>
from rest_framework.generics import ListCreateAPIView, RetrieveUpdateAPIView, RetrieveDestroyAPIView, RetrieveUpdateDestroyAPIView

#If we need Retrive update destroy all together
class StudentRetriveUpdateDestroy(RetrieveUpdateDestroyAPIView):
	queryset=Student.objects.all()
	serializer_class=StudentSerializer

urls.py
path('studentapi/', views.StudentListCreate.as_view()),
path('studentapi/int:pk', views.StudentRetriveUpdateDestroy.as_view()),

</pre>

# GenericAPIView with mixins.
In Django Rest Framework (DRF), mixins are classes that provide pre-defined behaviors and functionality that you can include in your views by inheriting from them. When used in conjunction with GenericAPIView, mixins help you create views for common CRUD (Create, Read, Update, Delete) operations with minimal code duplication. Here are some common mixins used with GenericAPIView:

<b>Method 1: using different mixins like CreateModelMixin for creating and another class for update etc.</b>

<pre>
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin

class YourModelCreateView(mixins.CreateModelMixin, GenericAPIView):
    queryset = YourModel.objects.all()
    serializer_class = YourModelSerializer

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

in above code we need to replace self.create with other functions accordingly, like self.update() to update api.
</pre>

<b>Method 2: grouping the mixins, like create and update togther in one class.</b>
We can group according to pk. For list and create we dont need pk, rest for all we need pk so group them according to pk.

<pre>
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin

class LCStudent(GenericAPIView, ListModelMixin, CreateModelMixin):        #LC: List, Create (pk not required)
	queryset=Student.objects.all()
	serializer_class=StudentSerializer

	def get(self, request, *args, **kwargs):
	    return self.list(request, *args, **kwargs)

	def post(self, request, *args, **kwargs):
	    return self.create(request, *args, **kwargs)

#update, destroy, retrive: need pk so group them.

urls:
path('studentapi/',views.LCStudent.as_view()),
path('studentapi/int:pk',views.RUDStudent.as_view()),
</pre>

<b>JWT Steps</b>
Refer old repo.


# Throttling
1. Your API might have a restrictive throttle for unauthenticated requests, and a less restrictive throttle for authentication requests.
2. By default, a user or client making API requests to a DRF view will be limited to 100 requests within a 24-hour period. This is a very low limit and is primarily meant for demonstration and testing purposes.
3. We have AnonRateThrottle, UserRateThrottle & Scope Rate Throttle.
4. In most cases, when a user refreshes a web page in their browser, it typically generates a new HTTP request to the server. This new request is treated as a separate request by the server and, depending on your server's configuration and any rate limiting or throttling mechanisms in place, it may count as an additional request.
	
<pre>
For globally
in settings.py
REST_FRAMEWORK = {
'DEFAULT_THROTTLE_CLASSES':[
'rest_framework.throttling.AnonRateThrottle',
'rest_framework.throttling.UserRateThrottle'
],
'DEFAULT_THROTTLE_RATES':{
#rates can be include in second, minute, hour or day
'anon':'100/day',       #only unauthenticated users can only login for 100 times per day
'user':'1000/day'       #authenticated users can login only 1000 times per day
}
}
</pre>

<b>Scope rate throtte</b>
<pre>
Example: this is used when we want to set different throtte limit to different api.
# settings.py

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_RATES': {
        'custom_scope_1': '1000/day',    # Example: 1000 requests per day for views with scope 'custom_scope_1'
        'custom_scope_2': '100/hour',    # Example: 100 requests per hour for views with scope 'custom_scope_2'
    },
}


class MyView(APIView):
    throttle_scope = 'custom_scope_1'
    # ...

class MyViewSet(viewsets.ModelViewSet):
    throttle_scope = 'custom_scope_2'
    # ...
</pre>

# Filter, Search & Ordering
1. Filter: used to filter in drf, it gives filter option in UI. 
2. Search: It gives search option in UI.
3. Gives ordering option in UI.
On postman it'll look like this: GET /api/books/?ordering=title sorts books by title in ascending order. And use minus for descending, same for filter and search too on postman.
<pre>
Example: 
from django_filters.rest_framework import DjangoFilterBackend, SearchFilter

class StudentList(ListAPIView):
	queryset=Student.objects.all()
	serializer_class=StudentSerializer
	# filter option
	filter_backends = [DjangoFilterBackend]
	filterset_fields = ['city']

	# search option
	filter_backends = [SearchFilter]
	search_fields = ['name','city']	
	
	# ordering option
	filter_backends = [OrderingFilter]
	ordering_fields = ['city']
</pre>

# CORS
1. If you want to run a third-party script in your Django Rest Framework (DRF) application, and that script makes requests to your DRF API from a different domain or origin, you may need to enable Cross-Origin Resource Sharing (CORS) to allow those requests.
2. We need one package, $ pip install django-cors-headers
<pre>
INSTALLED_APPS = [
    # ...
    'corsheaders',
    # ...
]

MIDDLEWARE = [
    # ...
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    # ...
]

CORS_ALLOW_ALL_ORIGINS = True  # Allow requests from any origin (for development)
# or specify allowed origins explicitly:
# CORS_ALLOWED_ORIGINS = ['https://example.com', 'https://another-domain.com']
</pre>


# Python Basics:
We have str, list, tuple, set and dict. </br>
<b>Operations on list</b>
<pre>
Functions of list:
li = [1,2,8,4,9,3]

1. copy() function:
eg: new_li = li.copy()        # new_li will be same as li

2. reverse() function:
eg: li.reverse()        # [3, 9, 4, 8, 2, 1]

3. sort() function:
eg: li.sort()         # [1, 2, 3, 4, 8, 9]

4. remove() function:    it'll remove one item at a time, if we have two '3''s then it'll remove the first occurance from left to right.
li.remove(1)        # [2, 3, 4, 8, 9]

5. count() function:
li.count(3)            # 1

6. len()
len(li)        # 7
</pre>

<b>Operations on set</b>
1. insertion order are not preserved
2. duplicate are not allowed
3. indicated by{}
4. set item are not indexed so slicing and indexing not allowed
5. immutable
6. we can add and remove item in set but cannot change the value by using index.
7. main fucntionality of set is, union, difference & intersetion
<pre>
Example:
Union:
a={1,2,3,4,5}
b={4,5,6,7,8,9}
c = a | b      # Output: {1, 2, 3, 4, 5, 6, 7, 8, 9}

Intersetion:
set1 = {1, 2, 3}
set2 = {3, 4, 5}
intersection_set = set1 & set2
print(intersection_set)  # Output: {3}

Difference:
set1 = {1, 2, 3, 4, 5}
set2 = {3, 4, 5, 6, 7}
difference_set = set1 - set2
print(difference_set)  # Output: {1, 2}
</pre>

<b>Operations on dict</b>
1. duplicated keys are not allowed
2. indexing and slicing not allowed
3. we can check the value of a perticuar key by giving its key
<pre>

d = {"red": 1, "blue": 2, "pink": 3, "black": 4}

<b>Get the keys.</b>
d.keys()         # dict_keys(['red', 'blue', 'pink', 'black'])

<b>Get the values.</b>
d.values()        # dict_values([1, 2, 3, 4])

<b>Get the values using get().</b>
d.get("red", "black")        # 1
d.get("cyan", "black")       # black

<b>Update the value of a key, or update the dict.</b>
d["red"] = 55                           # {'red': 55, 'blue': 2, 'pink': 3, 'black': 4}
d.update({"new_color": "green"})        # {'red': 5, 'blue': 2, 'pink': 3, 'black': 4, 'new_color': 'green'}
</pre>

<b>Map, Filter, Reduce</b>
1. Map: Used when we want to do operation on all items on a list.
2. Filter: Used when we want to do operations to get a single values, example to get the greatest value etc.
3. Reduce: When we want to get one values, example sum etc.

<b>Operations on string</b>
<pre>
Slicing:

name = "fareen"
name[::-1]    # reverse string neeraf

In positive i.e from left to right it add -1, and for negative from right to left it does +1
Example:
name[0:3]        # far
name[0:-3]        # far

</pre>

<b>Operators</b>
<pre>
<b>1. Arithmetic operator: </b>
/ is used for getting the actual division like we get in calculator, // is used to get quotent and % is used for getting the reminder. 

/ : used for divide		# always give floating value, will calculate the normal division
Example:
a=10
b=3
print(a / b)			# 3.3333333333333335

//: used for quotent
Example:
a=10
b=3
print(a // b)			# 3

%: used for reminder
a=8
b=3
print(a % b)			# 2

<b>Logical Operator: and, or and not</b>
If we do:
a = 10
b = 20

print(a and b)		# 20

It gave left hand value, we need to give condition in both side, like a > 10 and b < 20, likewise.

<b>Opeartor and precedence in python. All have left to right</b>
1. ()
2. **			# only this have right to left precedence
3. * / % //
4. + -

Question: 5 * 2 // 2 * (2 + 3)
So going for bracket first:	5 * 2 // 2 * 5
Now braket done, now go from left to right cuz all have same precedence.
So o/p:  10 // 2 = 5,   5 * 5 = 25

<b>in and is in python.</b>
For string: name = "fareen",  
'a' in name		# True

For lits: nums = [1,2,3,4,5,6]
1 in nums		# True

is is used to check for memory location, two values such as:
a = 10, b = 10
print(a is b)			# True, cuz values is same but when b = 20 the it'll be False
</pre>

<b>Extra functions:</b>
<pre>
1. Memory location
a = 10
id(a)			# will give memory location of a

2. chr() function, to get number from unicode
chr(65)			# A

3. ord() function to get unicode from character
ord('A')		# 65

4. floor and ceil
from math import floor, ceil

floor(4.5)		# 4
floor(4.8)		# 4

ceil(4.5)		# 5
ceil(4.1)		# 5
</pre>

<b>Extra points</b>
<pre>
1. Floating point error:
0.1 + 0.2			# 0.30000000000000004

Getting wong error, we should get 0.3

2. When we do -4 // 3, we should get one as quotent, but we get -2, 
-4 // 3		# -2

This is cuz it does the floor division, in case of minus floor acts as ceil and vice versa.

3. The __init__ method: it is called automatically when you create a new instance of a class (an object). 
It is the constructor for the class, and it allows you to perform any necessary setup or initialization for the object.
</pre>

<b>Function in python.</b>
<pre>
1. Non default argument should be after. Example: def sum(x, y, z=10).
2. In case of args and kwargs, args should come first then kwargs, args support list, tuple, kwargs support python dict. def nums(*args, **kwargs).
</pre>

# Programs Practice
1. Get the 2nd larget value in a list
2. The given list ["abc", "def"] convert the list in [["a", "b", "c"], ["d", "e", "f"]].
3. Reverse the name "fareen".
4. Fine a pallindrome.
5. Print pattern:
<pre>
*
**
***
****
*****
</pre>
6. What is the output: 
<pre>
1. 2 + 3 * 4 ** 2 / 2
2. 3 ** 3 ** 2
</pre>
7. Find a number is armstrong or not.
<pre>
Cube of all numbers and addition will give the same number.
For example, let's take the 3-digit number 153:

Calculate each digit raised to the third power, it is pow of 3 not num ** 3:

1^3 = 1
5^3 = 125
3^3 = 27

1 + 125 + 27 = 153
</pre>
7. Remove duplicate from a list num_list = [3,4,6,3,4,7].
8. Length of last word str = "python is great, it has many features".
9. In the given list arr = [5, 32, 45, 4, 12, 18, 25], do minus of first element with second.
10. Find all the pairs which has sum = 17 in the given list arr = [5, 7, 4, 3, 9, 8, 19, 21].
11. Find all the missing values in list arr = [1,2,3,5,6,7,8,10,17].
12. Convert two list into dict x = [1,2,3,4], y = ["a", "b", "c", "d"].
13. Find the count of word "python" in the given string, str = "python is great, it has many features, python".
14. Find the common element in two str, name = "Seema" and name = "FarEen", by repetation of words and without repetation also. Also ignoring the case.

# Program with solution
1. Shortest way to check for pallindrome.
<pre>
name = "madam"
if name == name[::-1]:
	print("pallindrome")
else:
	print("not a pallindrome")	
</pre>

# Interview Questions
<b>Python</b>
1. Difference between method and class?

<b>Django</b>
1. Difference between Abstract base class inheritance, Multi-table inheritance, Proxy Model?
2. Difference between django-admin and manage.py?
3. What are the common excptions in django?
4. Authentication & Authorization in Django?
5. Django Security?
6. Django Middleware?
7. Generic view in django?
8. Does django support multiple primary key, if not how to achive this?
9. Queryset in django?
10. Request response lifecycle.
11. ORM:
<pre>
i. Difference between select_related & prefetch related.
ii. Use of Q().
iii. Use of F().
iv. How to group by.
</pre>

<b>DRF</b>
1. Difference between put and patch?



# Interview Questions with answer:
<b>Python</b>

<b>Django Ansswers</b>
1. Difference between method and class?
<pre>
If it is inside the class then it is called as method and if it is outside of class the called as function.
</pre>
2. Difference between django-admin and manage.py?
<pre>
Both are command-line utility provided by django. django-admin is used for administrative tasks.
</pre>
3. What are the common exceptions in django?
<pre>
ObjectDoesNotExist, PermissionDenied, IntegrityError etc
</pre>
4. Authentication & Authorization in Django?
<pre>
Django provides build-in Authentication, along with Authentication it also provide authorization like users, groups & permissions etc
</pre>
5. Django Security?
<pre>
Django automatically provides some coommon security issues like:
i. SQL injection attack cuz we use ORM.

ii. CSRF attack:  CSRF protection ensures that actions initiated by a user, such as submitting a form or making a request, 
are only accepted if they come from a trusted source, and not from a malicious website or attacker-controlled source. 

iii. Pswd hashing

iv. Authentication & Authorization

v. XSS protection: XSS attacks occur when an attacker injects malicious scripts (usually JavaScript) into web pages viewed 
by other users. These scripts can then execute within the context of the victim's browser, potentially stealing user data, 
hijacking sessions, or performing other malicious actions.

Along with these security, we should make our web server secure also, cuz request first hit web server so we can avoid DDOS 
at web server level and continue.
</pre>

6. Middleware in django?
<pre>
Django provide some middleware like session, message & csrf middleware, along with that we can have custom middleware also.
</pre>

8. Does django support multiple primary key?
<pre>
Django does not support multiple column primary key by default, to achieve this we can use unique_together instead
</pre>

9. Queryset in django?
<pre>
Querysets are used for retrieving, updating, or deleting data from the database using Django's Object-Relational Mapping (ORM) system.
In short queryset is used to work with database using ORM.
</pre>

<b>DRF</b>


# Decorators, kafka, ORM, class & methods, DRF
