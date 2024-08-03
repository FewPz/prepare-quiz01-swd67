## 📖 - Django Notebook Extension
### Install

Install django-extensions and jupyter notebook with the command
```sh
pip install django-extensions ipython jupyter notebook
```
Next, update the package versions in Jupyter and Notebook
```sh
pip install ipython==8.25.0 jupyter_server==2.14.1 jupyterlab==4.2.2 jupyterlab_server==2.27.2
```
Update the Notebook version
```sh
pip install notebook==6.5.6
```
* If there are issues installing or running Jupyter, try changing the Notebook version to `6.5.7`
* `pip install notebook==6.5.7`
  
Then, create a directory named `notebooks`
```sh
mkdir notebooks
```
Add django-extensions to INSTALLED_APPS in the settings.py file:
```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    
    "django_extensions"
]
```
Finally, start the Jupyter Notebook server with the command
```sh
python manage.py shell_plus --notebook
```

### Usage
To use a kernel named "DJango shell" and add the specified code in the first cell, you can format it as follows
```python
import os
os.environ["DJANGO_ALLOW_ASYNC_UNSAFE"] = "true"
```
Caution
* While this setting can be useful for development and debugging, using it in production requires careful consideration
