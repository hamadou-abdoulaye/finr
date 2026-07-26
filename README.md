# FIN-R — Plateforme d'analyse du raisonnement en STEAM

FIN-R est une plateforme web complète permettant d'analyser et d'évaluer le raisonnement cognitif d'ingénieurs lors de sessions de travail en STEAM (Science, Technologie, Engineering, Arts, Mathématiques).

Elle combine une API Laravel, une interface React et un microservice NLP Python pour détecter en temps réel les types de raisonnement, les événements cognitifs clés et calculer un score de créativité automatique.

---

## Architecture

```
FIN-R/
├── finr-api/        # Backend Laravel 12 (API REST + JWT)
├── finr-app/        # Frontend React 19 + TypeScript
└── finr-nlp/        # Microservice NLP Python (FastAPI)
```

### Stack technique

| Couche      | Technologie                              |
|-------------|------------------------------------------|
| Backend     | PHP 8.2, Laravel 12, JWT (tymon/jwt-auth)|
| Frontend    | React 19, TypeScript, React Router 7     |
| NLP         | Python 3.11+, FastAPI, scikit-learn      |
| Base de données | SQLite (dev) / MySQL (prod)          |
| Temps réel  | Laravel Echo + Pusher                    |
| PDF         | barryvdh/laravel-dompdf                  |

---

## Structure détaillée

### `finr-api/` — Backend Laravel

```
finr-api/
├── app/
│   ├── Http/Controllers/
│   │   ├── AuthController.php       # Inscription, login, logout, refresh JWT
│   │   ├── EngineerController.php   # CRUD ingénieurs
│   │   └── SessionController.php   # Gestion sessions + export PDF
│   ├── Models/
│   │   ├── User.php                 # Utilisateur (researcher | engineer)
│   │   ├── Engineer.php             # Profil ingénieur
│   │   ├── Session.php              # Session de travail
│   │   ├── SessionEvent.php         # Événements cognitifs détectés
│   │   └── ReasoningScore.php       # Scores de raisonnement
│   ├── Services/
│   │   └── NlpService.php           # Client HTTP vers finr-nlp
│   └── Events/
│       └── SessionUpdated.php       # Événement broadcast temps réel
├── routes/
│   └── api.php                      # Toutes les routes API
└── database/migrations/             # Migrations SQLite/MySQL
```

**Rôles utilisateurs :**
- `researcher` — accès complet (dashboard, ingénieurs, sessions, stats)
- `engineer` — accès limité à ses propres sessions (espace de travail)

**Endpoints principaux :**

| Méthode | Route | Description |
|---------|-------|-------------|
| POST | `/api/auth/register` | Inscription |
| POST | `/api/auth/login` | Connexion (retourne JWT) |
| GET | `/api/engineers` | Liste des ingénieurs (researcher) |
| GET | `/api/sessions` | Liste des sessions (researcher) |
| POST | `/api/sessions` | Créer une session |
| POST | `/api/sessions/{id}/start` | Démarrer une session |
| POST | `/api/sessions/{id}/end` | Terminer une session |
| GET | `/api/sessions/{id}/pdf` | Exporter le rapport PDF |
| GET | `/api/sessions/stats/global` | Statistiques globales |

---

### `finr-app/` — Frontend React

```
finr-app/src/
├── pages/
│   ├── Login.tsx           # Authentification (login + inscription)
│   ├── Dashboard.tsx       # Vue d'ensemble chercheur
│   ├── Sessions.tsx        # Liste des sessions
│   ├── SessionDetail.tsx   # Détail d'une session
│   ├── Workspace.tsx       # Espace de travail ingénieur (saisie notes)
│   ├── Engineers.tsx       # Gestion des ingénieurs
│   ├── Stats.tsx           # Statistiques et graphiques
│   ├── Reports.tsx         # Rapports exportables
│   ├── Reasoning.tsx       # Visualisation du raisonnement
│   └── NewSession.tsx      # Création de session
├── components/
│   ├── layout/             # AppLayout, Sidebar, ProtectedRoute
│   ├── dashboard/          # MetricCard, SessionList
│   └── shared/             # Pill, ReasoningBars
├── hooks/
│   ├── useSessions.ts      # Fetch liste sessions
│   ├── useSession.ts       # Fetch session unique
│   └── useEngineers.ts     # Fetch ingénieurs
├── context/
│   └── AuthContext.tsx     # Contexte d'authentification JWT
└── lib/
    ├── api.ts              # Client Axios configuré
    └── echo.ts             # Configuration Laravel Echo (temps réel)
```

---

### `finr-nlp/` — Microservice NLP

```
finr-nlp/
├── app/
│   ├── main.py             # API FastAPI (endpoints /analyze, /events, /health)
│   ├── classifier.py       # Classification du type de raisonnement
│   └── event_detector.py   # Détection d'événements cognitifs
├── training_data.py        # Données d'entraînement du modèle
└── tests/
    └── test_nlp.py         # Tests unitaires
```

**Endpoints NLP :**

| Méthode | Route | Description |
|---------|-------|-------------|
| GET | `/health` | Statut du service |
| POST | `/analyze` | Analyse complète du texte (scores + créativité) |
| POST | `/events` | Détection d'événements cognitifs en temps réel |
| DELETE | `/sessions/{id}` | Libérer l'état d'une session terminée |

---

## Prérequis

Avant de commencer, assure-toi d'avoir installé :

- [PHP 8.2+](https://www.php.net/downloads) + [Composer](https://getcomposer.org/)
- [Node.js 18+](https://nodejs.org/) + npm
- [Python 3.11+](https://www.python.org/downloads/)
- [Git](https://git-scm.com/)

---

## Installation & lancement en local

### 1. Cloner le projet

```bash
git clone https://github.com/hamadou-abdoulaye/finr.git
cd finr
```

---

### 2. Backend Laravel (`finr-api`)

```bash
cd finr-api

# Installer les dépendances PHP
composer install

# Copier et configurer l'environnement
cp .env.example .env

# Générer la clé d'application
php artisan key:generate

# Générer la clé JWT
php artisan jwt:secret

# Créer la base de données et exécuter les migrations
touch database/database.sqlite
php artisan migrate

# Lancer le serveur (port 8000)
php artisan serve
```

> L'API sera disponible sur **http://localhost:8000/api**

---

### 3. Microservice NLP (`finr-nlp`)

```bash
cd finr-nlp

# Créer un environnement virtuel Python
python -m venv venv

# Activer l'environnement (Windows)
venv\Scripts\activate

# Activer l'environnement (macOS/Linux)
source venv/bin/activate

# Installer les dépendances
pip install -r requirements.txt

# Lancer le service (port 8001)
uvicorn app.main:app --host 0.0.0.0 --port 8001 --reload
```

> Le service NLP sera disponible sur **http://localhost:8001**
> Documentation interactive : **http://localhost:8001/docs**

---

### 4. Frontend React (`finr-app`)

```bash
cd finr-app

# Installer les dépendances
npm install

# Copier et configurer l'environnement
cp .env.example .env

# Lancer l'application (port 3000)
npm start
```

> L'application sera disponible sur **http://localhost:3000**

---

### 5. Configuration `.env` du frontend

Ouvre `finr-app/.env` et vérifie ces valeurs :

```env
REACT_APP_API_URL=http://localhost:8000/api
REACT_APP_REVERB_KEY=finr-key
REACT_APP_REVERB_HOST=localhost
REACT_APP_REVERB_PORT=8080
```

---

## Récapitulatif des ports

| Service | URL |
|---------|-----|
| Frontend React | http://localhost:3000 |
| API Laravel | http://localhost:8000 |
| Microservice NLP | http://localhost:8001 |

---

## Comptes de test

Après les migrations, tu peux créer un compte via l'interface de login ou directement via l'API :

```bash
curl -X POST http://localhost:8000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Chercheur Test",
    "email": "chercheur@test.com",
    "password": "password123",
    "password_confirmation": "password123",
    "role": "researcher"
  }'
```

---

## Développé par

**ISFAD - RDI**  
Plateforme d'analyse du raisonnement en STEAM
