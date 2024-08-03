# 📂 - Setting up the project environment.

---
## 🐍 - Create a virtual environment and django module
Create a directory for your project, for example, a folder named `lab`. This will be your project directory. Then, use the command `cd lab` to navigate into this directory. Once done, open your terminal and follow the instructions below.

```bash
pip install virtualenv
python -m venv env 
env\Scripts\activate.bat
```
* Installs the virtualenv package to create isolated Python environments.
* Creates a virtual environment named `env` using the built-in venv module.
* Activates the `env` virtual environment so you can use it.

Install Django
```bash
pip install django
```
To check if Django is installed, use the following command:
```bash
python -m django --version
```
Expected output:
```bash
> 4.2.13
```
This output will display the installed version of the Django module.

## 🦒 - Creating a Django project
Run the following command to create a new Django project named `quiz`
```bash
django-admin startproject quiz
```
You'll get a project file structure like this:
```bash
quiz/
    manage.py
    quiz/
        __init__.py
        settings.py  # for setting your project
        urls.py      # for setting path your project
        asgi.py
        wsgi.py
```
## ⭐️ - Creating a application
An "app" in Django is a web application that does something, such as a blog, a forum, or a simple poll. Each app typically focuses on a specific piece of functionality
When you run `python manage.py startapp [app_name]`, Django generates a new directory named `[app_name]` containing the basic files and folders needed for the app

Let's create an app called "Polls" using the command
```bash
python manage.py startapp polls
```

This command will create a folder named `polls` containing the following files
```bash
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
```
## 🎒 - Database Setup
First, ensure PostgreSQL is installed by checking the version with the command
```bash
postgres --version
```
Expected output
```
postgres (PostgreSQL) 15.0
```
Next, install the Postgres Client psycopg2:
```
pip install psycopg2
pip install psycopg2-binary
```
Then, open the `quiz/settings.py` file and locate the section:
```bash
# Database
# https://docs.djangoproject.com/en/4.2/ref/settings/#databases

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```
Modify it to use PostgreSQL as follows
```bash
# Database
# https://docs.djangoproject.com/en/4.2/ref/settings/#databases

DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.postgresql",
        "NAME": "mypolls",
        "USER": "db_username",
        "PASSWORD": "password",
        "HOST": "localhost",
        "PORT": "5432",
    }
}
```
