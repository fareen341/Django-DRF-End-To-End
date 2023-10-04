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

# Decorators, kafka, ORM, class & methods, DRF
