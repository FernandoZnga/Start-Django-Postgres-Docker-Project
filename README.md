# Start-Django-Postgres-Docker-Project
GIT INIT

After creating the GIT project, this should be the folder structure:
```bash
ProjectRoot
│   .gitattributes
│   .gitignore
│   LICENCE
└── README.md
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

Your `requirements.txt` files should look like this:

```bash
# requirements.txt

Django==5.0.4
gunicorn==22.0.0
psycopg2-binary==2.9.9
sqlparse==0.5.0
python-dotenv==1.0.1

```

At this point, we can start a Django project in the current folder using `.` at the end:
```bash
(env) $ django-admin startproject playground .
```
Folder structure should now looks like:
```bash
ProjectRoot
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
└── requirements.txt
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
It's time to deactivate the virtual environment and run the local server for the first time:
```bash
# this will deactivate the virutal environment
(env) $ deactivate
# now run the local server
$ python3 manage.py runserver 0.0.0.0:8000

# output
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 18 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
May 11, 2024 - 17:22:24
Django version 5.0.4, using settings 'playground.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.

```
In your browser, visit `http://localhost:8000`
To stop the local server `ctrl + c`
USING DOCKER
Maker sure you have Docker and Docker Desktop installed locally, if you have it already, you need to run the next piece of code and follow instructions:
```bash
$ docker init
```
```bash
# output
Welcome to the Docker Init CLI!

This utility will walk you through creating the following files with sensible defaults for your project:
  - .dockerignore
  - Dockerfile
  - compose.yaml
  - README.Docker.md

 Lets get started!

? What application platform does your project use?  [Use arrows to move, type to filter]
> Python - (detected) suitable for a Python server application
  Go - suitable for a Go server application
  Node - suitable for a Node server application
  Rust - suitable for a Rust server application
  ASP.NET Core - suitable for an ASP.NET Core application
  PHP with Apache - suitable for a PHP web application
  Java - suitable for a Java application that uses Maven and packages as an uber jar
  Other - general purpose starting point for containerizing your application
  Dont see something you need? Let us know!
  Quit
```
Using arrox up and down, select Python (detected) and answer the questions displayed:
```bash
# output
Welcome to the Docker Init CLI!

This utility will walk you through creating the following files with sensible defaults for your project:
  - .dockerignore
  - Dockerfile
  - compose.yaml
  - README.Docker.md

Lets get started!

? What application platform does your project use? Python
? What version of Python do you want to use? (3.11.6)
? What port do you want your app to listen on? (8000)
? What is the command you use to run your app? python3 manage.py runserver 0.0.0.0:8000

CREATED: .dockerignore
CREATED: Dockerfile
CREATED: compose.yaml
CREATED: README.Docker.md

✔ Your Docker files are ready!

Take a moment to review them and tailor them to your application.

When you are ready, start your application by running: docker compose up --build

Your application will be available at http://localhost:8000

Consult README.Docker.md for more information about using the generated files.
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
│   .dockerignore
│   .env
│   .gitattributes
│   .gitignore
│   compose.yaml
│   db.sqlite3
│   Dockerfile
│   LICENCE
│   manage.py
│   README.Docker.md
│   README.md
└── requirements.txt
```
I have my own `Dockerfile` and `compose.yaml` file already tunned and my suggestion is to use this one because is already tested, so let's change the `Dockerfile` with this:
```bash
# syntax=docker/dockerfile:1.4

ARG PYTHON_VERSION=3.11.6
FROM python:${PYTHON_VERSION}-alpine as base

# Prevents Python from writing pyc files.
ENV PYTHONDONTWRITEBYTECODE=1

# Keeps Python from buffering stdout and stderr to avoid situations where
# the application crashes without emitting any logs due to buffering.
ENV PYTHONUNBUFFERED=1

EXPOSE 8000
WORKDIR /app 
COPY requirements.txt /app
RUN pip3 install -r requirements.txt --no-cache-dir
COPY . /app 
ENTRYPOINT ["python3"] 
CMD ["manage.py", "runserver", "0.0.0.0:8000"]

FROM base as dev-envs
# We use apk instead of apt-get because we are using alpine
RUN <<EOF
apk update
apk add git
EOF

# Create a non-root user
RUN <<EOF
addgroup -S docker
adduser -S --shell /bin/bash --ingroup docker vscode
EOF
# install Docker tools (cli, buildx, compose)
COPY --from=gloursdocker/docker / /
CMD ["manage.py", "runserver", "0.0.0.0:8000"]
```
And `compose.yaml` file:
```bash
services:
  app:
    container_name: app
    build:
      context: .
    ports:
      - 8000:8000
    volumes:
      - .:/app
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
  db:
    container_name: postgres
    image: postgres:latest
    restart: always
    user: postgres
    env_file:
      - .env
    volumes:
      - db-data:/var/lib/postgresql/data
    # expose:
    #   - 5432
    ports:
      - 5432:5432
    healthcheck:
      test: [ "CMD", "pg_isready" ]
      interval: 10s
      timeout: 5s
      retries: 5
volumes:
  db-data:
```

If you notice, the `compose.yaml` file has to services, the one for the Django application and the one for the Database, we are not gonna use `sqlite.db`!

Before building the containers, let's add this env vars into `.env`:
```bash
# Application Database
DB_USER=django_user
DB_PASS=django_password
DB_NAME=django_db
DB_HOST=postgres
DB_PORT=5432
# Testing environment
TEST_DB_NAME=django_test
```

Now it's time to download all resources and create the containers, run this:
```bash
$ docker compose up --build
```
Final output:
```bash
. . . 
 ✔ Network start-django-postgres-docker-project_default  Created                             0.1s
 ✔ Container postgres                                    Created                             0.1s
 ✔ Container app                                         Created                             0.1s
Attaching to app, postgres
postgres  |
postgres  | PostgreSQL Database directory appears to contain a database; Skipping initialization
postgres  |
postgres  | 2024-05-11 23:29:41.227 UTC [1] LOG:  starting PostgreSQL 16.3 (Debian 16.3-1.pgdg120+1) on x86_64-pc-linux-gnu, compiled by gcc (Debian 12.2.0-14) 12.2.0, 64-bit
postgres  | 2024-05-11 23:29:41.228 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
postgres  | 2024-05-11 23:29:41.228 UTC [1] LOG:  listening on IPv6 address "::", port 5432
postgres  | 2024-05-11 23:29:41.234 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
postgres  | 2024-05-11 23:29:41.251 UTC [15] LOG:  database system was shut down at 2024-05-11 23:20:00 UTC
postgres  | 2024-05-11 23:29:41.268 UTC [1] LOG:  database system is ready to accept connections
app       | Watching for file changes with StatReloader
app       | Performing system checks...
app       |
app       | System check identified no issues (0 silenced).
app       | May 11, 2024 - 17:29:54
app       | Django version 5.0.4, using settings 'playground.settings'
app       | Starting development server at http://0.0.0.0:8000/
app       | Quit the server with CONTROL-C.
app       |
```
Go to [localhost](http://0.0.0.0:8000/) and you will see django page.

At this point, we have a default `postgres` db but we need to create our database user and Django database, for that to happen we need to open a shell from postgres container, to do that run:
```bash
$ docker exec -it postgres bash
# output
postgres@e6b2ecb3fdba:/$ 
```
Now that we are inside the postgres container, lets connect to our database and run the following commands (use the variables values from your `.env` file)
```bash
postgres@e6b2ecb3fdba:/$ psql -U postgres

postgres=# CREATE DATABASE django_db;
# CREATE DATABASE
postgres=# CREATE USER django_user WITH PASSWORD 'django_password';
# CREATE ROLE
postgres=# ALTER ROLE django_user SET client_encoding TO 'utf8';
# ALTER ROLE
postgres=# ALTER ROLE django_user SET default_transaction_isolation TO 'read committed';
# ALTER ROLE
postgres=# ALTER ROLE django_user SET timezone TO 'UTC';
# ALTER ROLE
postgres=# GRANT ALL PRIVILEGES ON DATABASE django_db TO django_user;
# GRANT
postgres=# ALTER DATABASE django_db OWNER TO django_user;
# ALTER DATABASE

postgres=# \q
postgres@17847735cf74:/$ exit
```
Now that we have the database for the Django application, we can update `playground/settings.py`
```bash
Then open `playground/settings.py` and update the DATABASES object:
```bash
# comment out
# DATABASES = {
#     'default': {
#         'ENGINE': 'django.db.backends.sqlite3',
#         'NAME': BASE_DIR / 'db.sqlite3',
#     }
# }

# and add this
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": os.environ.get("DB_NAME"),
        "USER": os.environ.get("DB_USER"),
        "PASSWORD": os.environ.get("DB_PASS"),
        "HOST": os.environ.get("DB_HOST"),
        "PORT": os.environ.get("DB_PORT"),
        "TEST": {
            "NAME": os.environ.get("TEST_DB_NAME"),
        }
    }
}
```
Press `ctrl + c` to stop the server and run again:
```bash
$ docker compose up --build
```
Now we have two containers up and running, let's connect to Django app server and run a couple of commands to add into the database, the basic structure:
```bash
$ docker exec -it app sh

/app $ python3 manage.py makemigrations
# Output
# No changes detected

/app $ python3 manage.py migrate
# Output
# Operations to perform: 
#   Apply all migrations: admin, auth, contenttypes, sessions
# Running migrations:
#   Applying contenttypes.0001_initial... OK
#   Applying auth.0001_initial... OK
#   Applying admin.0001_initial... OK
#   Applying admin.0002_logentry_remove_auto_add... OK
#   Applying admin.0003_logentry_add_action_flag_choices... OK
#   Applying contenttypes.0002_remove_content_type_name... OK
#   Applying auth.0002_alter_permission_name_max_length... OK
#   Applying auth.0003_alter_user_email_max_length... OK
#   Applying auth.0004_alter_user_username_opts... OK
#   Applying auth.0005_alter_user_last_login_null... OK
#   Applying auth.0006_require_contenttypes_0002... OK
#   Applying auth.0007_alter_validators_add_error_messages... OK
#   Applying auth.0008_alter_user_username_max_length... OK
#   Applying auth.0009_alter_user_last_name_max_length... OK
#   Applying auth.0010_alter_group_name_max_length... OK
#   Applying auth.0011_update_proxy_permissions... OK
#   Applying auth.0012_alter_user_first_name_max_length... OK
#   Applying sessions.0001_initial... OK

/app $ python3 manage.py createsuperuser
# Output
# Username (leave blank to use 'root'):
# Email address:
# Password:
# Password (again):
# Superuser created successfully.

/app $ exit
```
Go to [admin page](http://0.0.0.0:8000/admin) and use your credentials from last step.

If you have a Database explorer (PgAdmin, Datagrip, DBeaver, etc), you can connect to the local database using your credentials from your `.env` file.

I hope you find this instructions very helpful.

Happy Coding!