# MedFlow — Django Patient Management App for Freelance Physicians
### Sprint 0 — Complete Setup Guide (Windows 11 Compatible)

> **Stack sécurisée Windows 11 + Python 3.12/3.13/3.14**
> SQLite (inclus avec Django, zéro configuration) · PyMySQL en option ·
> Sessions en base · Celery désactivé proprement · HTMX · Tailwind CDN

---

## Ce qui a changé par rapport à la version originale

| Original | Cette version | Pourquoi |
|---|---|---|
| `mysqlclient` | **PyMySQL** (optionnel) | Pure Python — zéro compilation C |
| MySQL obligatoire | **SQLite par défaut** | Inclus avec Python — zéro install |
| Redis + sessions cache | **Sessions en base de données** | Pas de Redis requis en dev |
| Celery actif | **Celery désactivé proprement** | Import conditionnel — ne plante pas |
| `python-decouple` | Gardé | Pure Python, aucun risque |

> **Celery et Redis** peuvent être réactivés à Sprint 4 sans toucher à l'architecture.
> Tout est commenté avec `# Sprint 4 — requires Redis`, rien n'est perdu.

---

## Table of Contents

1. [Prérequis](#1-prérequis)
2. [Project Setup](#2-project-setup)
3. [Settings & Environment](#3-settings--environment)
4. [App Structure](#4-app-structure)
5. [Models](#5-models)
6. [Database Migrations](#6-database-migrations)
7. [Seeder — 100+ Records](#7-seeder--100-records)
8. [Admin Registration](#8-admin-registration)
9. [Core App — Views, URLs & Templates](#9-core-app--views-urls--templates)
10. [Running the Project](#10-running-the-project)
11. [Passer à MySQL plus tard](#11-passer-à-mysql-plus-tard)
12. [Troubleshooting](#12-troubleshooting)
13. [Architecture Overview](#13-architecture-overview)

---

## 1. Prérequis

### Ce dont tu as besoin

| Outil | Version | Où |
|---|---|---|
| Python | **3.12** (recommandé) | https://python.org — **cocher "Add to PATH"** |
| Git | any | https://git-scm.com |

> ⚠️ **Python 3.14 n'est pas recommandé** pour ce projet.
> Plusieurs packages (`PyMySQL`, `Faker`, etc.) n'ont pas encore de wheels
> stables pour 3.14 sur Windows. Installe Python 3.12 en parallèle :
>
> ```powershell
> # Vérifie ce que tu as
> python --version
> py -3.12 --version    # si plusieurs versions installées
> ```
>
> Si tu n'as que Python 3.14, ça fonctionnera quand même —
> les packages de cette stack sont tous **Pure Python** (pas de compilation C).

### SQLite

**Rien à installer.** SQLite est inclus dans Python et dans Django.
Le fichier de base de données `db.sqlite3` sera créé automatiquement
dans le dossier du projet au premier `migrate`.

### Packages Python — installation complète

```powershell
pip install django==5.2 python-decouple django-htmx faker django-extensions pytest pytest-django
```

C'est tout. Aucune dépendance système, aucune compilation.

---

## 2. Project Setup

### 2.1 Créer le dossier et le virtual environment

```powershell
# Crée le dossier projet
mkdir C:\My_Medflow_Django_Project
cd C:\My_Medflow_Django_Project

# Crée le venv avec Python 3.12 (recommandé)
py -3.12 -m venv venv
# OU si tu n'as qu'une version Python :
python -m venv venv

# Active le venv — ton prompt doit afficher (venv)
.\venv\Scripts\Activate.ps1

# Installe les dépendances
pip install django==5.2 python-decouple django-htmx faker django-extensions pytest pytest-django

# Vérifie que tout est bien dans le venv
pip list
```

> Si `Activate.ps1` est bloqué par la politique d'exécution PowerShell :
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

### 2.2 Créer le projet Django et les apps

```powershell
# Le point final place manage.py directement dans le dossier courant
django-admin startproject medflow .

# Crée le dossier apps et toutes les apps
mkdir apps
python manage.py startapp apps\core
python manage.py startapp apps\patients
python manage.py startapp apps\referrals
python manage.py startapp apps\consultations
python manage.py startapp apps\appointments
python manage.py startapp apps\history
python manage.py startapp apps\dashboard
python manage.py startapp apps\audit
python manage.py startapp apps\notifications
```

Crée les fichiers `__init__.py` manquants :

```powershell
# Crée apps/__init__.py
New-Item -ItemType File -Path "apps\__init__.py" -Force

# Crée les dossiers templates, static, media, scripts
mkdir templates\registration
mkdir templates\core
mkdir templates\partials
mkdir static
mkdir media
mkdir scripts
```

---

## 3. Settings & Environment

### 3.1 Créer le fichier `.env`

Crée `.env` à la **racine du projet** (même dossier que `manage.py`) :

```ini
# .env — ne jamais committer ce fichier

SECRET_KEY=medflow-secret-key-change-this-in-production-2025-xyz
DEBUG=True
ALLOWED_HOSTS=127.0.0.1,localhost

# Base de données — laisser vide pour SQLite (défaut)
# Remplir seulement si tu veux passer à MySQL/PostgreSQL
DB_ENGINE=sqlite
DB_NAME=
DB_USER=
DB_PASSWORD=
DB_HOST=127.0.0.1
DB_PORT=3306

# Email (console en développement)
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
DEFAULT_FROM_EMAIL=noreply@medflow.local
```

Ajoute `.env` au `.gitignore` :

```text
# .gitignore
.env
venv/
__pycache__/
*.pyc
db.sqlite3
media/
staticfiles/
```

### 3.2 `medflow/settings.py` — complet

Remplace **tout** le contenu du `settings.py` auto-généré par :

```python
# medflow/settings.py
"""
MedFlow — Django Settings
Sprint 0 — Windows 11 Compatible
Base de données : SQLite (défaut) ou MySQL via PyMySQL
"""

import sys
from pathlib import Path
from decouple import config

BASE_DIR = Path(__file__).resolve().parent.parent

# ── Sécurité ──────────────────────────────────────────────────────────────────
SECRET_KEY    = config("SECRET_KEY",
                        default="medflow-insecure-dev-key-change-in-prod")
DEBUG         = config("DEBUG", default=True, cast=bool)
ALLOWED_HOSTS = config("ALLOWED_HOSTS",
                        default="127.0.0.1,localhost").split(",")

# ── Applications ──────────────────────────────────────────────────────────────
DJANGO_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "django.contrib.humanize",
]

THIRD_PARTY_APPS = [
    "django_htmx",
    "django_extensions",
]

LOCAL_APPS = [
    "apps.core",
    "apps.patients",
    "apps.referrals",
    "apps.consultations",
    "apps.appointments",
    "apps.history",
    "apps.dashboard",
    "apps.audit",
    "apps.notifications",
]

INSTALLED_APPS = DJANGO_APPS + THIRD_PARTY_APPS + LOCAL_APPS

# ── Middleware ─────────────────────────────────────────────────────────────────
MIDDLEWARE = [
    "django.middleware.security.SecurityMiddleware",
    "django.contrib.sessions.middleware.SessionMiddleware",
    "django.middleware.common.CommonMiddleware",
    "django.middleware.csrf.CsrfViewMiddleware",
    "django.contrib.auth.middleware.AuthenticationMiddleware",
    "django.contrib.messages.middleware.MessageMiddleware",
    "django.middleware.clickjacking.XFrameOptionsMiddleware",
    "django_htmx.middleware.HtmxMiddleware",
]

ROOT_URLCONF = "medflow.urls"

# ── Templates ─────────────────────────────────────────────────────────────────
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [BASE_DIR / "templates"],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
                "apps.core.context_processors.site_context",
            ],
        },
    }
]

WSGI_APPLICATION = "medflow.wsgi.application"

# ── Base de données ───────────────────────────────────────────────────────────
#
#  Par défaut : SQLite — aucune configuration requise.
#  Le fichier db.sqlite3 est créé automatiquement au premier migrate.
#
#  Pour passer à MySQL plus tard, voir Section 11 de ce guide.
#
DB_ENGINE = config("DB_ENGINE", default="sqlite")

if DB_ENGINE == "mysql":
    # MySQL via PyMySQL (pure Python — pip install pymysql)
    import pymysql
    pymysql.install_as_MySQLdb()
    DATABASES = {
        "default": {
            "ENGINE":   "django.db.backends.mysql",
            "NAME":     config("DB_NAME",     default="medflow_db"),
            "USER":     config("DB_USER",     default="medflow_user"),
            "PASSWORD": config("DB_PASSWORD", default=""),
            "HOST":     config("DB_HOST",     default="127.0.0.1"),
            "PORT":     config("DB_PORT",     default="3306"),
            "OPTIONS": {
                "charset":      "utf8mb4",
                "init_command": "SET sql_mode='STRICT_TRANS_TABLES'",
            },
            "TEST": {"NAME": "test_medflow_db"},
        }
    }
else:
    # SQLite — défaut, aucune dépendance externe
    DATABASES = {
        "default": {
            "ENGINE": "django.db.backends.sqlite3",
            "NAME":   BASE_DIR / "db.sqlite3",
        }
    }

# ── Auth ──────────────────────────────────────────────────────────────────────
AUTH_PASSWORD_VALIDATORS = [
    {"NAME": "django.contrib.auth.password_validation.UserAttributeSimilarityValidator"},
    {"NAME": "django.contrib.auth.password_validation.MinimumLengthValidator"},
    {"NAME": "django.contrib.auth.password_validation.CommonPasswordValidator"},
    {"NAME": "django.contrib.auth.password_validation.NumericPasswordValidator"},
]

LOGIN_URL           = "/auth/login/"
LOGIN_REDIRECT_URL  = "/dashboard/"
LOGOUT_REDIRECT_URL = "/auth/login/"

# ── Internationalisation ──────────────────────────────────────────────────────
LANGUAGE_CODE = "fr-fr"
TIME_ZONE     = "America/Edmonton"
USE_I18N      = True
USE_TZ        = True

# ── Static & Media ────────────────────────────────────────────────────────────
STATIC_URL       = "/static/"
STATICFILES_DIRS = [BASE_DIR / "static"]
STATIC_ROOT      = BASE_DIR / "staticfiles"
MEDIA_URL        = "/media/"
MEDIA_ROOT       = BASE_DIR / "media"

DEFAULT_AUTO_FIELD = "django.db.models.BigAutoField"

# ── Sessions — base de données (pas besoin de Redis) ─────────────────────────
SESSION_ENGINE = "django.contrib.sessions.backends.db"

# ── Cache — mémoire locale (pas besoin de Redis) ──────────────────────────────
CACHES = {
    "default": {
        "BACKEND": "django.core.cache.backends.locmem.LocMemCache",
    }
}

# ── Celery — désactivé en Sprint 0 ───────────────────────────────────────────
# Sprint 4 : installer Redis + Memurai sur Windows, puis décommenter
#
# REDIS_URL                = "redis://127.0.0.1:6379/0"
# CELERY_BROKER_URL        = REDIS_URL
# CELERY_RESULT_BACKEND    = REDIS_URL
# CELERY_ACCEPT_CONTENT    = ["json"]
# CELERY_TASK_SERIALIZER   = "json"
# CELERY_RESULT_SERIALIZER = "json"
# CELERY_TIMEZONE          = TIME_ZONE
# CELERY_BEAT_SCHEDULE     = {}

# ── Email ─────────────────────────────────────────────────────────────────────
EMAIL_BACKEND      = config("EMAIL_BACKEND",
                             default="django.core.mail.backends.console.EmailBackend")
DEFAULT_FROM_EMAIL = config("DEFAULT_FROM_EMAIL",
                             default="noreply@medflow.local")

# ── Sécurité production (ignoré si DEBUG=True) ────────────────────────────────
if not DEBUG:
    SECURE_SSL_REDIRECT            = True
    SESSION_COOKIE_SECURE          = True
    CSRF_COOKIE_SECURE             = True
    SECURE_HSTS_SECONDS            = 31536000
    SECURE_HSTS_INCLUDE_SUBDOMAINS = True
    SECURE_CONTENT_TYPE_NOSNIFF    = True
    X_FRAME_OPTIONS                = "DENY"
```

### 3.3 `medflow/__init__.py`

```python
# medflow/__init__.py
# Celery désactivé en Sprint 0 — activer à Sprint 4
# from .celery import app as celery_app
# __all__ = ("celery_app",)
```

### 3.4 `medflow/celery.py` — désactivé proprement

Ce fichier existe mais **n'est pas importé**. Il sera activé à Sprint 4.

```python
# medflow/celery.py
# Sprint 4 — activer quand Redis (Memurai) est installé sur Windows
#
# import os
# from celery import Celery
#
# os.environ.setdefault("DJANGO_SETTINGS_MODULE", "medflow.settings")
# app = Celery("medflow")
# app.config_from_object("django.conf:settings", namespace="CELERY")
# app.autodiscover_tasks()
#
# @app.task(bind=True, ignore_result=True)
# def debug_task(self):
#     print(f"Request: {self.request!r}")
```

### 3.5 `medflow/urls.py`

```python
# medflow/urls.py
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    path("admin/",  admin.site.urls),
    path("auth/",   include("django.contrib.auth.urls")),
    path("",        include("apps.core.urls")),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL,
                          document_root=settings.MEDIA_ROOT)
    urlpatterns += static(settings.STATIC_URL,
                          document_root=settings.STATIC_ROOT)
```

### 3.6 `pytest.ini`

```ini
[pytest]
DJANGO_SETTINGS_MODULE = medflow.settings
python_files = tests.py test_*.py *_tests.py
addopts = -v --tb=short
```

---

## 4. App Structure

```
C:\My_Medflow_Django_Project\         ← racine (manage.py ici)
│
├── medflow/
│   ├── __init__.py                   ← Celery commenté
│   ├── celery.py                     ← commenté — Sprint 4
│   ├── settings.py
│   ├── urls.py
│   └── wsgi.py
│
├── apps/
│   ├── __init__.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── apps.py
│   │   ├── context_processors.py
│   │   ├── urls.py
│   │   └── views.py
│   ├── patients/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   ├── referrals/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   ├── consultations/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   ├── appointments/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   ├── history/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   ├── dashboard/
│   │   ├── __init__.py
│   │   └── apps.py
│   ├── audit/
│   │   ├── __init__.py
│   │   ├── admin.py
│   │   ├── apps.py
│   │   └── models.py
│   └── notifications/
│       ├── __init__.py
│       └── apps.py
│
├── templates/
│   ├── base.html
│   ├── registration/
│   │   └── login.html
│   ├── core/
│   │   └── dashboard.html
│   └── partials/
│       └── stat_card.html
│
├── static/
├── media/
├── scripts/
├── db.sqlite3                        ← créé automatiquement au premier migrate
├── seed_data.py
├── pytest.ini
├── .env
├── .env.example
├── .gitignore
└── manage.py
```

### `apps/<name>/apps.py` — copie pour chaque app

**`apps/core/apps.py`**
```python
from django.apps import AppConfig

class CoreConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.core"
    label = "core"
```

**`apps/patients/apps.py`**
```python
from django.apps import AppConfig

class PatientsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.patients"
    label = "patients"
```

**`apps/referrals/apps.py`**
```python
from django.apps import AppConfig

class ReferralsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.referrals"
    label = "referrals"
```

**`apps/consultations/apps.py`**
```python
from django.apps import AppConfig

class ConsultationsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.consultations"
    label = "consultations"
```

**`apps/appointments/apps.py`**
```python
from django.apps import AppConfig

class AppointmentsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.appointments"
    label = "appointments"
```

**`apps/history/apps.py`**
```python
from django.apps import AppConfig

class HistoryConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.history"
    label = "history"
```

**`apps/dashboard/apps.py`**
```python
from django.apps import AppConfig

class DashboardConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.dashboard"
    label = "dashboard"
```

**`apps/audit/apps.py`**
```python
from django.apps import AppConfig

class AuditConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.audit"
    label = "audit"
```

**`apps/notifications/apps.py`**
```python
from django.apps import AppConfig

class NotificationsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name  = "apps.notifications"
    label = "notifications"
```

---

## 5. Models

### 5.1 `apps/patients/models.py`

```python
# apps/patients/models.py
from django.db import models


class Tag(models.Model):
    """Étiquette réutilisable pour les patients : chronique, diabétique, etc."""
    name = models.CharField(max_length=50, unique=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ["name"]


class ConsultationSite(models.Model):
    """Lieu physique où le médecin consulte."""
    name      = models.CharField(max_length=200)
    address   = models.TextField(blank=True)
    latitude  = models.DecimalField(
        max_digits=10, decimal_places=7, null=True, blank=True
    )
    longitude = models.DecimalField(
        max_digits=10, decimal_places=7, null=True, blank=True
    )
    telephone = models.CharField(max_length=30, blank=True)
    is_active = models.BooleanField(default=True)

    def __str__(self):
        return self.name

    class Meta:
        ordering = ["name"]


class Patient(models.Model):
    """Dossier principal du patient."""

    class SexChoices(models.TextChoices):
        MALE   = "M", "Masculin"
        FEMALE = "F", "Féminin"
        OTHER  = "O", "Autre / Non précisé"

    # Identité
    first_name    = models.CharField(max_length=100)
    last_name     = models.CharField(max_length=100)
    date_of_birth = models.DateField()
    sex           = models.CharField(
        max_length=1, choices=SexChoices.choices, default=SexChoices.OTHER
    )
    # Contact
    telephone = models.CharField(max_length=30, blank=True)
    email     = models.EmailField(blank=True)
    address   = models.TextField(blank=True)

    # Méta clinique
    tags           = models.ManyToManyField(
        Tag, blank=True, related_name="patients"
    )
    health_card_no = models.CharField(
        max_length=50, blank=True,
        verbose_name="N° carte santé / assurance"
    )
    notes    = models.TextField(blank=True)
    is_active = models.BooleanField(default=True)

    # Horodatages
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.first_name} {self.last_name}"

    @property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"

    class Meta:
        ordering = ["last_name", "first_name"]
```

### 5.2 `apps/referrals/models.py`

```python
# apps/referrals/models.py
from django.db import models
from apps.patients.models import Patient


class ReferralSource(models.Model):
    """Entité référente : médecin, clinique ou hôpital."""

    class SourceType(models.TextChoices):
        PHYSICIAN = "PHY", "Médecin"
        CLINIC    = "CLI", "Clinique"
        HOSPITAL  = "HOS", "Hôpital"
        OTHER     = "OTH", "Autre"

    name        = models.CharField(max_length=200)
    source_type = models.CharField(
        max_length=3, choices=SourceType.choices,
        default=SourceType.PHYSICIAN
    )
    telephone = models.CharField(max_length=30, blank=True)
    email     = models.EmailField(blank=True)
    address   = models.TextField(blank=True)
    notes     = models.TextField(blank=True)
    is_active = models.BooleanField(default=True)

    def __str__(self):
        return f"{self.name} ({self.get_source_type_display()})"

    class Meta:
        ordering = ["name"]


class Referral(models.Model):
    """Patient adressé par une source de référence."""

    class StatusChoices(models.TextChoices):
        PENDING   = "PEND", "En attente"
        CONFIRMED = "CONF", "Confirmée"
        COMPLETED = "DONE", "Terminée"
        CANCELLED = "CANC", "Annulée"
        DISPUTED  = "DISP", "En litige"

    patient       = models.ForeignKey(
        Patient, on_delete=models.CASCADE, related_name="referrals"
    )
    source        = models.ForeignKey(
        ReferralSource, null=True, blank=True,
        on_delete=models.SET_NULL, related_name="referrals"
    )
    referral_date = models.DateField()
    reason        = models.TextField(blank=True)
    status        = models.CharField(
        max_length=4, choices=StatusChoices.choices,
        default=StatusChoices.PENDING
    )
    notes      = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"Référence #{self.pk} — {self.patient} via {self.source}"

    class Meta:
        ordering = ["-referral_date"]


class RewardLedger(models.Model):
    """Récompense financière due à une source de référence."""

    class PaymentStatus(models.TextChoices):
        PENDING = "PEND", "À payer"
        PAID    = "PAID", "Payée"
        HOLD    = "HOLD", "En suspens"
        WAIVED  = "WAIV", "Annulée"

    referral       = models.OneToOneField(
        Referral, on_delete=models.CASCADE, related_name="reward"
    )
    amount         = models.DecimalField(max_digits=8, decimal_places=2)
    payment_status = models.CharField(
        max_length=4, choices=PaymentStatus.choices,
        default=PaymentStatus.PENDING
    )
    payment_date = models.DateField(null=True, blank=True)
    notes        = models.TextField(blank=True)
    created_at   = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return (f"Récompense {self.pk} — "
                f"{self.amount} ({self.get_payment_status_display()})")

    class Meta:
        ordering = ["-created_at"]
```

### 5.3 `apps/consultations/models.py`

```python
# apps/consultations/models.py
from django.db import models
from apps.patients.models import Patient, ConsultationSite


class Consultation(models.Model):
    """Rencontre clinique entre le médecin et un patient."""

    class StatusChoices(models.TextChoices):
        DRAFT     = "DRFT", "Brouillon"
        VALIDATED = "VALI", "Validée"
        CANCELLED = "CANC", "Annulée"

    patient        = models.ForeignKey(
        Patient, on_delete=models.CASCADE, related_name="consultations"
    )
    site           = models.ForeignKey(
        ConsultationSite, null=True, blank=True,
        on_delete=models.SET_NULL, related_name="consultations"
    )
    date_time      = models.DateTimeField()
    reason         = models.CharField(
        max_length=300, blank=True,
        verbose_name="Motif / plainte principale"
    )
    clinical_notes = models.TextField(blank=True)
    diagnosis      = models.TextField(blank=True)
    blood_pressure = models.CharField(
        max_length=20, blank=True,
        verbose_name="TA (ex. 120/80)"
    )
    temperature    = models.DecimalField(
        max_digits=4, decimal_places=1,
        null=True, blank=True,
        verbose_name="Température (°C)"
    )
    weight_kg  = models.DecimalField(
        max_digits=5, decimal_places=2, null=True, blank=True
    )
    status     = models.CharField(
        max_length=4, choices=StatusChoices.choices,
        default=StatusChoices.DRAFT
    )
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return (f"Consultation #{self.pk} — {self.patient} "
                f"le {self.date_time.strftime('%Y-%m-%d')}")

    class Meta:
        ordering = ["-date_time"]


class Prescription(models.Model):
    """Médicament prescrit lors d'une consultation."""
    consultation  = models.ForeignKey(
        Consultation, on_delete=models.CASCADE,
        related_name="prescriptions"
    )
    medication    = models.CharField(max_length=200)
    dosage        = models.CharField(max_length=100, blank=True)
    frequency     = models.CharField(max_length=100, blank=True)
    duration_days = models.PositiveIntegerField(null=True, blank=True)
    remarks       = models.TextField(blank=True)

    def __str__(self):
        return f"{self.medication} ({self.dosage})"

    class Meta:
        ordering = ["medication"]


class ExamRequest(models.Model):
    """Examen complémentaire demandé lors d'une consultation."""

    class PriorityChoices(models.TextChoices):
        ROUTINE = "ROUT", "Routine"
        URGENT  = "URGE", "Urgent"

    class ExamStatusChoices(models.TextChoices):
        REQUESTED = "REQU", "Demandé"
        RECEIVED  = "RECV", "Résultats reçus"
        REVIEWED  = "REVD", "Analysé par le médecin"

    consultation = models.ForeignKey(
        Consultation, on_delete=models.CASCADE,
        related_name="exam_requests"
    )
    exam_type    = models.CharField(max_length=200)
    priority     = models.CharField(
        max_length=4, choices=PriorityChoices.choices,
        default=PriorityChoices.ROUTINE
    )
    status       = models.CharField(
        max_length=4, choices=ExamStatusChoices.choices,
        default=ExamStatusChoices.REQUESTED
    )
    result_notes = models.TextField(blank=True)
    requested_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"{self.exam_type} — {self.get_status_display()}"

    class Meta:
        ordering = ["-requested_at"]
```

### 5.4 `apps/appointments/models.py`

```python
# apps/appointments/models.py
from django.db import models
from apps.patients.models import Patient, ConsultationSite
from apps.consultations.models import Consultation


class Appointment(models.Model):
    """Rendez-vous planifié pour un patient."""

    class StatusChoices(models.TextChoices):
        SCHEDULED   = "SCHE", "Planifié"
        CONFIRMED   = "CONF", "Confirmé"
        COMPLETED   = "DONE", "Terminé"
        NO_SHOW     = "NOSH", "Non présenté"
        CANCELLED   = "CANC", "Annulé"
        RESCHEDULED = "RESC", "Reporté"

    patient             = models.ForeignKey(
        Patient, on_delete=models.CASCADE,
        related_name="appointments"
    )
    site                = models.ForeignKey(
        ConsultationSite, null=True, blank=True,
        on_delete=models.SET_NULL, related_name="appointments"
    )
    source_consultation = models.ForeignKey(
        Consultation, null=True, blank=True,
        on_delete=models.SET_NULL,
        related_name="follow_up_appointments",
        help_text="Consultation ayant généré ce rendez-vous"
    )
    date_time     = models.DateTimeField()
    reason        = models.CharField(max_length=300, blank=True)
    status        = models.CharField(
        max_length=4, choices=StatusChoices.choices,
        default=StatusChoices.SCHEDULED
    )
    reminder_sent = models.BooleanField(default=False)
    notes         = models.TextField(blank=True)
    created_at    = models.DateTimeField(auto_now_add=True)
    updated_at    = models.DateTimeField(auto_now=True)

    def __str__(self):
        return (f"RDV #{self.pk} — {self.patient} "
                f"le {self.date_time.strftime('%Y-%m-%d %H:%M')}")

    class Meta:
        ordering = ["date_time"]
```

### 5.5 `apps/history/models.py`

```python
# apps/history/models.py
from django.db import models
from apps.patients.models import Patient


class PatientHistoryEntry(models.Model):
    """Événement longitudinal attaché à un patient — timeline clinique."""

    class EventType(models.TextChoices):
        CONSULTATION = "CONS", "Consultation"
        EXAM_RESULT  = "EXAM", "Résultat d'examen"
        PRESCRIPTION = "PRSC", "Changement de traitement"
        NOTE         = "NOTE", "Note clinique"
        ALLERGY      = "ALLE", "Allergie enregistrée"
        BACKGROUND   = "BACK", "Antécédent"
        ALERT        = "ALRT", "Alerte"

    patient    = models.ForeignKey(
        Patient, on_delete=models.CASCADE,
        related_name="history_entries"
    )
    event_type = models.CharField(
        max_length=4, choices=EventType.choices,
        default=EventType.NOTE
    )
    event_date = models.DateField()
    title      = models.CharField(max_length=300)
    content    = models.JSONField(
        default=dict, blank=True,
        help_text="Contenu structuré de l'événement"
    )
    is_alert   = models.BooleanField(
        default=False,
        help_text="Afficher en évidence sur la fiche patient"
    )
    created_by = models.ForeignKey(
        "auth.User", null=True, blank=True,
        on_delete=models.SET_NULL,
        related_name="history_entries_created"
    )
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return (f"[{self.get_event_type_display()}] "
                f"{self.patient} — {self.title}")

    class Meta:
        ordering = ["-event_date", "-created_at"]


class Allergy(models.Model):
    """Allergie ou intolérance connue pour un patient."""

    class SeverityChoices(models.TextChoices):
        MILD     = "MILD", "Légère"
        MODERATE = "MODR", "Modérée"
        SEVERE   = "SEVR", "Sévère"
        UNKNOWN  = "UNKN", "Inconnue"

    patient    = models.ForeignKey(
        Patient, on_delete=models.CASCADE,
        related_name="allergies"
    )
    allergen   = models.CharField(max_length=200)
    reaction   = models.CharField(max_length=300, blank=True)
    severity   = models.CharField(
        max_length=4, choices=SeverityChoices.choices,
        default=SeverityChoices.UNKNOWN
    )
    onset_date = models.DateField(null=True, blank=True)
    is_active  = models.BooleanField(default=True)
    notes      = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return (f"{self.allergen} — {self.patient} "
                f"({self.get_severity_display()})")

    class Meta:
        ordering = ["allergen"]
        verbose_name_plural = "Allergies"
```

### 5.6 `apps/audit/models.py`

```python
# apps/audit/models.py
from django.db import models


class AuditLog(models.Model):
    """Piste d'audit immuable — chaque action sensible est enregistrée ici."""

    class ActionChoices(models.TextChoices):
        CREATE = "CREATE", "Créer"
        UPDATE = "UPDATE", "Modifier"
        DELETE = "DELETE", "Supprimer"
        LOGIN  = "LOGIN",  "Connexion"
        LOGOUT = "LOGOUT", "Déconnexion"
        EXPORT = "EXPORT", "Export"

    user        = models.ForeignKey(
        "auth.User", null=True, blank=True,
        on_delete=models.SET_NULL,
        related_name="audit_logs"
    )
    action      = models.CharField(
        max_length=10, choices=ActionChoices.choices
    )
    model_name  = models.CharField(max_length=100)
    object_id   = models.CharField(max_length=50, blank=True)
    object_repr = models.CharField(max_length=300, blank=True)
    details     = models.JSONField(default=dict, blank=True)
    ip_address  = models.GenericIPAddressField(null=True, blank=True)
    timestamp   = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        ts = self.timestamp.strftime("%Y-%m-%d %H:%M") if self.timestamp else "?"
        return (f"[{ts}] {self.user} — "
                f"{self.action} on {self.model_name} #{self.object_id}")

    class Meta:
        ordering = ["-timestamp"]
        default_permissions = ("add", "view")  # pas de change/delete sur les logs
```

---

## 6. Database Migrations

```powershell
# Génère les migrations app par app (respecte les dépendances FK)
python manage.py makemigrations patients
python manage.py makemigrations referrals
python manage.py makemigrations consultations
python manage.py makemigrations appointments
python manage.py makemigrations history
python manage.py makemigrations audit

# Applique toutes les migrations (crée db.sqlite3 automatiquement)
python manage.py migrate
```

Résultat attendu :

```
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions,
                        patients, referrals, consultations,
                        appointments, history, audit
Running migrations:
  Applying patients.0001_initial ... OK
  Applying referrals.0001_initial ... OK
  Applying consultations.0001_initial ... OK
  Applying appointments.0001_initial ... OK
  Applying history.0001_initial ... OK
  Applying audit.0001_initial ... OK
  ...
```

> Après le `migrate`, le fichier `db.sqlite3` apparaît dans la racine du projet.
> C'est ta base de données — tu peux l'ouvrir avec
> [DB Browser for SQLite](https://sqlitebrowser.org/) pour vérifier les tables.

---

## 7. Seeder — 100+ Records

Le seeder est un script Python **autonome** placé à la racine du projet.
Il utilise l'ORM Django directement après `django.setup()`.

```powershell
# venv actif, depuis la racine du projet
python seed_data.py
```

Résultat attendu :

```
Clearing existing data...
Tags created        : 12
Sites created       : 6
Referral sources    : 10
Patients created    : 100
Consultations       : 287   (varie — entre 1 et 5 par patient)
Prescriptions       : 412
Exam requests       : 193
Appointments        : 217
Referrals           : 80
Rewards             : 67
History entries     : 298
Allergies           : 87
Audit logs          : 10
─────────────────────────────────
Seed complete ✓
```

### `seed_data.py` — fichier complet

```python
# seed_data.py
"""
MedFlow — Seeder
Génère des données médicales réalistes pour le développement.

Lancer depuis la racine du projet (venv actif) :
    python seed_data.py

Pré-requis : pip install faker
Idempotent : efface et recrée toutes les données à chaque lancement.
"""

import os
import sys
import django
import random
from datetime import date
from decimal import Decimal

# ── Bootstrap Django ──────────────────────────────────────────────────────────
os.environ.setdefault("DJANGO_SETTINGS_MODULE", "medflow.settings")
sys.path.insert(0, os.path.dirname(os.path.abspath(__file__)))
django.setup()

from faker import Faker
from django.contrib.auth.models import User
from apps.patients.models      import Tag, ConsultationSite, Patient
from apps.referrals.models     import ReferralSource, Referral, RewardLedger
from apps.consultations.models import Consultation, Prescription, ExamRequest
from apps.appointments.models  import Appointment
from apps.history.models       import PatientHistoryEntry, Allergy
from apps.audit.models         import AuditLog

fake = Faker("fr_FR")
Faker.seed(2025)
random.seed(2025)

# ── 1. Nettoyage — ordre FK-safe ──────────────────────────────────────────────
print("Clearing existing data...")
AuditLog.objects.all().delete()
Allergy.objects.all().delete()
PatientHistoryEntry.objects.all().delete()
Appointment.objects.all().delete()
RewardLedger.objects.all().delete()
Referral.objects.all().delete()
ExamRequest.objects.all().delete()
Prescription.objects.all().delete()
Consultation.objects.all().delete()
Patient.objects.all().delete()
ReferralSource.objects.all().delete()
ConsultationSite.objects.all().delete()
Tag.objects.all().delete()

# ── 2. Tags ───────────────────────────────────────────────────────────────────
tag_names = [
    "Diabétique", "Hypertendu", "Asthmatique", "Senior 65+",
    "Femme enceinte", "Maladie chronique", "Immunodéprimé",
    "Obèse", "Fumeur", "VIH+", "Cardiaque", "Épileptique",
]
tags = [Tag.objects.create(name=n) for n in tag_names]
print(f"Tags created        : {len(tags)}")

# ── 3. Sites de consultation ──────────────────────────────────────────────────
site_data = [
    ("Clinique Saint-Luc",
     "12 Rue de la Paix, Yaoundé",       Decimal("3.8677"),  Decimal("11.5174")),
    ("Centre Médical du Centre",
     "45 Avenue Kennedy, Yaoundé",        Decimal("3.8700"),  Decimal("11.5215")),
    ("Hôpital Général",
     "Boulevard du 20 Mai, Yaoundé",      Decimal("3.8600"),  Decimal("11.5100")),
    ("Polyclinique de Bastos",
     "Quartier Bastos, Yaoundé",          Decimal("3.8900"),  Decimal("11.5300")),
    ("Cabinet Médical Mvan",
     "Rue des Manguiers, Mvan",           Decimal("3.8450"),  Decimal("11.5050")),
    ("Centre de Santé Biyem-Assi",
     "Avenue de l'Indépendance, Yaoundé", Decimal("3.8550"),  Decimal("11.5250")),
]
sites = []
for name, address, lat, lng in site_data:
    s = ConsultationSite.objects.create(
        name=name,
        address=address,
        latitude=lat,
        longitude=lng,
        telephone=fake.phone_number()[:20],
        is_active=True,
    )
    sites.append(s)
print(f"Sites created       : {len(sites)}")

# ── 4. Sources de référence ───────────────────────────────────────────────────
source_data = [
    ("Dr. Amina Bello",       "PHY"),
    ("Dr. Jean-Paul Mvondo",  "PHY"),
    ("Dr. Fatima Oumarou",    "PHY"),
    ("Clinique des Palmiers", "CLI"),
    ("Polyclinique Chanas",   "CLI"),
    ("Hôpital Central",       "HOS"),
    ("Centre Mère-Enfant",    "HOS"),
    ("Dr. Samuel Nkoa",       "PHY"),
    ("Laboratoire BioMed",    "OTH"),
    ("Dr. Christelle Eyenga", "PHY"),
]
referral_sources = []
for sname, stype in source_data:
    rs = ReferralSource.objects.create(
        name=sname,
        source_type=stype,
        telephone=fake.phone_number()[:20],
        email=fake.ascii_email(),
        is_active=True,
    )
    referral_sources.append(rs)
print(f"Referral sources    : {len(referral_sources)}")

# ── 5. Patients (100) ─────────────────────────────────────────────────────────
sex_choices = ["M", "F", "O"]
patients = []

for _ in range(100):
    sex = random.choice(sex_choices)
    if sex == "M":
        fn = fake.first_name_male()
    elif sex == "F":
        fn = fake.first_name_female()
    else:
        fn = fake.first_name()

    p = Patient.objects.create(
        first_name=fn,
        last_name=fake.last_name(),
        date_of_birth=fake.date_of_birth(minimum_age=5, maximum_age=90),
        sex=sex,
        telephone=fake.phone_number()[:20],
        email=fake.ascii_email() if random.random() > 0.3 else "",
        address=fake.address(),
        health_card_no=fake.bothify("??-######").upper(),
        notes=fake.sentence() if random.random() > 0.5 else "",
        is_active=random.choices([True, False], weights=[90, 10])[0],
    )
    p.tags.set(random.sample(tags, k=random.randint(0, 3)))
    patients.append(p)

print(f"Patients created    : {len(patients)}")

# ── 6. Consultations (~1–5 par patient) ──────────────────────────────────────
reasons = [
    "Fièvre persistante", "Douleurs abdominales", "Maux de tête",
    "Contrôle tension artérielle", "Renouvellement ordonnance",
    "Toux chronique", "Douleurs thoraciques", "Fatigue générale",
    "Bilan de santé annuel", "Grossesse — suivi prénatal",
    "Diabète — contrôle glycémique", "Éruption cutanée",
    "Douleurs articulaires", "Bilan post-opératoire", "Vertiges",
]
diagnoses = [
    "Infection virale des voies respiratoires supérieures",
    "Hypertension artérielle — stade 1",
    "Diabète de type 2 non contrôlé",
    "Migraine sans aura",
    "Gastrite aiguë",
    "Anémie ferriprive",
    "Lombalgie mécanique",
    "Dermatite atopique",
    "Anxiété généralisée",
    "Rhinosinusite chronique",
    "Hypothyroïdie",
    "RAS — aucune pathologie identifiée",
]

all_consultations = []
for patient in patients:
    for _ in range(random.randint(1, 5)):
        dt = fake.date_time_between(start_date="-3y", end_date="now")
        status = random.choices(
            ["DRFT", "VALI", "CANC"],
            weights=[10, 80, 10],
        )[0]
        bp = (f"{random.randint(100, 160)}/{random.randint(60, 100)}"
              if random.random() > 0.2 else "")
        temp = (Decimal(str(round(random.uniform(36.0, 40.5), 1)))
                if random.random() > 0.3 else None)
        weight = (Decimal(str(round(random.uniform(45.0, 130.0), 1)))
                  if random.random() > 0.3 else None)
        c = Consultation.objects.create(
            patient=patient,
            site=random.choice(sites),
            date_time=dt,
            reason=random.choice(reasons),
            clinical_notes=fake.paragraph(nb_sentences=3),
            diagnosis=random.choice(diagnoses) if status == "VALI" else "",
            blood_pressure=bp,
            temperature=temp,
            weight_kg=weight,
            status=status,
        )
        all_consultations.append(c)

print(f"Consultations       : {Consultation.objects.count()}")

# ── 7. Prescriptions ──────────────────────────────────────────────────────────
medications = [
    ("Paracétamol 500 mg",    "1 comprimé",  "3 fois/jour",   5),
    ("Amoxicilline 500 mg",   "1 gélule",    "3 fois/jour",   7),
    ("Metformine 500 mg",     "1 comprimé",  "2 fois/jour",  90),
    ("Amlodipine 5 mg",       "1 comprimé",  "1 fois/jour",  30),
    ("Ibuprofène 400 mg",     "1 comprimé",  "3 fois/jour",   5),
    ("Oméprazole 20 mg",      "1 gélule",    "1 fois/jour",  14),
    ("Salbutamol spray",      "2 bouffées",  "Au besoin",    30),
    ("Lévothyroxine 50 µg",   "1 comprimé",  "1 fois/jour",  90),
    ("Metronidazole 500 mg",  "1 comprimé",  "2 fois/jour",   7),
    ("Cetirizine 10 mg",      "1 comprimé",  "1 fois/soir",  30),
    ("Atorvastatine 20 mg",   "1 comprimé",  "1 fois/soir",  90),
    ("Glibenclamide 5 mg",    "1 comprimé",  "2 fois/jour",  30),
]

for c in all_consultations:
    if c.status == "VALI":
        for _ in range(random.randint(1, 3)):
            med, dosage, freq, duration = random.choice(medications)
            Prescription.objects.create(
                consultation=c,
                medication=med,
                dosage=dosage,
                frequency=freq,
                duration_days=duration,
                remarks=fake.sentence() if random.random() > 0.7 else "",
            )

print(f"Prescriptions       : {Prescription.objects.count()}")

# ── 8. Examens complémentaires ────────────────────────────────────────────────
exam_types = [
    "Numération Formule Sanguine (NFS)",
    "Glycémie à jeun",
    "Créatinine / Urée",
    "Bilan lipidique",
    "Échographie abdominale",
    "Radiographie thoracique",
    "ECG",
    "TSH / T4",
    "Test VIH",
    "Hémoculture",
    "ECBU (examen cytobactériologique des urines)",
    "Sérologie hépatite B & C",
]

for c in all_consultations:
    if c.status == "VALI" and random.random() > 0.5:
        for _ in range(random.randint(1, 2)):
            ExamRequest.objects.create(
                consultation=c,
                exam_type=random.choice(exam_types),
                priority=random.choice(["ROUT", "URGE"]),
                status=random.choice(["REQU", "RECV", "REVD"]),
                result_notes=fake.sentence() if random.random() > 0.6 else "",
            )

print(f"Exam requests       : {ExamRequest.objects.count()}")

# ── 9. Rendez-vous ────────────────────────────────────────────────────────────
appt_statuses = ["SCHE", "CONF", "DONE", "NOSH", "CANC", "RESC"]
appt_weights  = [30, 20, 25, 10, 10, 5]

for patient in random.sample(patients, k=80):
    patient_consults = [
        c for c in all_consultations
        if c.patient_id == patient.pk and c.status == "VALI"
    ]
    for _ in range(random.randint(1, 4)):
        src = None
        if patient_consults and random.random() > 0.5:
            src = random.choice(patient_consults)
        Appointment.objects.create(
            patient=patient,
            site=random.choice(sites),
            source_consultation=src,
            date_time=fake.date_time_between(start_date="-1y", end_date="+6M"),
            reason=random.choice(reasons),
            status=random.choices(appt_statuses, weights=appt_weights)[0],
            reminder_sent=random.choice([True, False]),
            notes=fake.sentence() if random.random() > 0.6 else "",
        )

print(f"Appointments        : {Appointment.objects.count()}")

# ── 10. Références ────────────────────────────────────────────────────────────
referral_statuses = ["PEND", "CONF", "DONE", "CANC", "DISP"]
referral_weights  = [15, 30, 40, 10, 5]

referral_objs = []
for patient in random.sample(patients, k=80):
    r = Referral.objects.create(
        patient=patient,
        source=random.choice(referral_sources),
        referral_date=fake.date_between(start_date="-2y", end_date="today"),
        reason=fake.sentence(),
        status=random.choices(referral_statuses, weights=referral_weights)[0],
        notes=fake.sentence() if random.random() > 0.5 else "",
    )
    referral_objs.append(r)

print(f"Referrals           : {Referral.objects.count()}")

# ── 11. Récompenses ───────────────────────────────────────────────────────────
for ref in referral_objs:
    if ref.status in ("CONF", "DONE") and random.random() > 0.1:
        pstatus = random.choices(
            ["PEND", "PAID", "HOLD"],
            weights=[40, 50, 10],
        )[0]
        paid_date = None
        if pstatus == "PAID":
            paid_date = fake.date_between(
                start_date=ref.referral_date,
                end_date="today",
            )
        RewardLedger.objects.create(
            referral=ref,
            amount=Decimal(str(round(random.uniform(5000, 50000), 0))),
            payment_status=pstatus,
            payment_date=paid_date,
            notes=fake.sentence() if random.random() > 0.7 else "",
        )

print(f"Rewards             : {RewardLedger.objects.count()}")

# ── 12. Historique patient ────────────────────────────────────────────────────
event_types = ["CONS", "EXAM", "PRSC", "NOTE", "ALLE", "BACK", "ALRT"]

for patient in patients:
    for _ in range(random.randint(2, 4)):
        PatientHistoryEntry.objects.create(
            patient=patient,
            event_type=random.choice(event_types),
            event_date=fake.date_between(start_date="-5y", end_date="today"),
            title=fake.sentence(nb_words=6),
            content={"detail": fake.paragraph(nb_sentences=2)},
            is_alert=random.choices([True, False], weights=[15, 85])[0],
        )

print(f"History entries     : {PatientHistoryEntry.objects.count()}")

# ── 13. Allergies ─────────────────────────────────────────────────────────────
allergens = [
    "Pénicilline", "Aspirine", "Ibuprofène", "Sulfamides",
    "Latex", "Arachides", "Fruits de mer", "Lait de vache",
    "Pollen", "Poussière", "Codéine", "Produit de contraste iodé",
]

for patient in random.sample(patients, k=65):
    for _ in range(random.randint(1, 2)):
        onset = (fake.date_between(start_date="-20y", end_date="today")
                 if random.random() > 0.4 else None)
        Allergy.objects.create(
            patient=patient,
            allergen=random.choice(allergens),
            reaction=fake.sentence() if random.random() > 0.4 else "",
            severity=random.choice(["MILD", "MODR", "SEVR", "UNKN"]),
            onset_date=onset,
            is_active=random.choices([True, False], weights=[85, 15])[0],
        )

print(f"Allergies           : {Allergy.objects.count()}")

# ── 14. Logs d'audit ──────────────────────────────────────────────────────────
admin_user, created = User.objects.get_or_create(
    username="admin",
    defaults={
        "email":        "admin@medflow.local",
        "is_staff":     True,
        "is_superuser": True,
    },
)
if created:
    admin_user.set_password("MedFlow2025!")
    admin_user.save()
    print("Admin user created  : admin / MedFlow2025!")

for patient in random.sample(patients, k=10):
    AuditLog.objects.create(
        user=admin_user,
        action="CREATE",
        model_name="Patient",
        object_id=str(patient.pk),
        object_repr=str(patient),
        details={"source": "seed"},
        ip_address="127.0.0.1",
    )

print(f"Audit logs          : {AuditLog.objects.count()}")

# ── Résumé ────────────────────────────────────────────────────────────────────
print("\n─────────────────────────────────")
print(f"Tags                : {Tag.objects.count()}")
print(f"Sites               : {ConsultationSite.objects.count()}")
print(f"Referral sources    : {ReferralSource.objects.count()}")
print(f"Patients            : {Patient.objects.count()}")
print(f"Consultations       : {Consultation.objects.count()}")
print(f"Prescriptions       : {Prescription.objects.count()}")
print(f"Exam requests       : {ExamRequest.objects.count()}")
print(f"Appointments        : {Appointment.objects.count()}")
print(f"Referrals           : {Referral.objects.count()}")
print(f"Rewards             : {RewardLedger.objects.count()}")
print(f"History entries     : {PatientHistoryEntry.objects.count()}")
print(f"Allergies           : {Allergy.objects.count()}")
print(f"Audit logs          : {AuditLog.objects.count()}")
print("─────────────────────────────────")
print("Seed complete ✓")
```

---

## 8. Admin Registration

### `apps/patients/admin.py`

```python
from django.contrib import admin
from .models import Tag, ConsultationSite, Patient

@admin.register(Tag)
class TagAdmin(admin.ModelAdmin):
    list_display  = ["name"]
    search_fields = ["name"]

@admin.register(ConsultationSite)
class ConsultationSiteAdmin(admin.ModelAdmin):
    list_display = ["name", "address", "is_active"]
    list_filter  = ["is_active"]

@admin.register(Patient)
class PatientAdmin(admin.ModelAdmin):
    list_display      = ["full_name", "date_of_birth", "sex",
                         "telephone", "is_active"]
    list_filter       = ["sex", "is_active", "tags"]
    search_fields     = ["first_name", "last_name", "email", "health_card_no"]
    filter_horizontal = ["tags"]
```

### `apps/referrals/admin.py`

```python
from django.contrib import admin
from .models import ReferralSource, Referral, RewardLedger

@admin.register(ReferralSource)
class ReferralSourceAdmin(admin.ModelAdmin):
    list_display  = ["name", "source_type", "is_active"]
    list_filter   = ["source_type", "is_active"]
    search_fields = ["name"]

@admin.register(Referral)
class ReferralAdmin(admin.ModelAdmin):
    list_display  = ["patient", "source", "referral_date", "status"]
    list_filter   = ["status"]
    search_fields = ["patient__first_name", "patient__last_name"]

@admin.register(RewardLedger)
class RewardLedgerAdmin(admin.ModelAdmin):
    list_display = ["referral", "amount", "payment_status", "payment_date"]
    list_filter  = ["payment_status"]
```

### `apps/consultations/admin.py`

```python
from django.contrib import admin
from .models import Consultation, Prescription, ExamRequest

@admin.register(Consultation)
class ConsultationAdmin(admin.ModelAdmin):
    list_display  = ["patient", "date_time", "site", "status"]
    list_filter   = ["status", "site"]
    search_fields = ["patient__first_name", "patient__last_name", "diagnosis"]

@admin.register(Prescription)
class PrescriptionAdmin(admin.ModelAdmin):
    list_display  = ["consultation", "medication", "dosage", "duration_days"]
    search_fields = ["medication"]

@admin.register(ExamRequest)
class ExamRequestAdmin(admin.ModelAdmin):
    list_display = ["consultation", "exam_type", "priority", "status"]
    list_filter  = ["priority", "status"]
```

### `apps/appointments/admin.py`

```python
from django.contrib import admin
from .models import Appointment

@admin.register(Appointment)
class AppointmentAdmin(admin.ModelAdmin):
    list_display  = ["patient", "date_time", "site", "status", "reminder_sent"]
    list_filter   = ["status", "site", "reminder_sent"]
    search_fields = ["patient__first_name", "patient__last_name"]
```

### `apps/history/admin.py`

```python
from django.contrib import admin
from .models import PatientHistoryEntry, Allergy

@admin.register(PatientHistoryEntry)
class PatientHistoryEntryAdmin(admin.ModelAdmin):
    list_display  = ["patient", "event_type", "event_date", "title", "is_alert"]
    list_filter   = ["event_type", "is_alert"]
    search_fields = ["patient__first_name", "patient__last_name", "title"]

@admin.register(Allergy)
class AllergyAdmin(admin.ModelAdmin):
    list_display  = ["patient", "allergen", "severity", "is_active"]
    list_filter   = ["severity", "is_active"]
    search_fields = ["allergen", "patient__first_name", "patient__last_name"]
```

### `apps/audit/admin.py`

```python
from django.contrib import admin
from .models import AuditLog

@admin.register(AuditLog)
class AuditLogAdmin(admin.ModelAdmin):
    list_display    = ["timestamp", "user", "action", "model_name", "object_id"]
    list_filter     = ["action", "model_name"]
    search_fields   = ["user__username", "object_repr"]
    readonly_fields = [f.name for f in AuditLog._meta.get_fields()
                       if hasattr(f, 'name')]

    def has_change_permission(self, request, obj=None):
        return False

    def has_delete_permission(self, request, obj=None):
        return False
```

---

## 9. Core App — Views, URLs & Templates

### `apps/core/context_processors.py`

```python
# apps/core/context_processors.py
def site_context(request):
    return {
        "app_name":    "MedFlow",
        "app_version": "0.1.0",
    }
```

### `apps/core/views.py`

```python
# apps/core/views.py
import datetime
from django.shortcuts import render, redirect
from django.contrib.auth.decorators import login_required
from django.contrib.auth import logout
from django.views.decorators.http import require_POST

from apps.patients.models      import Patient
from apps.consultations.models import Consultation
from apps.appointments.models  import Appointment
from apps.referrals.models     import Referral


def home(request):
    if request.user.is_authenticated:
        return redirect("dashboard")
    return redirect("login")


@login_required
def dashboard(request):
    today = datetime.date.today()
    context = {
        "page_title": "Tableau de bord",
        "stats": {
            "patients_actifs":       Patient.objects.filter(
                                         is_active=True).count(),
            "consultations_jour":    Consultation.objects.filter(
                                         date_time__date=today).count(),
            "rdv_semaine":           Appointment.objects.filter(
                                         status="SCHE").count(),
            "references_en_attente": Referral.objects.filter(
                                         status="PEND").count(),
        },
        "recent_patients":       Patient.objects.order_by("-created_at")[:5],
        "upcoming_appointments": Appointment.objects.filter(
                                     status__in=["SCHE", "CONF"]
                                 ).order_by("date_time")[:5],
    }
    return render(request, "core/dashboard.html", context)


@require_POST
def custom_logout(request):
    logout(request)
    return redirect("login")
```

### `apps/core/urls.py`

```python
# apps/core/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("",           views.home,          name="home"),
    path("dashboard/", views.dashboard,     name="dashboard"),
    path("logout/",    views.custom_logout, name="custom_logout"),
]
```

### `templates/registration/login.html`

```html
<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8">
  <title>Connexion — MedFlow</title>
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="min-h-screen bg-slate-900 flex items-center justify-center px-4">
<div class="w-full max-w-md bg-white rounded-2xl shadow-2xl p-8">

  <div class="text-center mb-8">
    <div class="w-14 h-14 bg-blue-600 rounded-2xl flex items-center justify-center mx-auto mb-4">
      <span class="text-white text-2xl">🩺</span>
    </div>
    <h1 class="text-2xl font-bold text-slate-800">MedFlow</h1>
    <p class="text-slate-400 text-sm mt-1">Gestion patients — Médecin freelance</p>
  </div>

  {% if form.errors %}
  <div class="bg-red-50 border border-red-200 text-red-700 rounded-lg px-4 py-3 mb-5 text-sm">
    Identifiant ou mot de passe incorrect.
  </div>
  {% endif %}

  <form method="post">
    {% csrf_token %}
    <input type="hidden" name="next" value="{{ next }}">

    <div class="mb-4">
      <label class="block text-sm font-medium text-slate-700 mb-1.5">
        Identifiant
      </label>
      <input type="text" name="username" autofocus
        class="w-full border border-slate-300 rounded-lg px-3 py-2.5 text-sm
               focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent">
    </div>

    <div class="mb-6">
      <label class="block text-sm font-medium text-slate-700 mb-1.5">
        Mot de passe
      </label>
      <input type="password" name="password"
        class="w-full border border-slate-300 rounded-lg px-3 py-2.5 text-sm
               focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent">
    </div>

    <button type="submit"
      class="w-full bg-blue-600 hover:bg-blue-500 active:bg-blue-700
             text-white font-semibold py-2.5 rounded-lg text-sm transition-colors">
      Se connecter →
    </button>
  </form>

  <p class="text-center text-slate-400 text-xs mt-6">
    MedFlow Sprint 0 · SQLite · Accès réservé
  </p>
</div>
</body>
</html>
```

### `templates/base.html`

```html
<!DOCTYPE html>
<html lang="fr" class="h-full">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>
    {% block title %}{{ page_title|default:"MedFlow" }}{% endblock %} — MedFlow
  </title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script src="https://unpkg.com/htmx.org@1.9.12" defer></script>
  {% block extra_head %}{% endblock %}
</head>
<body class="h-full bg-slate-50"
      hx-headers='{"X-CSRFToken": "{{ csrf_token }}"}'>

{% if user.is_authenticated %}
<div class="flex h-full min-h-screen">

  <!-- ── Sidebar ─────────────────────────────────────────── -->
  <aside class="w-60 bg-slate-900 flex flex-col fixed h-full z-20 shadow-2xl">
    <div class="px-5 py-5 border-b border-slate-700/60">
      <div class="flex items-center gap-3">
        <div class="w-8 h-8 bg-blue-600 rounded-xl flex items-center justify-center">
          <span class="text-white text-sm">🩺</span>
        </div>
        <span class="text-white font-bold text-lg">MedFlow</span>
      </div>
    </div>

    <nav class="flex-1 px-3 py-4 space-y-1 overflow-y-auto">
      <p class="px-3 text-xs font-semibold text-slate-500 uppercase
                tracking-widest mb-2">Principal</p>

      <a href="{% url 'dashboard' %}"
         class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                text-slate-300 hover:bg-slate-700 hover:text-white
                text-sm font-medium transition-colors">
        🏠 Tableau de bord
      </a>

      <p class="px-3 text-xs font-semibold text-slate-500 uppercase
                tracking-widest mt-4 mb-2">Clinique</p>

      <!-- Ces liens seront activés aux sprints suivants -->
      <span class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                   text-slate-600 text-sm font-medium cursor-not-allowed">
        👤 Patients
        <span class="ml-auto text-xs font-mono bg-slate-800
                     text-slate-500 px-1.5 py-0.5 rounded">S1</span>
      </span>
      <span class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                   text-slate-600 text-sm font-medium cursor-not-allowed">
        🩺 Consultations
        <span class="ml-auto text-xs font-mono bg-slate-800
                     text-slate-500 px-1.5 py-0.5 rounded">S3</span>
      </span>
      <span class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                   text-slate-600 text-sm font-medium cursor-not-allowed">
        📅 Rendez-vous
        <span class="ml-auto text-xs font-mono bg-slate-800
                     text-slate-500 px-1.5 py-0.5 rounded">S4</span>
      </span>
      <span class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                   text-slate-600 text-sm font-medium cursor-not-allowed">
        🔗 Références
        <span class="ml-auto text-xs font-mono bg-slate-800
                     text-slate-500 px-1.5 py-0.5 rounded">S2</span>
      </span>
      <span class="flex items-center gap-3 px-3 py-2.5 rounded-lg
                   text-slate-600 text-sm font-medium cursor-not-allowed">
        📊 Statistiques
        <span class="ml-auto text-xs font-mono bg-slate-800
                     text-slate-500 px-1.5 py-0.5 rounded">S6</span>
      </span>
    </nav>

    <!-- User block -->
    <div class="px-4 py-4 border-t border-slate-700/60">
      <div class="flex items-center gap-3 mb-3">
        <div class="w-8 h-8 rounded-full bg-blue-600/20 border border-blue-500/30
                    flex items-center justify-center shrink-0">
          <span class="text-blue-400 font-semibold text-sm">
            {{ user.username|first|upper }}
          </span>
        </div>
        <span class="text-slate-300 text-sm truncate">{{ user.username }}</span>
      </div>
      <form method="post" action="{% url 'custom_logout' %}">
        {% csrf_token %}
        <button type="submit"
          class="w-full text-left text-slate-400 hover:text-white text-xs
                 px-2 py-1.5 rounded-lg hover:bg-slate-700 transition-colors">
          ↪ Déconnexion
        </button>
      </form>
    </div>
  </aside>

  <!-- ── Main area ────────────────────────────────────────── -->
  <div class="flex-1 flex flex-col ml-60 min-h-screen">
    <header class="bg-white border-b border-slate-200 px-8 py-4
                   sticky top-0 z-10 shadow-sm">
      <h1 class="font-semibold text-slate-800 text-lg">
        {% block page_heading %}{{ page_title|default:"MedFlow" }}{% endblock %}
      </h1>
    </header>

    <main class="flex-1 p-8">
      {% if messages %}
        {% for message in messages %}
        <div class="mb-4 px-4 py-3 rounded-lg text-sm
          {% if message.tags == 'success' %}
            bg-green-50 text-green-700 border border-green-200
          {% elif message.tags == 'error' %}
            bg-red-50 text-red-700 border border-red-200
          {% else %}
            bg-blue-50 text-blue-700 border border-blue-200
          {% endif %}">
          {{ message }}
        </div>
        {% endfor %}
      {% endif %}

      {% block content %}{% endblock %}
    </main>

    <footer class="px-8 py-3 border-t border-slate-200 bg-white">
      <p class="text-slate-400 text-xs font-mono">
        MedFlow v0.1 · Sprint 0 · SQLite · Django 5
      </p>
    </footer>
  </div>
</div>

{% else %}
  {% block auth_content %}{% endblock %}
{% endif %}

{% block extra_scripts %}{% endblock %}
</body>
</html>
```

### `templates/core/dashboard.html`

```html
{% extends "base.html" %}
{% block title %}Tableau de bord{% endblock %}
{% block page_heading %}Tableau de bord{% endblock %}

{% block content %}

<!-- KPI Cards -->
<div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-4 gap-5 mb-8">

  <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm
              hover:shadow-md transition-shadow">
    <p class="text-slate-500 text-sm mb-2">Patients actifs</p>
    <p class="text-3xl font-bold text-slate-800">
      {{ stats.patients_actifs }}
    </p>
  </div>

  <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm
              hover:shadow-md transition-shadow">
    <p class="text-slate-500 text-sm mb-2">Consultations aujourd'hui</p>
    <p class="text-3xl font-bold text-slate-800">
      {{ stats.consultations_jour }}
    </p>
  </div>

  <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm
              hover:shadow-md transition-shadow">
    <p class="text-slate-500 text-sm mb-2">RDV planifiés</p>
    <p class="text-3xl font-bold text-slate-800">
      {{ stats.rdv_semaine }}
    </p>
  </div>

  <div class="bg-white rounded-2xl border border-slate-200 p-5 shadow-sm
              hover:shadow-md transition-shadow">
    <p class="text-slate-500 text-sm mb-2">Références en attente</p>
    <p class="text-3xl font-bold text-slate-800">
      {{ stats.references_en_attente }}
    </p>
  </div>

</div>

<!-- Tables -->
<div class="grid grid-cols-1 lg:grid-cols-2 gap-6">

  <!-- Derniers patients -->
  <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm">
    <h2 class="font-semibold text-slate-800 mb-4">
      Derniers patients enregistrés
    </h2>
    <table class="w-full text-sm">
      <thead>
        <tr class="text-left text-slate-400 border-b border-slate-100">
          <th class="pb-2 font-medium">Nom</th>
          <th class="pb-2 font-medium">Sexe</th>
          <th class="pb-2 font-medium">Enregistré le</th>
        </tr>
      </thead>
      <tbody>
        {% for p in recent_patients %}
        <tr class="border-b border-slate-50 hover:bg-slate-50">
          <td class="py-2.5 font-medium">{{ p.full_name }}</td>
          <td class="py-2.5 text-slate-500">{{ p.get_sex_display }}</td>
          <td class="py-2.5 text-slate-400">
            {{ p.created_at|date:"d/m/Y" }}
          </td>
        </tr>
        {% empty %}
        <tr>
          <td colspan="3" class="py-6 text-center text-slate-400">
            Aucun patient enregistré.
          </td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
  </div>

  <!-- Prochains RDV -->
  <div class="bg-white rounded-2xl border border-slate-200 p-6 shadow-sm">
    <h2 class="font-semibold text-slate-800 mb-4">Prochains rendez-vous</h2>
    <table class="w-full text-sm">
      <thead>
        <tr class="text-left text-slate-400 border-b border-slate-100">
          <th class="pb-2 font-medium">Patient</th>
          <th class="pb-2 font-medium">Date</th>
          <th class="pb-2 font-medium">Statut</th>
        </tr>
      </thead>
      <tbody>
        {% for appt in upcoming_appointments %}
        <tr class="border-b border-slate-50 hover:bg-slate-50">
          <td class="py-2.5 font-medium">{{ appt.patient.full_name }}</td>
          <td class="py-2.5 text-slate-500">
            {{ appt.date_time|date:"d/m/Y H:i" }}
          </td>
          <td class="py-2.5 text-slate-400">{{ appt.get_status_display }}</td>
        </tr>
        {% empty %}
        <tr>
          <td colspan="3" class="py-6 text-center text-slate-400">
            Aucun rendez-vous à venir.
          </td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
  </div>

</div>
{% endblock %}
```

---

## 10. Running the Project

Lance ces commandes **dans l'ordre**, venv actif :

```powershell
# 1. Migrations
python manage.py makemigrations patients referrals consultations appointments history audit
python manage.py migrate

# 2. Superuser (le seeder en crée un aussi — admin / MedFlow2025!)
#    Passe cette étape si tu vas lancer le seeder juste après
python manage.py createsuperuser

# 3. Seeder — 100+ enregistrements
python seed_data.py
#    Le seeder crée automatiquement : admin / MedFlow2025!

# 4. Serveur de développement
python manage.py runserver
```

### URLs disponibles

| URL | Description |
|---|---|
| http://127.0.0.1:8000/ | Redirige vers login |
| http://127.0.0.1:8000/auth/login/ | Page de connexion |
| http://127.0.0.1:8000/dashboard/ | Dashboard Sprint 0 (login requis) |
| http://127.0.0.1:8000/admin/ | Interface admin — toutes les données seedées |

**Identifiants :** `admin` / `MedFlow2025!`

---

## 11. Passer à MySQL plus tard

Quand tu seras prêt à migrer de SQLite vers MySQL (Sprint 5 ou prod) :

### Étape 1 — Installer PyMySQL

```powershell
pip install pymysql
```

### Étape 2 — Créer la base MySQL

```sql
-- Exécuter en tant que root dans MySQL Workbench ou CLI
CREATE DATABASE IF NOT EXISTS medflow_db
  CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

CREATE USER IF NOT EXISTS 'medflow_user'@'127.0.0.1'
  IDENTIFIED BY 'MedFlow2025!';
CREATE USER IF NOT EXISTS 'medflow_user'@'localhost'
  IDENTIFIED BY 'MedFlow2025!';

GRANT ALL PRIVILEGES ON medflow_db.* TO 'medflow_user'@'127.0.0.1';
GRANT ALL PRIVILEGES ON medflow_db.* TO 'medflow_user'@'localhost';
GRANT ALL PRIVILEGES ON `test_medflow_db`.* TO 'medflow_user'@'127.0.0.1';
GRANT ALL PRIVILEGES ON `test_medflow_db`.* TO 'medflow_user'@'localhost';
FLUSH PRIVILEGES;
```

### Étape 3 — Modifier `.env`

```ini
DB_ENGINE=mysql
DB_NAME=medflow_db
DB_USER=medflow_user
DB_PASSWORD=MedFlow2025!
DB_HOST=127.0.0.1
DB_PORT=3306
```

### Étape 4 — Remigrer et re-seeder

```powershell
python manage.py migrate
python seed_data.py
```

> **Pas besoin de toucher à settings.py** — le bloc conditionnel
> `if DB_ENGINE == "mysql"` s'occupe de tout.

---

## 12. Troubleshooting

### `ModuleNotFoundError: No module named 'apps'`

`manage.py` ne trouve pas le dossier `apps`. Vérifie que :
- Tu lances la commande **depuis la racine du projet** (là où `manage.py` se trouve)
- Le fichier `apps/__init__.py` existe

```powershell
# Vérifie ta position
Get-Location
# Doit afficher : C:\My_Medflow_Django_Project
ls apps\__init__.py   # doit exister
```

### `No module named 'django_htmx'`

Le package n'est pas installé dans le venv actif.

```powershell
# Vérifie que le venv est actif (prompt commence par (venv))
pip install django-htmx
```

### `TemplateDoesNotExist: registration/login.html`

Le dossier `templates/` n'est pas dans `TEMPLATES[0]['DIRS']`.
Vérifie `settings.py` :

```python
# "DIRS": [BASE_DIR / "templates"],  # ← doit être présent
```

Vérifie aussi que le fichier existe :

```powershell
ls templates\registration\login.html
```

### `django.db.utils.OperationalError: no such table`

Les migrations n'ont pas été appliquées.

```powershell
python manage.py migrate
```

### `ImportError: attempted relative import with no known parent package`

Tu essaies de lancer un fichier app directement au lieu de passer par `manage.py`.
Toujours lancer depuis la racine avec `python manage.py <commande>`.

### `seed_data.py — IntegrityError` au deuxième lancement

Normal si tu relances sans effacer les données. Le seeder efface tout au début —
si l'erreur survient, c'est qu'une contrainte `unique` ou `OneToOne` a été violée
avant le nettoyage. Solution :

```powershell
# Efface la base SQLite et recommence proprement
del db.sqlite3
python manage.py migrate
python seed_data.py
```

### `Set-ExecutionPolicy` — erreur PowerShell sur Activate.ps1

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# Puis :
.\venv\Scripts\Activate.ps1
```

---

## 13. Architecture Overview

```
Domaine MedFlow
═══════════════════════════════════════════════

Patient  (entité centrale)
 ├── Allergies           (Allergy)
 ├── Timeline clinique   (PatientHistoryEntry)
 ├── Consultations
 │    ├── Prescriptions  (Prescription)
 │    └── Examens        (ExamRequest)
 ├── Rendez-vous         (Appointment)  ─── ↗ lien optionnel vers Consultation
 └── Références          (Referral)  ─── ↗ ReferralSource
      └── Récompense     (RewardLedger)

Entités de support
 ├── ConsultationSite    (lieu physique — lat/lng pour carte Folium Sprint 6)
 ├── Tag                 (étiquette patient)
 └── ReferralSource      (médecin / clinique / hôpital)

Transversal
 └── AuditLog            (piste d'audit immuable)

Roadmap des sprints
═══════════════════════════════════════════════

Sprint 0  ✅  Fondations (ce guide) — SQLite, modèles, seeder, dashboard
Sprint 1  ─   apps/patients  — CRUD patient, recherche, tags, documents
Sprint 2  ─   apps/referrals — Sources, Référence, RewardLedger
Sprint 3  ─   apps/consultations — Circuit consultation, Prescriptions, Examens
Sprint 4  ─   apps/appointments — Calendrier HTMX, Celery activé (Redis requis)
Sprint 5  ─   apps/history  — Timeline, Allergies, Alertes
Sprint 6  ─   apps/dashboard — Bokeh charts, carte Folium, KPI widgets
Sprint 7  ─   apps/audit    — RBAC, AuditLog UI, sauvegardes, durcissement sécurité

Migration vers MySQL/production
═══════════════════════════════════════════════
Voir Section 11 — Changer DB_ENGINE=mysql dans .env suffit.
PyMySQL remplace mysqlclient — pure Python, zéro compilation.
```

---

*Fin du guide Sprint 0 — Windows 11 Compatible.*
*Le serveur tourne, tous les modèles sont migrés, 100+ enregistrements chargés.*
*Sprint 1 — Gestion des patients — peut démarrer immédiatement.*
