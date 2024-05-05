# Start-Django-Postgres-Docker-Project
GIT INIT

After creating the GIT project, this should be the folder structure:
```bash
Project Root
│   .gitattributes
│   .gitignore
│   LICENCE
│   README.md
│   main.py
```
 
STEP BY STEP TO CREATE THE DJANDO PROJECT

```bash
# Create a python virtual environment
$ python3 -m venv env

# Activate virutal environment
source env/bin/activate

# update pip before installing packages
(env) $ pip install --upgrade pip

# install packages
(env) $ pip install django gunicorn psycopg2-binary python-dotenv

# Save packages and versions in a file
(env) $ pip freeze > requirements.txt
```

Your `requirements.txt` files shoudl look like this:

```bash
# requirements.txt

asgiref==3.8.1
Django==5.0.4
gunicorn==22.0.0
packaging==24.0
psycopg2-binary==2.9.9
sqlparse==0.5.0

```

At this point, we can start a Django project in the current folder using `.` at the end:
```bash
(env) $ django-admin startproject playground .
```
Folder structure should now looks like:
```bash
Project Root
├── env
├── playground
    ├── __init__.py
    ├── asgi.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
│   .gitattributes
│   .gitignore
│   LICENCE
│   manage.py
│   README.md
│   requirements.txt
```
Before launching the first time, we need to config the Django project to run locally, lets open `/playground/settings.py` and make some changes:
```bash
from pathlib import Path
# add this section
from dotenv import load_dotenv
import os

load_dotenv()
```
There are some variables in the `settings.py` file we need to secretly save, change the ones below for an environment variables (we will set in a moment):
```bash
# SECURITY WARNING: keep the secret key used in production secret!
SECRET_KEY = os.environ.get("DJANGO_SETTINGS_SECRET_KEY")

# SECURITY WARNING: don't run with debug turned on in production!
DEBUG = bool(os.environ.get("DJANGO_SETTINGS_DEBUG"))

ALLOWED_HOSTS = os.environ.get("DJANGO_SETTINGS_ALLOWED_HOSTS").split(" ")

. . .

STATIC_URL = 'static/'

# Default primary key field type
# https://docs.djangoproject.com/en/4.0/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
```
We need some environment variables now, lets create a new file using:
```bash
(env) $ touch .env
```
And open the file, this way we can add the next variables:
```bash
# Django settings
DJANGO_SETTINGS_SECRET_KEY=Password1234!
DJANGO_SETTINGS_DEBUG=True
DJANGO_SETTINGS_ALLOWED_HOSTS=0.0.0.0 localhost 127.0.0.1
```
Let's complete the basic configuration running the databsae structure and creating a super user:
```bash
(env) $ python3 manage.py makemigrations
(env) $ python3 manage.py migrate

(env) $ python3 manage.py createsuperuser
# You'll need to choose a username, provide email and confirm the password

(env) $ python3 manage.py collectstatic
```
It's time to run the server the first time:
```bash
(env) $ python3 manage.py runserver 0.0.0.0:8000
```
In your browser, visit `http://localhost:8000`