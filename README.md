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

### 2. Installer la base de données MySQL

Assure-toi d'avoir MySQL installé et en cours d'exécution, puis crée la base et importe le dump :

```bash
mysql -u root -p -e "CREATE DATABASE finr;"
mysql -u root -p finr < finr.sql
```

Ensuite, configure le `.env` de Laravel pour pointer vers MySQL :

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=finr
DB_USERNAME=root
DB_PASSWORD=ton_mot_de_passe
```

---

### 3. Installer les dépendances (une seule fois)

**Backend Laravel (`finr-api`) :**
```bash
cd finr-api
composer install
cp .env.example .env
# Remplis DB_DATABASE, DB_USERNAME, DB_PASSWORD dans .env
php artisan key:generate
php artisan jwt:secret
```

**Microservice NLP (`finr-nlp`) :**
```bash
cd finr-nlp
python -m venv venv

# Windows
venv\Scripts\activate
# macOS/Linux
source venv/bin/activate

pip install -r requirements.txt
```

**Frontend React (`finr-app`) :**
```bash
cd finr-app
cp .env.example .env
npm install
```

---

### 3. Lancer l'application (3 terminaux)

Ouvre **3 fenêtres de terminal** et lance chaque service :

**Terminal 1 — API Laravel :**
```bash
cd FIN-R/finr-api
php artisan serve
```

**Terminal 2 — Microservice NLP :**
```bash
cd FIN-R/finr-nlp
python -m uvicorn app.main:app --port 8001 --reload
```

**Terminal 3 — Frontend React :**
```bash
cd FIN-R/finr-app
npm start
```

---

### 4. Configuration `.env` du frontend

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

- Pour se Connecter en tant que Chercheur:
    login : admin@finr.com
    mot de passe : Admin@1234

- Pour se connecter en tant que Ingenieur:
    login : ingenieur@finr.com
    mot de passe : passer@1234

