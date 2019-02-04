###Initial project set up
go to home directory
```cd ~ ```
make a project directory
```mkdir celery-startn ```
go to project directory
```cd celery-start ```
create a virtual environment with python3 named venv
``` virtualenv -p python3 venv ```
activate virtual environment
``` source celery-start/venv/bin/activate ```
install django in virtual environment
``` pip install django ```
create django project named mysite
``` django-admin startproject mysite ```
create django app named polls
``` python manage.py startapp polls ```
this is for db migration default sqlite3
``` python manage.py migrate ```
###Now we need to add polls url to mysite
Now add bellow in mysite.urls.py
```python 
	urlpatterns = [
	   path('polls/', include('polls.urls')), #this is needed to add
	   path('admin/', admin.site.urls),
	]
```
Then add a polls.urls.py file and add bellow things:
```python
	from django.urls import path
	from . import views
	urlpatterns = [
	   path('', views.index, name='index'),
	]
```
This is for run the application.
```console
python manage.py runserver ```
Then it looks ok. A server will run at 0.0.0.0:8000
Go to http://127.0.0.1:8000/polls/
You will get bellow things in a web page:
- Hello, world. You're at the polls index.

### Now we need to install rabbitmq as a message broker as following link:
[install rabbitmq installation guideline](https://www.rabbitmq.com/install-debian.html)

####Now let's integrate celery with this project.
Create a mysite.celery.py file and paste this code:
```python
	from __future__ import absolute_import, unicode_literals
	import os
	from celery import Celery

	# set the default Django settings module for the 'celery' program.
	os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'mysite.settings')

	app = Celery('celery_app')

	# Using a string here means the worker don't have to serialize
	# the configuration object to child processes.
	# - namespace='CELERY' means all celery-related configuration keys
	#   should have a `CELERY_` prefix.
	app.config_from_object('django.conf:settings', namespace='CELERY')

	# Load task modules from all registered Django app configs.
	app.autodiscover_tasks()

	@app.task(bind=True)
	def debug_task(self):
	   print('Request: {0!r}'.format(self.request))
```
Add followings in setting.py
```python
	CELERY_BROKER_URL = "amqp://guest:guest@localhost:5672"
	CELERY_RESULT_BACKEND = 'django-db'
	CELERY_ACCEPT_CONTENT = ['application/json']
	CELERY_RESULT_SERIALIZER = 'json'
	CELERY_TASK_SERIALIZER = 'json'
	CELERY_TASK_RESULT_EXPIRES = None
	CELERY_TASK_TRACK_STARTED = True
	CELERY_TASK_TIME_LIMIT = 86400
	CELERY_MAX_RETRIES = 0
	CELERY_RESULT_PERSISTENT = True
	CELERY_TASK_CREATE_MISSING_QUEUES = True
	CELERY_TASK_DEFAULT_QUEUE = 'mysite'
	CELERYD_PREFETCH_MULTIPLIER = 1
```
Add bellow in installed_apps in settings.py
```python
	'polls.apps.PollsConfig',
	'django_celery_results',
```
Then run migrations for celery_result_back_end:
```console
python manage.py migrate django_celery_results
```
Now you can see related table in sqlite db installing like  sqlitebrowser similar tool
```console
sudo apt-get install sqlitebrowser
```

