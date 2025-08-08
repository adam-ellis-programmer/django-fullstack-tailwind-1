# Complete Django + Docker + Tailwind Setup Guide

## Prerequisites

- Docker and Docker Compose installed
- Text editor

## Step 1: Create Project Directory

```bash
mkdir my-django-app
cd my-django-app
```

## Step 2: Create Dockerfile

Create `Dockerfile`:

```dockerfile
# Use Python 3.11 slim image
FROM python:3.11-slim

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

# Install system dependencies
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
        curl \
        build-essential \
    && curl -fsSL https://deb.nodesource.com/setup_18.x | bash - \
    && apt-get install -y nodejs \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Set work directory
WORKDIR /app

# Install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy project
COPY . .

# Expose port
EXPOSE 8000

# Run the application
CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## Step 3: Create requirements.txt

Create `requirements.txt`:

```txt
# Django and core packages
Django==5.0.1
djangorestframework==3.14.0

# Database
psycopg2-binary==2.9.9

# Environment variables
python-decouple==3.8

# CORS handling
django-cors-headers==4.3.1

# Tailwind CSS for Django
django-tailwind[reload]==3.6.0

# Hot reload for browser
django-browser-reload==1.12.1

# Additional useful packages
Pillow==10.2.0
django-extensions==3.2.3
whitenoise==6.6.0

# Development tools
black==23.12.1
flake8==7.0.0
```

## Step 4: Create docker-compose.yml

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      POSTGRES_DB: django_db
      POSTGRES_USER: django_user
      POSTGRES_PASSWORD: django_password
    ports:
      - '5432:5432'

  web:
    build: .
    command: python manage.py runserver 0.0.0.0:8000
    volumes:
      - .:/app
    ports:
      - '8000:8000'
    depends_on:
      - db
    environment:
      DEBUG: 1
      DATABASE_URL: postgres://django_user:django_password@db:5432/django_db
    stdin_open: true
    tty: true

volumes:
  postgres_data:
```

## Step 5: Create .env file

Create `.env`:

```env
# Django settings
DEBUG=True
SECRET_KEY=django-insecure-your-secret-key-here-change-in-production

# Database
DATABASE_URL=postgres://django_user:django_password@db:5432/django_db

# Tailwind settings
TAILWIND_APP_NAME=theme
```

## Step 6: Create .dockerignore

Create `.dockerignore`:

```gitignore
# Git
.git
.gitignore

# Docker
.dockerignore
Dockerfile
docker-compose.yml

# Python
__pycache__
*.pyc
*.pyo
*.pyd
.Python
env
pip-log.txt
pip-delete-this-directory.txt
.tox
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
*.log
.git
.mypy_cache
.pytest_cache
.hypothesis

# Django
*.log
local_settings.py
db.sqlite3
db.sqlite3-journal
media

# Environment variables
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# IDEs
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Node modules (for Tailwind)
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Static/Media files (these get generated)
staticfiles/
static/
media/

# Documentation
README.md
docs/
```

## Step 7: Build Docker Image

```bash
docker-compose build
```

## Step 8: Create Django Project

```bash
docker-compose run --rm web django-admin startproject config .
```

## Step 9: Configure Django Settings

Edit `config/settings.py`:

### Add these imports at the top:

```python
from pathlib import Path
import os
```

### Update ALLOWED_HOSTS:

```python
ALLOWED_HOSTS = ['localhost', '127.0.0.1', '0.0.0.0']
```

### Update INSTALLED_APPS:

```python
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",

    # Third party apps
    'corsheaders',
    'rest_framework',
    'django_browser_reload',
    'tailwind',
    'theme',  # Will be created by tailwind init
    'django_extensions',

    # Local apps will go here
]
```

### Update TEMPLATES:

```python
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / 'templates'],  # Add this line
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```

### Update DATABASES:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'django_db',
        'USER': 'django_user',
        'PASSWORD': 'django_password',
        'HOST': 'db',
        'PORT': '5432',
    }
}
```

### Add at the bottom of settings.py:

```python
# Tailwind CSS settings
TAILWIND_APP_NAME = 'theme'

INTERNAL_IPS = [
    "127.0.0.1",
    "0.0.0.0",
    "localhost",
]

# Browser reload middleware (development only)
if DEBUG:
    MIDDLEWARE += [
        'django_browser_reload.middleware.BrowserReloadMiddleware',
    ]
```

## Step 10: Start Services

```bash
docker-compose up -d
```

## Step 11: Run Database Migrations

```bash
docker-compose exec web python manage.py migrate
```

## Step 12: Install Tailwind

```bash
docker-compose exec web python manage.py tailwind install
```

## Step 13: Initialize Tailwind Theme App

```bash
docker-compose exec web python manage.py tailwind init
```

Press Enter when prompted for app name (accepts default "theme").

## Step 14: Create Core Django App

```bash
docker-compose exec web python manage.py startapp core
```

## Step 15: Add Core App to Settings

Add `'core',` to INSTALLED_APPS in `config/settings.py`.

## Step 16: Create Templates Directory

```bash
mkdir -p templates/core
```

## Step 17: Create URL Configuration

### Create core/urls.py:

```python
from django.urls import path
from . import views

app_name = 'core'

urlpatterns = [
    path('', views.home, name='home'),
    path('about/', views.about, name='about'),
    path('contact/', views.contact, name='contact'),
]
```

### Update config/urls.py:

```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('core.urls')),
]

if settings.DEBUG:
    urlpatterns += [
        path("__reload__/", include("django_browser_reload.urls")),
    ]
```

## Step 18: Create Views

Edit `core/views.py`:

```python
from django.shortcuts import render

def home(request):
    return render(request, 'core/home.html')

def about(request):
    return render(request, 'core/about.html')

def contact(request):
    return render(request, 'core/contact.html')
```

## Step 19: Create Base Template

Create `templates/base.html`:

```html
{% load static tailwind_tags %}
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{% block title %}Django + Tailwind{% endblock %}</title>
    {% tailwind_css %}
  </head>
  <body class="bg-gray-100 min-h-screen">
    <!-- Navigation -->
    <nav class="bg-blue-600 shadow-lg">
      <div class="max-w-7xl mx-auto px-4">
        <div class="flex justify-between h-16">
          <div class="flex items-center">
            <a href="{% url 'core:home' %}" class="text-white text-xl font-bold"
              >Django App</a
            >
          </div>
          <div class="flex items-center space-x-4">
            <a
              href="{% url 'core:home' %}"
              class="text-white hover:text-blue-200 px-3 py-2 rounded-md"
              >Home</a
            >
            <a
              href="{% url 'core:about' %}"
              class="text-white hover:text-blue-200 px-3 py-2 rounded-md"
              >About</a
            >
            <a
              href="{% url 'core:contact' %}"
              class="text-white hover:text-blue-200 px-3 py-2 rounded-md"
              >Contact</a
            >
          </div>
        </div>
      </div>
    </nav>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto py-6 px-4">
      {% block content %} {% endblock %}
    </main>

    <!-- Footer -->
    <footer class="bg-gray-800 text-white mt-auto">
      <div class="max-w-7xl mx-auto py-4 px-4 text-center">
        <p>&copy; 2025 Django + Tailwind Test App</p>
      </div>
    </footer>
  </body>
</html>
```

## Step 20: Create Home Template

Create `templates/core/home.html`:

```html
{% extends 'base.html' %} {% block title %}Home - Django + Tailwind{% endblock
%} {% block content %}
<div class="text-center">
  <h1 class="text-4xl font-bold text-gray-900 mb-4">
    Welcome to Django + Tailwind!
  </h1>
  <p class="text-xl text-gray-600 mb-8">
    Testing Tailwind CSS integration with Django
  </p>

  <!-- Tailwind Test Components -->
  <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mt-8">
    <!-- Card 1 -->
    <div class="bg-white rounded-lg shadow-md p-6">
      <div class="text-blue-500 text-3xl mb-4">🚀</div>
      <h3 class="text-lg font-semibold mb-2">Fast Setup</h3>
      <p class="text-gray-600">
        Django + Docker + Tailwind configured and ready to go!
      </p>
    </div>

    <!-- Card 2 -->
    <div class="bg-white rounded-lg shadow-md p-6">
      <div class="text-green-500 text-3xl mb-4">💡</div>
      <h3 class="text-lg font-semibold mb-2">Modern UI</h3>
      <p class="text-gray-600">
        Beautiful, responsive design with Tailwind CSS utilities.
      </p>
    </div>

    <!-- Card 3 -->
    <div class="bg-white rounded-lg shadow-md p-6">
      <div class="text-purple-500 text-3xl mb-4">⚡</div>
      <h3 class="text-lg font-semibold mb-2">Hot Reload</h3>
      <p class="text-gray-600">
        Automatic CSS compilation and browser refresh.
      </p>
    </div>
  </div>

  <!-- Button Test -->
  <div class="mt-8">
    <a
      href="{% url 'core:about' %}"
      class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-6 rounded-lg transition duration-300 ease-in-out transform hover:scale-105"
    >
      Learn More
    </a>
  </div>
</div>
{% endblock %}
```

## Step 21: Start Tailwind in Watch Mode

In a new terminal:

```bash
docker-compose exec web python manage.py tailwind start
```

## Step 22: Test Your Application

Visit `http://localhost:8000` - you should see a beautiful styled page!

## Step 23: Test Hot Reload

1. Keep Tailwind running in one terminal
2. Edit any template file
3. Save the file
4. Browser should automatically refresh

## Final Project Structure

```
my-django-app/
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env
├── .dockerignore
├── manage.py
├── config/
│   ├── __init__.py
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── core/
│   ├── __init__.py
│   ├── admin.py
│   ├── apps.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   ├── tests.py
│   └── migrations/
├── templates/
│   ├── base.html
│   └── core/
│       └── home.html
└── theme/
    ├── __init__.py
    ├── apps.py
    └── static_src/
        ├── package.json
        ├── tailwind.config.js
        └── src/
            └── styles.css
```

## Daily Development Workflow

1. `docker-compose up -d` (start services)
2. `docker-compose exec web python manage.py tailwind start` (start CSS compilation)
3. Edit templates/views
4. Browser automatically refreshes!

## Troubleshooting

- If Tailwind styles don't appear: Restart `tailwind start`
- If hot reload doesn't work: Check browser console for WebSocket errors
- If Django shows welcome page: Check URL configuration
- If containers won't start: Check Docker logs with `docker-compose logs`
