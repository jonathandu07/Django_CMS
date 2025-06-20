# Django_CMS
AutoFormation Django CMS
# 📘 Installer django CMS

## 🔧 Prérequis

Pas besoin d’être expert Django ou Python. Plusieurs options gratuites existent :
- **Divio Cloud** (simple, rapide, mais propriétaire)
- **Docker** (recommandé pour tests ou déploiement local)
- **Installation manuelle** (pour les devs qui veulent tout contrôler)

---

## 🚀 Méthode 1 – Installation rapide avec Docker

### Étapes :

```bash
git clone https://github.com/django-cms/django-cms-quickstart.git
cd django-cms-quickstart
docker compose build web
docker compose up -d database_default
docker compose run web python manage.py migrate
docker compose run web python manage.py createsuperuser
docker compose up -d
```

> Accède à `http://localhost:8000/admin` pour te connecter.

### Créer et publier ta première page :
- Clique sur **Create** > **New Page** > Suivant
- Renseigne un titre > clique sur **Create**
- Clique ensuite sur **Publish** pour la rendre visible

💡 Pour activer la barre d’édition : ajoute `?toolbar_on` à l’URL

---

## 🔧 Méthode 2 – Installation manuelle (sans Docker)

### 1. Créer un environnement virtuel

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install django-cms
```

### 2. Générer le projet

```bash
djangocms mon_projet
cd mon_projet
python manage.py runserver
```

📝 Cela :
- Crée un projet Django préconfiguré
- Ajoute les plugins de base (ckeditor, frontend, filer…)
- Lance les migrations
- Te demande de créer un superuser
- Vérifie la config avec : `python manage.py cms check`

---

## 🧩 Ajouter django CMS à un projet Django existant

### Modifie `settings.py` :

```python
INSTALLED_APPS += [
    "django.contrib.sites",
    "cms",
    "menus",
    "treebeard",
    "sekizai",
]
SITE_ID = 1
```

Optionnel mais conseillé :

```bash
pip install djangocms-admin-style
```

Ajoute dans `INSTALLED_APPS` **avant** `django.contrib.admin` :

```python
"djangocms_admin_style",
```

### Configuration linguistique :

```python
LANGUAGE_CODE = "fr"
LANGUAGES = [("fr", "Français"), ("en", "Anglais")]
```

### Middleware et context_processors :

Ajoute dans `MIDDLEWARE` :

```python
"django.middleware.locale.LocaleMiddleware",
"cms.middleware.user.CurrentUserMiddleware",
"cms.middleware.page.CurrentPageMiddleware",
"cms.middleware.toolbar.ToolbarMiddleware",
"cms.middleware.language.LanguageCookieMiddleware",
```

Ajoute dans `TEMPLATES["OPTIONS"]["context_processors"]` :

```python
"django.template.context_processors.i18n",
"sekizai.context_processors.sekizai",
"cms.context_processors.cms_settings",
```

---

## 📁 Templates et fichiers statiques

### Structure minimale `templates/home.html` :

```html
{% load cms_tags sekizai_tags %}
<html>
<head>
    <title>{% page_attribute "page_title" %}</title>
    {% render_block "css" %}
</head>
<body>
    {% cms_toolbar %}
    {% placeholder "content" %}
    {% render_block "js" %}
</body>
</html>
```

Ajoute dans `settings.py` :

```python
CMS_TEMPLATES = [('home.html', 'Template accueil')]
TEMPLATES[0]['DIRS'] = ['templates']
```

Fichiers médias :

```python
MEDIA_URL = "/media/"
MEDIA_ROOT = os.path.join(BASE_DIR, "media")
```

Dans `urls.py` :

```python
from django.conf import settings
from django.conf.urls.static import static
urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## 🔌 Plugins recommandés

### Versioning & alias

```bash
pip install djangocms-versioning djangocms-alias
```

Ajoute à `INSTALLED_APPS` :

```python
"djangocms_versioning",
"djangocms_alias",
```

### Gestion des fichiers (filer)

```bash
pip install django-filer>=3.0
```

Ajoute à `INSTALLED_APPS` :

```python
"filer",
"easy_thumbnails",
```

Et dans `settings.py` :

```python
THUMBNAIL_HIGH_RESOLUTION = True
THUMBNAIL_PROCESSORS = (
    'easy_thumbnails.processors.colorspace',
    'easy_thumbnails.processors.autocrop',
    'filer.thumbnail_processors.scale_and_crop_with_subject_location',
    'easy_thumbnails.processors.filters'
)
```

### CKEditor

```bash
pip install djangocms-text-ckeditor
```

Ajoute à `INSTALLED_APPS`, puis :
```bash
python manage.py migrate
```

### Frontend (Bootstrap 5)

```bash
pip install djangocms-frontend djangocms-link
```

Ajoute les sous-modules :

```python
"djangocms_frontend",
"djangocms_frontend.contrib.accordion",
"djangocms_frontend.contrib.alert",
...
"djangocms_frontend.contrib.utilities",
```

### Autres plugins utiles

```bash
pip install djangocms-file djangocms-picture djangocms-video djangocms-googlemap djangocms-snippet djangocms-style
```

```python
"djangocms_file",
"djangocms_picture",
"djangocms_video",
"djangocms_googlemap",
"djangocms_snippet",
"djangocms_style",
```

---

## ✅ Vérification finale

```bash
python manage.py cms check
```

Cela analysera ta config et signalera les erreurs.

---

🎉 Voilà, tu as une base django CMS fonctionnelle et prête à être enrichie avec tes propres contenus ou modules.