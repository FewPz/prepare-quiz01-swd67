# 📂 - Setting up the project environment.

---
## 🐍 - Create a virtual environment and django module
Create a directory for your project, for example, a folder named `quiz`. This will be your project directory. Then, use the command `cd quiz` to navigate into this directory. Once done, open your terminal and follow the instructions below.

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
