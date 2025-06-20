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

# 🎨 Templates & Placeholders dans django CMS

## 🧱 Templates

Les templates dans django CMS fonctionnent comme dans Django, avec l’héritage via `base.html`.

Un template pour django CMS doit inclure :

```html
{% load cms_tags sekizai_tags %}
<head>
    {% render_block "css" %}
</head>
<body>
    {% cms_toolbar %}
    {% placeholder "content" %}
    {% render_block "js" %}
</body>
```

### Exemple de configuration dans `settings.py`

```python
CMS_TEMPLATES = [
    ('bootstrap5.html', 'Bootstrap 5'),
    ('minimal.html', 'Minimal'),
    ('whitenoise-static-files-demo.html', 'Démo Fichiers Statiques'),
]
```

> 📁 Les templates se trouvent dans `backend/templates/` dans le projet quickstart.

---

## 🧩 Placeholders

Un **placeholder** est une zone dynamique dans le HTML que le CMS remplit avec du contenu stocké en base de données.

```html
{% block content %}
    {% placeholder "Feature" %}
    {% placeholder "Content" %}
    {% placeholder "Splashbox" %}
{% endblock %}
```

> Pour voir les zones disponibles, passe en mode **Structure** via l’interface (en haut à droite).

---

## ♻️ Aliases statiques (ex : pied de page)

Les aliases statiques sont utiles pour afficher **le même contenu sur plusieurs pages** (comme un footer géré via l’interface).

### Étapes :

1. Installer :

```bash
pip install djangocms-alias
```

2. Ajouter à `INSTALLED_APPS` :

```python
"djangocms_alias",
```

3. Dans `base.html`, insérer :

```html
{% load djangocms_alias_tags %}

{% block content %}
    ...
    <footer>
        {% static_alias 'footer' %}
    </footer>
{% endblock %}

{% render_block "js" %}
```

4. Une fois ajouté, va dans le menu “Aliases…” pour l’éditer depuis le CMS.

> 📝 Tu peux créer ou modifier son contenu comme une page normale. Le contenu est partagé sur toutes les pages.

---

## 📂 Menus dynamiques

Pour afficher le menu du CMS :

```html
{% load menu_tags %}

<ul class="nav">
    {% show_menu 0 100 100 100 %}
</ul>
```

> `show_menu` affiche la hiérarchie des pages. Les chiffres définissent la profondeur mais peuvent rester tels quels.

---

# 🔗 Intégration d’applications dans django CMS

## Pourquoi intégrer une app dans django CMS ?

Intégrer une application ne signifie pas juste la faire cohabiter avec django CMS, mais **lier leur logique** pour créer un site cohérent, facilement gérable et extensible via le CMS.

👍 Avantage : tu n’as **pas besoin de modifier l’application** originale (utile pour les apps tierces).

---

## 🗳 Exemple : Intégrer une app de sondages (polls)

### 1. Installer l’app `polls`

```bash
pip install git+http://git@github.com/divio/django-polls.git#egg=polls
```

Ajoute `'polls'` à `INSTALLED_APPS` dans `settings.py`.

### 2. Ajouter les URLs

```python
from django.urls import re_path, include
from django.conf.urls.i18n import i18n_patterns

urlpatterns = i18n_patterns(
    re_path(r'^admin/', include(admin.site.urls)),
    re_path(r'^polls/', include('polls.urls')),
    re_path(r'^', include('cms.urls')),
)
```

> 🔁 Important : les URLs de django CMS doivent être en dernier.

### 3. Migrer les données

```bash
python manage.py migrate polls
```

Va sur [http://localhost:8000/admin/](http://localhost:8000/admin/) → onglet **Polls**.

Crée un sondage :

- Question : *Quel navigateur préférez-vous ?*
- Choix : Safari, Firefox, Chrome

Visite ensuite : [http://localhost:8000/en/polls/](http://localhost:8000/en/polls/)

---

## 🎨 Améliorer l'intégration visuelle

Par défaut, les templates de `polls` sont minimaux. Fais-les hériter de `base.html` :

Crée ce fichier : `mysite/templates/polls/base.html` :

```html
{% extends 'base.html' %}

{% block content %}
    {% block polls_content %}
    {% endblock %}
{% endblock %}
```

Recharge `/polls/` : la page utilise maintenant le même design que le reste du site CMS.

---

## 🧩 Créer l’app d’intégration CMS

### 1. Créer l’app dédiée

```bash
python manage.py startapp polls_cms_integration
```

Structure obtenue :

```
mon-projet/
├── polls_cms_integration/
│   ├── models.py, views.py, etc.
```

### 2. Ajouter à `INSTALLED_APPS`

Dans `settings.py` :

```python
INSTALLED_APPS += ["polls_cms_integration"]
```

> Cette app servira à **relier polls au CMS** via un plugin personnalisé. Elle contiendra le code d’intégration (plugins, apphooks…).

---

💡 Prochaine étape : créer un **plugin CMS** qui affiche un sondage dans une page.

Souhaites-tu que je te prépare ce plugin complet directement ?