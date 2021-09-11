
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
echo "Django~=2.2.4" > requirements.txt
pip install -r requirements.txt
django-admin startproject mysite .

In mysite/settings.py
  - set TIME_ZONE to America/Chicago
  - Add the following to the end
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static')
  - Change ALLOWED_HOSTS as follows
ALLOWED_HOSTS = ['127.0.0.1', '.pythonanywhere.com']

Generate a secret key

echo SECRET=`python -c 'from django.core.management.utils import get_random_secret_key; \
      print(get_random_secret_key())'` > .env

Create the database
python manage.py migrate

In mysite/settings.py
  - modify SECRET_KEY to read
SECRET_KEY = os.getenv('SECRET')

In your shell, define SECRET
export $(xargs <.env)

Start the web server
python manage.py runserver

Create the blog app
python manage.py startapp blog

Inform Django to use the site - edit mysite/settings.py to INSTALLED_APPS block

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog.apps.BlogConfig',
]

Define the model in the database - blog/models.py

from django.conf import settings
from django.db import models
from django.utils import timezone


class Post(models.Model):
    author = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    text = models.TextField()
    created_date = models.DateTimeField(default=timezone.now)
    published_date = models.DateTimeField(blank=True, null=True)

    def publish(self):
        self.published_date = timezone.now()
        self.save()

    def __str__(self):
        return self.title


Add model to database

python manage.py makemigrations blog

Apply changes to database

python manage.py migrate blog


Edit blog/admin.py

from django.contrib import admin
from .models import Post

admin.site.register(Post)


Create a superuser

python manage.py createsuperuser

