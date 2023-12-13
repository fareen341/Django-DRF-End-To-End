# Django
# Django ORM
1. Group By: Get count of views by views, meaning group by using views:
<pre>
  #group by only views:
  queryset = Blog.objects.values('views').annotate(view_count=Count('views'))

  #group by name and views, whatever we put in values() that becomes group by
  queryset = Blog.objects.values('views', 'name').annotate(view_count=Count('id'))

<b>Another use of annotate</b>
Annotate is also used to do calculation on database fields.
Consider example:

Books.objects.annotate(
    actual_price = ExpressionWrapper(
        F('total_price') - F('discount_price')
    ),
    output_field = FloatField()
).filter(actual_price__gt=200)
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

2. Using F() expressions:<br>
In Django, an F() expression is a way to reference a database field and perform operations on its values at the database level. 
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
<pre>
<b>Abstract</b>
# This is common class, we cannot create object of this class also cant register it on admin
# No db table would be created
class People(models.Model):
    name = models.CharField(max_length=200)
    age = models.IntegerField()

    class Meta:
        abstract = True

class Employee(People):
    salary = models.IntegerField()

<b>Proxy</b>
# This is like normal inheritance
# Db table will be created
class People(models.Model):
    name = models.CharField(max_length=200)
    age = models.IntegerField()

# we cannot give more fields like in above abstract example of salary
# proxy models should not have their own fields; they should only inherit fields and behavior from the base model.
# whenever we create Employee object, it'll also get created in Person too, vice versa is not true.
class Employee(People):
    class Meta:
        proxy = True
	ordering = ['last_name', 'first_name']

<b>Multi-Table</b>
# unlike proxy we can add other fields in multi-table, both parent and child will have different table.
# So Employee will link to parent People via Foreign key 
# whenever we create Employee object, it'll also get created in Person too, vice versa is not true.
# will create one to one relation between both table.
class People(models.Model):
    name = models.CharField(max_length=200)
    age = models.IntegerField()

class Employee(People):
    salary = models.IntegerField()

So:
$ Employee.objects.all().values()[0]
{'id': 3, 'name': 'Fareen', 'age': 23, 'people_ptr_id': 3, 'salary': 3657}

$ People.objects.filter(id=3).values()
<QuerySet [{'id': 3, 'name': 'Fareen', 'age': 23}]>


<b>So the difference in proxy and multitable is we can give extra column in multi-table whereas not possible in proxy,
proxy is basically used when we want to modify its functionality like in above i did ordering
Altho parent and child is same but child will have extra functionality</b>
no new database table is created; both the parent model and the proxy model share the same database table. 
While the database structure is the same, you can indeed add different functionality to the parent and proxy models, allowing them to behave differently.
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
3. In Django REST framework (DRF), the default router is a convenient tool provided by DRF to simplify the process of defining URL patterns for your API views. It automatically generates URL patterns for the standard CRUD (Create, Retrieve, Update, Delete) operations based on your views and viewsets.

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
<b>For it to work we need:</b>
Install and include in installed_apps:
pip install django-filter



from django_filters.rest_framework import DjangoFilterBackend
from rest_framework.filters import SearchFilter
from rest_framework.filters import OrderingFilter

class StudentList(ListAPIView):
	queryset=Student.objects.all()
	serializer_class=StudentSerializer
	filter_backends = [DjangoFilterBackend, SearchFilter, OrderingFilter]
	# filter option
	filterset_fields = ['city']

	# search option
	search_fields = ['name','city']	
	
	# ordering option
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

# Complete DRF API Example
<pre>
1. pip install django-rest_framework
2. write in installed apps: rest_framework
3. migrate

4. create models:
class Employee(models.Model):
    name = models.CharField(max_length=200)
    age = models.IntegerField()
    salary = models.IntegerField()

5. create serializers.py:
from rest_framework import serializers
from .models import Employee

class EmployeeSerializers(serializers.ModelSerializer):
    class Meta:
        model = Employee
        fields = '__all__'

6. create views:
from django.shortcuts import render
from .models import Employee
from .serializers import EmployeeSerializer
from rest_framework.response import Response
from rest_framework import status, viewsets
from rest_framework.authentication import SessionAuthentication
from rest_framework.permissions import IsAuthenticated


class EmployeeViewSet(viewsets.ViewSet):
    authentication_classes = [SessionAuthentication]
    permission_classes = [IsAuthenticated]

    def list(self, request):
        stu=Employee.objects.all()
        serializer=EmployeeSerializer(stu, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        id=pk
        if id is not None:
            stu=Employee.objects.get(id=id)
            serializer=EmployeeSerializer(stu)
            return Response(serializer.data)

    def create(self, request):
        serializer=EmployeeSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Data Created'},status=status.HTTP_201_CREATED)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def update(self, request, pk):
        id=pk
        stu=Employee.objects.get(pk=id)
        serializer=EmployeeSerializer(stu,data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Compltete Data Updated'})
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def partial_update(self, request, pk):
        id=pk
        stu=Employee.objects.get(pk=id)
        serializer=EmployeeSerializer(stu,data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Partial Data Updated'})
            return Response(serializer.errors)

    def destroy(self, request, pk):
        id=pk
        stu=Employee.objects.get(pk=id)
        stu.delete()
        return Response({'msg':'Data Deleted'})

7. create router:
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from BlogApp.views import EmployeeViewSet

router = DefaultRouter()

router.register(r'employee', EmployeeViewSet, basename='employee')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
    path('auth/',include('rest_framework.urls',namespace='rest_framework'))             # will provide login/logout functionality
]

<b>NOTE: </br>
1. Along with authentication we must give permission also, then only it'll work.
2. To get login button in DRF itself we should give 3rd line of urls.py
</b>
</pre>

# Django Interview questions with answer
1. Meta class in django?
<pre>
1. Meta class provides the meta data about the class.
2. So meta data meaning is data about data, so here meta data is data about class.
3. Example: in models.py

class Meta:
    verbose_name = "Redable table name"
</pre>

2. render vs render_to_response?
<pre>
Both are used to render HTML template, render is most common in modern django whereas render_to_response is old and it's removed since Django 2.0.
</pre>

3. How django manage many to many relationships ?
It creates a new table of name with underscore of both table, eg for song and user table it'll create a new table with name song_user of user_song, this table store the primary key of both table, song_id and user_id.
<pre>
Step: create two models student and models
class Course(models.Model):
    name = models.CharField(max_length=300)

    def __str__(self):
        return self.name

class Student(models.Model):
    name = models.CharField(max_length=300)
    course = models.ManyToManyField(Course)

    def course_student(self):
        return ','.join([str(p) for p in self.course.all()])


Step 2: If we need to fetch using shell we'll get the object first
st_obj = Student.objects.get(id=1)
st_obj.course.all()
</pre>

4. Django with multiple database, possible? if yes in which case we need it?
<pre>
Yes, You should provide a dictionary of database connections where each key represents a database alias, and the value is a
dictionary containing the database settings like engine, name, user, password, host, port, etc.

Case 1: We need to use different db for developement & production i.e different database for production and developement,
so we can give condition if debug = False, use developement db.
Case 2: We need seperate database for read and write, i.e one is read intensive and another one is write intensive task.
To improve the database speed.

Step 1: In settings.py give comma seperated dict of database.
Step 2: create a router.py file:
# routers.py
class MyAppRouter:
    def db_for_read(self, model, **hints):
        if model._meta.app_label == 'myapp':
            return 'second_db'
        return 'default'

    def db_for_write(self, model, **hints):
        if model._meta.app_label == 'myapp':
            return 'second_db'
        return 'default'
</pre>


5. What are Q objects in django?

6. What is the difference between syncdb and migrate command ?
<pre>
In modern versions of Django, syncdb is deprecated, and you should use migrate for all database schema management tasks.
</pre>

7. What are request middleware and response middleware?
<pre>
Explain the life cycle.
</pre>

8. What do you mean by direct_to_template view in django ?
<pre>
It was used by old django to render template in class based view. New version use TemplateView to render HTML template.
</pre>

9. What are modular class based generic views?
<pre>
Modular class-based generic views, often referred to as "modular CBVs" or "class-based views,"
are a powerful feature in Django that provides a reusable and organized way to handle common web patterns.

EXample:

from django.views.generic import ListView
from .models import MyModel

class MyModelListView(ListView):
    model = MyModel
    template_name = 'myapp/mymodel_list.html'
    context_object_name = 'mymodels'

</pre>

10. Why is the CSRF token used in Django?
<pre>
In Django, you use {% csrf_token %} template tag to include a CSRF token in your HTML forms.
The purpose of this token is to ensure that the request is coming from a genuine user and not from an
attacker attempting a CSRF (Cross-Site Request Forgery) attack.

We use: {% csrf_token %}
</pre>

11. What does the collectstatic command do ?
<pre>
It traverses through all installed apps, including your project's STATICFILES_DIRS and static directories within app folders,
and copies their static files into a designated directory, usually defined by the STATIC_ROOT setting in your project's settings.
</pre>

12. How to change default timezone in django ?
<pre>
in settings.py
TIME_ZONE = 'America/New_York'

python manage.py migrate
</pre>

15. Does django have a loosely coupled architecture ? Explain the same.
<pre>
Django has a loosely coupled architecture. It's designed with modularity and flexibility in mind, allowing its components to be used independently,
and it encourages the creation of reusable, self-contained apps. Django's middleware and pluggable nature enable customization, and it offers a
wide choice of components like databases and templates. This loose coupling makes it adaptable and promotes clean, maintainable code.


In software design, "loosely coupled" refers to a system's components or modules that are designed to interact with each other with minimal dependencies.
</pre>

16. What is the purpose of wsgi.py file in django project ?
<pre>
wsgi acts as a interface between django application and our python project.
We need to tell gunicorn to use wsgi in order to server the django application rather than python’s built in http server.
$ gunicorn — bind 0.0.0.0:8000 <your-project>.wsgi:application
</pre>

17. What is the difference between absolute path and relative path ?
<pre>
absolute paths provide a complete reference from the root directory,
while relative paths reference files or directories relative to the current working directory.
</pre>

18. What is list view in django ?
<pre>

</pre>

19. What is template view in django ?
<pre>
In class based view we use template view to render html page.
Example:

from django.views.generic import TemplateView

class MyTemplateView(TemplateView):
    template_name = 'my_template.html'
</pre>

20. What do you mean by pagination of resultset ?
<pre>
Django, as a web framework, provides built-in support for implementing pagination, making it easier for
developers to manage large result sets and create user-friendly interfaces.
Pagination is often used in Django views to display data to users in a structured and manageable way.
</pre>

21. How to access a django settings variable ?
<pre>
from django.conf import settings

debug_mode = settings.DEBUG
database_config = settings.DATABASES
</pre>

22. Why do we have a "__init__.py" file in a django app ?
<pre>
The __init__.py file in the app directory is used to indicate that the directory should be recognized as a Python package.
Without the __init__.py file, Python wouldn't recognize the directory as a package, and you wouldn't be able to import its contents.
</pre>

23. What is the Architecture is following Django?
<pre>
MVT
</pre>

24. Have you heard about Middlewares? what is it? how it’s Important in Django? How to write custom middleware in Django? give one sample( you can use IP filtering for the example)
<pre>
1. In your app:
# ip_filter_middleware.py
class IPFilterMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # Add your IP address filtering logic here
        allowed_ips = ['127.0.0.1', '192.168.1.1']
        client_ip = request.META.get('REMOTE_ADDR')

        if client_ip not in allowed_ips:
            return HttpResponseForbidden("Access denied")

        response = self.get_response(request)
        return response

2. In your Django project's settings, add the fully-qualified path to your middleware class to the MIDDLEWARE setting.
Make sure it's placed before other middlewares that you want to apply after it.
# settings.py
MIDDLEWARE = [
    # ...
    'yourapp.middleware.IPFilterMiddleware',  # Add the path to your middleware class here
    # ...
]
</pre>

25. Explain the alredy provided django middleware, and how it works in request and response time?
<pre>
1. Django provodes the following default middleware.
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

2. The middleware will run in the order and it is important, so that first SecurityMiddleware should run before SessionMiddleware and so on.
3. At time of request the first middleware which hit is SecurityMiddleware and last in XFrameOptionsMiddleware in the given order.
4. At time of response the middleware follow the reverse order.
so first middleware which hit is XFrameOptionsMiddleware and last in XFrameOptionsMiddleware in the given order.
5. Django checks the middleware at the time of request and response too.
6. So the above custom middleware which is ip filtering, should be given after security middleware.
</pre>

25. What is the new version of Django? what all the differences? (don’t forget to say about the latest version support only python 3. x version.
<pre>
Latest django support python 3.x only.
</pre>

26. What is Backend in Django? Can we write own backend? when?
1. In Django, a "backend" typically refers to the authentication backend, which is a part of Django's authentication system. Authentication backends in Django are responsible for verifying the user's credentials (username and password), as well as providing the user's information after successful authentication.</br>
2. Django provides a default authentication backend that works for most cases, and it's usually sufficient for typical web applications. This backend uses the username and password fields in the Django user model for authentication.</br>
3. However, you can write your custom authentication backend when you have specific authentication requirements that go beyond the default behavior. Common scenarios where you might want to write your own authentication backend include:</br>
4. Integrating with an external authentication system, like LDAP, OAuth, or a single sign-on (SSO) service.</br>


28. What is Decorator? How to write custom decorator in Django ? Give one sample
<pre>
Simple decorator example:   below example UnauthorizerException is optional

class UnauthorizerException(Exception):
    def __init__(self, message = "You are not authorize to view."):
        self.message = message
        super().__init__(self.message)

def superuser_permission(render_func):
    def wrapper(request, *args, **kwargs):
        if request == "auth_user":          # in django replace request.user.is_superuser()
            return render_func(request, *args, **kwargs)
        else:
            raise UnauthorizerException # Return an PermissionError response indicating unauthorized access.
    return wrapper


@superuser_permission
def render_func(request):
    # If auth user can view all users
    # users = User.objects.all()
    return f'<h1>This is a protected view</h1>{request}'

# Simulate a request (you should use Django's test client or an actual request object).
request = "auth_user"  # Simulate an authorized request.
res = render_func(request)
print(res)
</pre>

29. How is the session Handling In Django? What are other session storage provided by django?
<pre>
The default session engine is the "database-backed sessions" engine.
This means that by default, session data is stored in a database table called django_session.

Other session methods are:
To include other session in django we need to give SESSION_ENGINE:
SESSION_ENGINE = 'django.contrib.sessions.backends.cache'

1. Database Storage: Session data is stored in a database table named django_session. This one is default.
2. Cached sessions: Use memcached or redis.
3. File-Based Sessions: session data is stored as files on the server's file system.
4. Cookie-Based Sessions: stores session data directly in encrypted cookies sent to the client's browser.
5. Cache Database Sessions: Session data is first stored in a cache, and it falls back to the database if the cache is unavailable.
This approach can be a good compromise between speed and persistence.
</pre>

30. What is the Usage of ALLOWED_HOST in Django project settings?
<pre>
The hosts which are allowed, if we have frontend host port, we need to give here.
If we give ALLOWED_HOST = "*" everyone is allowed, dont use in production.
</pre>

31. How are the static(CSS, JS, Images)files rendering in Django?
<pre>
Step1 : give the static path, join with base dir. STATIC_DIR=os.path.join(BASE_DIR,'static')
Step2: {% load static %}
</pre>

32. Difference between extend and include in jinja template? And What do you mean by template inheritence in django ?
32. What is Template(HTML) Inheritance (inherit base.html to index.html using extends)?
<pre>
Join the template dir
'DIRS': os.path.join(BASE_DIR,'templates')

extend example:

base_template.html
{% block title %}Default Title{% endblock %}

child.html
{% extends "base_template.html" %}
{% block title %}Child Title{% endblock %}

when you use {% extends %} to extend a base template, the entire content of the base template is
inherited into the child template, except for the content enclosed within the {% block %} tags in the child template.

include example
header.html
footer.html

index.html
{% include "header.html" %}
{% include "footer.html" %}

The content of header and footer will be included in the index.html file, wherever we've given
like if we give it inside body tag both will be visble inside body.
</pre>

33. What is Template Tags? What is the use? How to write custom tags?
<pre>
1. Whatever in inside this {% tag %} is template tag, eg: {% for %}, {% if %} etc.
2. Creating custom template-tag for reverse of a string.

step 1: create a new module # myapp/templatetags/my_tags.py
from django import template

register = template.Library()

@register.filter
def reverse_string(value):
    return value[::-1]

step 2: {% load my_tags %}
step 3: {{ my_string|reverse_string }}
</pre>

34. What is Queryset? what is the instance? (all, filter and get)? Difference between filter and get in queryset?
<pre>
Queryset: to query the data, it's same like database query. Protect from sql injection attack etc.
get(): used to fetch single record, if it found more than one record than it'll raise exceptions.
filter(): use when filtering multiple records. will return multiple records.
</pre>

35. Do you know about model Manager in Django? Can we write custom manager? How and why?
<pre>
1. In Django, a manager is an interface through which database query operations are provided to a Django model.
Managers are used to encapsulate the logic for interacting with the database and performing queries on the model.
2. DRY (Don't Repeat Yourself) principles: By reusing common query logic across your application, you reduce duplication in your code.
3. Example:

from django.db import models

class CustomManager(models.Manager):
    def published_books(self):
        return self.filter(published=True)

    def books_in_offer(self):
        return self.filter(in_offer=True)

class Book(models.Model):
    title = models.CharField(max_length=100)
    author = models.CharField(max_length=100)
    published = models.BooleanField(default=False)

    # Attach the custom manager to the model
    custom_objects = CustomManager()


4. Accessing the manager object:
published_books = Book.custom_objects.published_books()
books_in_offer = Book.custom_objects.books_in_offer()
5. Django already have one manager which is objects, eg: Student.objects.filter().
<b>
Note: When we use custom manager `objects` will no loner be available, we have to use custom_objects instead of objects:
Eg: Student.custom_objects.all()
</b>
</pre>

36. Have you heard about model permissions? why it's using?
<pre>
class MyModel(models.Model):
    name = models.CharField(max_length=100)
    objects = MyModelManager()

    class Meta:
        permissions = [
            ("can_view_mymodel", "Can view MyModel"),
            ("can_change_mymodel", "Can change MyModel"),
            ("can_delete_mymodel", "Can delete MyModel"),
        ]
</pre>

37. Consider you have got a project, How will you create models?
<pre>

</pre>

38. What is __init__ method do in models?
<pre>

</pre>

39. What is Inheritance in Models ? Give one sample? </br>
39. Difference between Abstract base class inheritance, Multi-table inheritance, Proxy Model?
<pre>
1. Abstract Base Class
2. Proxy Model
3. Multi-Table Inheritance

SEE ABOVE
</pre>

40. What’s the difference between Django OneToOneField and ForeignKey?
<pre>
OneToOneField example:
class Student(models.Model):
    name = models.CharField(max_length=300)

class Aadhar(models.Model):
    aadhar_num = models.IntegerField()
    student = models.OneToOneField(Student, on_delete=models.CASCADE)
</pre>

41. What’s the difference between OneToMany, ForeignKey and ManyToMany field?
<pre>
OneToMany: one parent can have only one child.
ForeignKey: one parent can have many child.
ManyToMany: one parent can have many childs, many childs can have many parents.
</pre>

42. How to insert a data into the M2M field in the models(field.add.()), also update and delete.
<pre>
# ADDING
class Student(models.Model):
    name = models.CharField(max_length=300)
    course = models.ManyToManyField(Course)

In above models, to insert data in course model, we need to first insert data in student object and save
$ st = Student(pk=1)
$ st.save()
$ st.course.add(Course.objects.get(id=1), Course.objects.get(id=2))     # we cannot give filter here cuz it expect an pk.


# UPDATING
$ st = Student.objects.get(pk=1)
$ new_course = [Course.objects.get(id=2)]
$ st.course.set(new_course)


# DELETING
We can use `remove`, `clear`

remove: to remove few objects
$ st = Student.objects.get(pk=1)
$ remove_course = [Course.objects.get(id=1), Course.objects.get(id=2)]
$ st.course.remove(remove_course)


clear: to remove entire course
$ st = Student.objects.get(pk=1)
$ st.course.clear()


<b>Remember all methods add, set, remove takes list of values. And if we want to use filter we need to use astrick(*).</b>
# Using get 
$ obj = Student.objects.get(pk=1)
$ courses = [Course.objects.get(pk=2), Course.objects.get(pk=3))
$ obj.course.add(courses)

# Using filter
$ obj = Student.objects.get(pk=1)
$ courses = Course.objects.filter(name__in=["python", "java"])
$ obj.course.add(*courses)		# compulsory use artrick
</pre>

44. How to do CRUD operations in Queryset?
<pre>
# Adding
$ Student(name="Fareen", age="26").save()

# Update
Using filter:
$ Student.objects.filter(pk=1).update(name="Python_update")

Using get:
obj = Student.objects.get(pk=1)
obj.name = "new name"
obj.save()

# Delete
$ Student(name="Fareen", age="26").delete()
</pre>

46. CRUD in OneToOneField?
<pre>
This is same as adding the object liek above, difference is we just need to give queryset instead of object, at place where it has foreign key

# ADDING
$ Aadhar(aadhar_num=123, student_name=Student.custom_objects.get(pk=3)).save()

In above instead of providing name, we give student_name queryset. Rest all is same like above
</pre>

47. CRUD in ForeignKey?
<pre>
Same as above, just give foreign key queryset instead of name:

We need to create the Book(parent) object first and then add child

Adding:
# Creating a Book instance
book = Book(title="Python with Django", version="1.1")
book.save()  # Save the book instance

# Retrieving authors with primary keys 2 and 4
author_1 = Author.objects.get(pk=2)
author_2 = Author.objects.get(pk=4)

# Adding authors to the Book instance
book.authors.add(author_1, author_2)
</pre>

43. Difference between User, AbstractUser and AbstractBaseUser models in Django?
<pre>
<b>User</b>
Adding user have one function, which is create_user:

from django.contrib.auth.models import User
new_user = User.objects.create_user(username='john_doe', password='password123', email='john@example.com')


<b>Extending the Built-In User Model:</b>
from django.contrib.auth.models import User

class CustomUserModel(User):
    user = models.OneToOneField(User, on_delete=models.CASCADE)
    age = models.IntegerField()

Note: above will create an entirely new model `CustomUserModel` where we have to give foreign key of user


Use below option to entirely create custom model for user
<b>AbstractUser</b>
Subclassing AbstractUser to add custom fields and methods to the default user model.

Example:
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    age = models.IntegerField()


<b>AbstractBaseUser</b>
Creating a completely custom user model using AbstractBaseUser to define your own authentication system.
from django.contrib.auth.models import AbstractBaseUser, BaseUserManager

class CustomUserManager(BaseUserManager):
    def create_user(self, username, email, password=None):
        if not email:
            raise ValueError('The Email field must be set')
        email = self.normalize_email(email)
        user = self.model(username=username, email=email)
        user.set_password(password)
        user.save(using=self._db)
        return user

class CustomUser(AbstractBaseUser):
    username = models.CharField(unique=True, max_length=30)
    email = models.EmailField(unique=True)
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    # Add your custom fields

    objects = CustomUserManager()

    USERNAME_FIELD = 'email'
    REQUIRED_FIELDS = ['username']

    def __str__(self):
        return self.email
</pre>

44. How to inherit User model in django?
<pre>
See above 1st option
</pre>

45. What is the difference between Function based and Class-based views?
<pre>
Simple views can be efficiently implemented with FBVs, while more complex views, especially those that follow
common patterns like CRUD operations, can benefit from the structure and reusability that CBVs provide.
 In many cases, Django's Class-Based Views can lead to more maintainable and DRY (Don't Repeat Yourself) code.
</pre>

46. What is serialization & deserialization?
<pre>
See above
</pre>

47. What’s the difference between select_related and prefetch_related in Django ORM?
<pre>
See above
</pre>

48. What all basic request methods? Difference between POST & PUT
<pre>
GET: Typically used to retrieve a list of records or a single record.
POST: Used for creating new data records.
PUT: Used to replace the complete data of a resource.
PATCH: Used to partially update a resource, typically specific fields.
DELETE: Used to remove data records.
</pre>

49. Difference between Django’s annotate and aggregate methods?
<pre>
Annotate is used for `group_by` and aggregate is used for `aggregrate methods` like sum, min, max, count etc

Example:
`Annotate`:
obj = Book.objects.values('author__name').annotate(count=Count('author'))
print(obj.query)

`SELECT "BlogApp_author"."name", COUNT("BlogApp_book"."author_id") AS "count"
FROM "BlogApp_book"
INNER JOIN "BlogApp_author"
ON ("BlogApp_book"."author_id" = "BlogApp_author"."id")
GROUP BY "BlogApp_author"."name"`

whether we give in values will become group by:
obj=Book.objects.values('author', 'version').annotate(count=Count('author'))
print(obj.query)

`SELECT "BlogApp_book"."author_id", "BlogApp_book"."version", COUNT("BlogApp_book"."author_id") AS "count"
FROM "BlogApp_book"
GROUP BY "BlogApp_book"."author_id", "BlogApp_book"."version"`


Example:
`Aggregrate`:

from django.db.models import Min, Max, Count, Sum, Avg, StdDev, Variance, ExpressionWrapper, F, FloatField
obj = Price.objects.aggregate(price_min = Min('price'))

<b>CUSTOM AGGREGATE</b>
Do the addition of two database fields

Books.objects.aggregate(
    add = ExpressionWrapper(
        F('actual_price') + F('discount_price')
    ), 
    output_field = FloatField()
)

</pre>

50. How to filter multiple fields in a filter?
<pre>
Using Q objects.
Example: see above
</pre>

51. How the Ajax request is working Django ? is it secured?
<pre>
You can make it secure, using many practices one of them is including csrf token.
</pre>

52. What is CSRF? How it’s preventing in Django?
<pre>
See above
</pre>

53. What Are messages in Django? How it’s helping Django application development
<pre>

</pre>

54. What are signals in Django? what all the methods available?
<pre>
Signals are used to add extra functionality when an object is saved before of after

Example of pre and post save: To create a signal:
1. In apps.py add ready function:

from django.apps import AppConfig

class BlogappConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'BlogApp'

    def ready(self):
        import BlogApp.signals

2. Create signals.py in the app:
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import Book

@receiver(post_save, sender=Book)
def book_after_save(sender, instance, **kwargs):
    if not instance.pk:
        print("CREATED")
        instance.version = instance.version * 2
    else:
        print("UPDATED")
        instance.version = instance.version * 2

    # Disconnect the signal temporarily to avoid recursion
    post_save.disconnect(book_after_save, sender=Book)
    instance.save()
    # Reconnect the signal
    post_save.connect(book_after_save, sender=Book)

Same code of pre_save() just replace everywhere with pre_save

</pre>

55. What is the Django Admin interface? How to override its admin panel. What is Jazzmine?
<pre>

</pre>

56. What is the context in Django?
<pre>
In django we pass context from views to templatea as a dict.
</pre>

57. What's the use of a session framework?
<pre>
In Django, a web framework for Python, the session framework is built-in and provides a convenient way to use sessions in your web applications.
You can store and retrieve data in sessions using the request.session dictionary in views. This is particularly useful for tasks like
user authentication and maintaining a user's shopping cart throughout their visit to an e-commerce site.
</pre>

58. What is difference in makemigrations and migrate, what are the default tables are created when we run migrate?
<pre>
what are the default tables are created when we run migrate?
Session, Auth, Contenttype, Logentry, Migration table are get created
</pre>

59. What are Django.shortcuts.render functions?
<pre>

In Django, render is a function provided by the django.shortcuts module. It is commonly used in view
functions to simplify the process of rendering an HTML template and returning an HttpResponse object.
The render function takes care of loading a template, rendering it with a given context
(a dictionary containing data to be displayed in the template), and returning an HttpResponse object.

from django.shortcuts import render

def myview(request):
    return render("template.html", context={})
</pre>

60. What does a URLs-config file contain?
<pre>
For class based view:
path("home/", view.home, name="home")

The name parameter provides a way to reference these URLs in templates and views using the {% url %} template tag and the reverse() function.

For class based view:
path("home/", HomeView.as_view(), name="home")
</pre>

61. Does Django support multiple-column primary keys? If not what we can use to have that functionality?
61. Write a multi-column index?
<pre>
No, django does not support multiple-column primary keys, instead we use unique_together for this requirement.

class Person(models.Model):
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)

    class Meta:
        unique_together = ('first_name', 'last_name')

<b>Write a multi-column index</b>
Accept list of tuple

class Person(models.Model):
    first_name = models.CharField(max_length=30, db_index=True)
    last_name = models.CharField(max_length=30, db_index=True)

    class Meta:
        index_together = [
            ('first_name', 'last_name'),
        ]
</pre>

62. How can you see raw SQL queries running in Django?
<pre>
Using query

$ b = Book.objects.all()
$ print(b.query)
</pre>

63. List several caching strategies supported by Django.
<pre>
Django support both database & cache like redis and memcached caching strategies.
</pre>

64. What do you use django.test.Client class for?
<pre>

</pre>

65. What is mixin?<br>
Just like decorators are used for extending the functionality of a function, we can use mxin to extend the functionality of a class. 
<pre>
Its like simple inheritance:

class LoggingMixin:
    def log(self, message):
        print(f"Log: {message}")

class MyClass(LoggingMixin):
    def some_method(self):
        self.log("This is a log message.")

obj = MyClass()
obj.some_method()

Django Exaple:
from django.contrib.auth.mixins import LoginRequiredMixin

class MyProtectedView(LoginRequiredMixin, View):
    # Your view logic here

</pre>

66. What is Django Field Class?
66. Describe Django Field Class types? 
<pre>
CharField, DecimalField, TextField, DateField, and BooleanField are all field class.
</pre>

67. Why is permanent redirection not a good option?
<pre>

</pre>

68. How to combine multiple QuerySets in a View? Example: using chain.Mention the ways used for the customization of the functionality of the Django admin interface.
<pre>
To combine multiple QuerySets in a Django view, you can use the chain function.
Example:

q1 = ook.objects.all()
q2 = Author.objects.all()
q3 = list(chain(q1, q2))

O/p:
`[<Book: Cloud & devops>, <Book: s3>, <Book: ecs>, <Book: data analyst>, <Book: data science>,
<Book: rds>, <Book: Python with django>, <Book: gitt>, <Book: JAVA>,
<Author: Fareen>, <Author: Anamika>]`
</pre>

69. Explain Q objects in Django ORM.
<pre>

</pre>

70. What are Django exceptions?
<pre>
Example: ObjectDoesNotExist, IntegrityError, Http404 etc.
Raising a custom exception is same as python,
We can also create custom exception and raise, also we can handle the raised exception.


def custom_exception_handler(request):
    try:
        # Code that might raise the custom exception
        if some_condition:
            raise MyCustomException("This is a custom exception message.")
    except MyCustomException as e:
        # Handle the custom exception
        return HttpResponse(f"Custom Exception: {e}", status=400)

Now whenever MyCustomException is raise it'll get handled and return HttpResponse with status code 400
</pre>

71. Describe the inheritance styles in Django?
<pre>
Give example of model class inheritence.
</pre>

72. Give a brief about the settings.py file.
<pre>
We can give global settings here, it has allowed hosts, middleware, timezone info, static,
template path info, broker info if we use celery.
</pre>

73. What is the difference between CharField and TextField in Django?
<pre>
A CharField is used to store a relatively short, fixed-length string.
A TextField is used for storing longer, unstructured text data.
</pre>

75. What is the usage of "django-admin" and "manage.py"?
<pre>
Both are command-line utility provided by django.
In django, django-admin is used for administrative related tasks like creating project.
manage.py is a command-line tool for various management tasks in Django.
</pre>

76. What are Django cookies?
<pre>

</pre>

77. Give a brief about Django Template?
<pre>

</pre>

78. When to use iterators in Django ORM?
<pre>
1. The data in lazy load in, This lazy loading behavior is an important aspect of Django's ORM, allowing you to work with large datasets without consuming excessive memory.
2. When we do
queryset = MyModel.objects.all()  # No data is fetched yet,

for item in queryset:
    # Data is fetched from the database in batches as you iterate
    # this is lazy loading.

specific_item = queryset.get(pk=1)  # Data is fetched for the specific item with primary key 1

3. It only loads data when we use for loop or we do  .get(), .filter(), .count(), and so on.

<b>When to use iterator?</b>
When we do
queryset = MyModel.objects.all()  # No data is fetched yet,

for item in queryset:
    # entire data will load into the memory

Instead of loading the big large data into memory itself use iterator.
Iterator breaks the data into chunks, it fetches a small number of rows at a time
(controlled by the chunk_size option, which defaults to 200) from the database and yields them one at a time.

for item in queryset.iterator():
    # will load chunk_size data

<b>Whenever we do just the ORM it does not hit the database, until we manually do for loop or filter:</b>
bk_obj = Book.objects.select_related('title', 'author__name')
bk_obj = Book.objects.all()

Both query will hit the database when data is needed, so that is why when we 
use iterator in for loop it hit the database at that time and get the data into chunks.
</pre>

79. How does Django process a request?
<pre>
Explain request, response.
</pre>

80. Is Django too monolithic? Explain this statement.
<pre>

</pre>

81. What is the use of forms in Django?
<pre>

</pre>

82. Can you explain how to add View functions to the urls.py file?
<pre>
see above
</pre>

83. Explain Django Security.?
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

84. What are Django generic views?
<pre>

</pre>

85. What is the correct way to make a variable available to all your templates?
<pre>

</pre>

86. What is a QuerySet? In the given model write queryset for following?
<pre>
What is a QuerySet: See above

In the given model write queryset for following, There is Book & Author database table, Find:
Consider thr bewlo model:
class Author(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
    bio = models.TextField()

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    title = models.CharField(max_length=200, default="NA")
    version = models.CharField(max_length=13)
    price = models.CharField(max_length=13)

1. Get all the Books count by author.
2. Get all the Books count by author and verision. 
Like Fareen has written 5 books of java of version 1, and 2 books of java with version3 & 3 books of python with version 1 and so on.
3. Fetch all the records of book along with there author, from Book table.
4. Fetch all the record of book along with there author from Author table.
5. Get a single record of Book.
6. Get a multiple record of Book.
7. Get the record of Book which is not written by Fareen and and price is less than 500, using exclude as well as Q.
8. Get the maximum price of the book which is written by Fareen.
</pre>

87. Are Django signals asynchronous?
<pre>
Django signals themselves are synchronous and execute in the same thread as the emitting code.
</pre>

88. Explain different status codes?
<pre>
200 (OK): The request was successful.
201 (Created)

400 (Bad Request): The request could not be understood or was missing required parameters.
401 (Unauthorized)
403 (Forbidden)
404 (Not Found)

500 (Internal Server Error): A generic error message indicating a server problem.
502 (Bad Gateway): The server, while acting as a gateway or proxy, received an invalid response from the upstream server.
503 (Service Unavailable): The server is currently unable to handle the request, typically due to overloading or maintenance.
504 (Gateway Timeout): The server, while acting as a gateway or proxy, did not receive a timely response from the upstream server.
</pre>

89. Authentication & Authorization in Django?
<pre>
Django provides build-in Authentication, along with Authentication it also provide authorization like users, groups & permissions etc
</pre>


90. What are the files gets created when we create new project and new app?
<pre>
project: init.py, settings.py, asgi.py, wsgi.py, urls.py
app: views.py, models.py, init.py, tests.py, admin.py, apps.py

`__init__.py`: In Django, the __init__.py file is a special file that indicates to Python that the directory
containing it should be considered a Python package or module.

asgi.py:
The asgi.py file in Django is used for running Django applications with ASGI (Asynchronous Server Gateway Interface) servers.
ASGI is a specification for asynchronous web servers and frameworks, which allows Django to handle asynchronous operations,
such as handling WebSocket connections, long polling, and other non-blocking tasks.

See old directory for answer.
</pre>

92. How to make query fast, give analogy between all(), selected_related() and iterator().
<pre>
all():
Book.objects.all("author__name")                : join again and again for every data
for i in qs:
    # running join for data 1
    # running join for data 2

selected_related()
Book.objects.selected_related("author__name")   : join once and load data into memory
for i in qs:
    # data is loaded into the memory
    # no load onto the database
    # not fast for big data, to make it fast use Iterator chunk

Iterator():
for i in qs.iterator():
    # load the data in chunk of 200
    # make result faster
    # this is like a typical electric generator, store electricity and we can generat electricity when needed.


Same for prefetch_related also:
Author.objects.all("book__title")
Author.objects.prefetch_related("book__title")
</pre>
	
93. What are the parameters of models?
<pre>
def name_validation(name):
    if name.Startswith("s"):
        raise InvalidNameException

name = models.CharField(
    max_length=100, blank=True,
    null=True,
    default='',
    editable=True,
    choices=NAME_CHOICES,
    verbose_name='Full Name',
    help_text='Enter your full name',
    unique=True,
    db_index=True
    db_column=db_column_name,
    validators=[name_validation]           # accept list of validations
    )
</pre>

94. Difference between (blank = True) and (null = True), (blank = True, null = True).
<pre>
blank = True	# on ui accept blank value, but nothing given for null so blank space will be saved in db.
null = True	# in db "NULL" will be saved in db .
</pre>

95. What is migration management command? </br>
Difference between makemigration and migrate? </br>
How to undo a migration?</br>
How to show migrations file?</br>
<pre>
See above
</pre>

97. Different types of join in ORM?
<pre>
For sql join see below SQL joins.

1. By default in django ORM does `INNER JOIN`, i.e join of column which matches like `emp.id = dept.id`.
2. To do `CROSS JOIN`,:
a = Book.objects.all()
b = Author.objects.all()

# avoid using
cross_res = [i*j for i in a for j in b]	

# must use for cross join	
from itertools import product
cross_join_result = list(product(books, authors))

4. To do `LEFT JOIN`, `OUTER JOIN` or Other join which ORM does not support
then we can use Pandas for that or use the raw sql query:

import pandas as pd
from BlogApp.models import Book, Author

# Get your data from Django models
books = Book.objects.values()
authors = Author.objects.values()

# Create DataFrames from the QuerySets
df_books = pd.DataFrame.from_records(books)
df_authors = pd.DataFrame.from_records(authors)

# Perform an outer join using Pandas
result = pd.merge(df_books, df_authors, left_on='author_id', right_on='id', how='outer')

# If you want to convert the result back to a QuerySet, you can do it manually
result_dict = result.to_dict(orient='records')

# Now result_dict contains a list of dictionaries that you can use like a QuerySet
</pre>

98. Give another use case of annotate besides group_by?
<pre>
See above.
</pre>

# DRF Interview questions with answer
1. Difference between put and patch?
2. What is serializer and deserializer?
3. What is cors?
4. Authentication in DRF?
5. Authorization/Permission in DRF?
6. What is JWT, What is access and refresh token?
7. Routers in DRF?
8. Difference in APIView & ViewSet?
9. Difference in GenericAPIViewSet & GenericViewSet?


# SQL
# Joins
<pre>
left: 
right:
cross: it'll be table A x table B, and the result will be records of table A x table B
inner: matching data only. 
outer: matching data from both, its like union
</pre>


# Decorators, kafka, ORM, class & methods, DRF


# Extra - will change afterwords
1. ExpressionWrapper:<br>
It allows you to perform complex operations on fields at the database level, which can be more efficient than fetching data into Python code and then performing the operations.<br>
<pre>

</pre>

2. Between operator in django orm.
<pre>
from datetime import date

# Define your date range
start_date = date(2023, 1, 1)
end_date = date(2023, 12, 31)

# Query the database
result_queryset = YourModel.objects.filter(date_field__range=(start_date, end_date))
</pre>
