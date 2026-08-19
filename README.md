# RentalCar — Global Rental Car Management

Marketplace multi-agences de location de voitures.
Projet tutoré — 3GINF-TA — École des Nouvelles Sciences & Ingénierie.

**Stack technique :** Laravel 12 (API REST) · Vue 3 (SPA) · PostgreSQL · Docker · Nginx

---

## 1. Architecture de l'environnement

L'environnement de développement est entièrement conteneurisé avec **Docker Compose**.
Il repose sur une architecture **découplée** : une API Laravel d'un côté, une SPA Vue de
l'autre, qui consomme l'API via Axios. Quatre services communiquent sur un réseau privé
(`grcm-net`) :

```
  Navigateur
      │
      ├──────────────► http://localhost:5173  ──►  [ frontend ]  Vue 3 + Vite
      │                                                  │  (appels API via Axios)
      └──────────────► http://localhost:8080  ──►  [ nginx ]  ──►  [ backend ]  Laravel 12 (PHP-FPM)
                                                                        │
                                                                        ▼
                                                                   [ db ]  PostgreSQL 16
```

| Service    | Rôle                                                        | Port hôte |
|------------|-------------------------------------------------------------|-----------|
| `frontend` | Interface utilisateur Vue 3 (serveur Vite, rechargement HMR)| 5173      |
| `nginx`    | Serveur web : expose l'API et relaie le PHP vers PHP-FPM     | 8080      |
| `backend`  | API REST Laravel 12 (PHP-FPM)                               | interne   |
| `db`       | Base de données PostgreSQL 16                               | 5433      |

> Les ports hôte (5433, 8080) ont été choisis pour éviter les conflits avec des services
> déjà présents sur la machine (un PostgreSQL local sur 5432, un autre service sur 8000).
> À l'intérieur du réseau Docker, les conteneurs communiquent toujours sur leurs ports
> standards (5432 pour la base, 80 pour Nginx).

---

## 2. Prérequis

À installer sur la machine avant de commencer :

- **Docker** ≥ 24 et **Docker Compose** v2 ([Docker Desktop](https://www.docker.com/products/docker-desktop/) sur Windows/Mac, ou `docker` + `docker compose` sur Linux)
- **Git**

> Aucune installation locale de PHP, Composer, Node ou PostgreSQL n'est nécessaire :
> tout tourne dans les conteneurs.

---

## 3. Structure du dépôt

```
RentalCar/
├── backend/               # Application Laravel 12 (API REST)
│   └── Dockerfile
├── frontend/              # Application Vue 3 (SPA)
│   └── Dockerfile
├── docker/
│   └── nginx/
│       └── default.conf   # Configuration du serveur web
├── docker-compose.yml     # Orchestration des 4 services
├── .env.example           # Modèle des variables d'environnement Docker
└── README.md
```

---

## 4. Premier démarrage

### 4.1 Cloner le dépôt

```bash
git clone https://github.com/omardoukkali/RentalCar.git
cd RentalCar
```

### 4.2 Configurer les variables d'environnement

```bash
# Variables Docker (base de données, ports)
cp .env.example .env

# Variables Laravel
cp backend/.env.example backend/.env
```

Dans `backend/.env`, vérifier que la connexion pointe vers le conteneur `db` :

```env
DB_CONNECTION=pgsql
DB_HOST=db
DB_PORT=5432
DB_DATABASE=grcm
DB_USERNAME=grcm
DB_PASSWORD=secret
```

> Remarque : `DB_HOST=db` et `DB_PORT=5432` désignent le conteneur PostgreSQL vu depuis
> l'intérieur du réseau Docker. Le port hôte (5433 dans `.env`) ne sert que pour se
> connecter à la base depuis un outil externe (DBeaver, pgAdmin).

### 4.3 Construire et lancer les conteneurs

```bash
docker compose up -d --build
```

### 4.4 Installer les dépendances et initialiser la base

```bash
# Backend : dépendances PHP, clé d'application, migrations
docker compose exec backend composer install
docker compose exec backend php artisan key:generate
docker compose exec backend php artisan migrate

# Frontend : les dépendances s'installent automatiquement au démarrage du conteneur
```

### 4.5 Accéder à l'application

| Ressource            | URL                            |
|----------------------|--------------------------------|
| Interface (Vue)      | http://localhost:5173          |
| API (Laravel)        | http://localhost:8080          |
| Base PostgreSQL      | localhost:5433 (base `grcm`)   |

---

## 5. Commandes utiles

```bash
# Voir l'état des conteneurs
docker compose ps

# Suivre les logs d'un service (en temps réel)
docker compose logs -f backend
docker compose logs -f frontend

# Ouvrir un shell dans un conteneur
docker compose exec backend bash
docker compose exec frontend sh

# Commandes Artisan (Laravel)
docker compose exec backend php artisan migrate:fresh --seed
docker compose exec backend php artisan route:list

# Arrêter / relancer
docker compose down          # arrête et supprime les conteneurs
docker compose down -v       # + supprime le volume de la base (remise à zéro)
```

---

## 6. Stratégie de gestion des branches (Git Flow)

Le projet suit le modèle **Git Flow**. Voir le document dédié *Stratégie de gestion des branches* pour la justification complète.

| Branche      | Rôle                                                                      |
|--------------|---------------------------------------------------------------------------|
| `main`       | Production. Uniquement du code stable et validé. Aucun push direct.       |
| `develop`    | Intégration des développements en cours. Base des branches de fonctionnalité. |
| `feature/*`  | Une branche par fonctionnalité, créée depuis `develop`.                   |
| `hotfix/*`   | Correction urgente, créée depuis `main`, fusionnée dans `main` **et** `develop`. |

`main` et `develop` sont **protégées** : intégration uniquement par Pull Request, après revue de code.

### Conventions

- **Branches :** minuscules, mots séparés par des tirets, préfixe explicite.
  Exemple : `feature/reservation-voiture`, `hotfix/correction-paiement`.
- **Commits :** convention *Conventional Commits* précédée de la clé Jira.
  Exemple : `GRC-42 feat: validation des dates de réservation`.
  Préfixes : `feat`, `fix`, `docs`, `refactor`, `test`, `chore`.

---

## 7. Jira & traçabilité

Chaque développement est associé à une tâche **Jira**. Le dépôt GitHub est intégré à Jira :
en citant la clé de la tâche dans les commits et les Pull Requests (ex. `GRC-42`), les
commits et PR remontent automatiquement dans la tâche correspondante, assurant la
traçabilité **tâche ↔ commit ↔ Pull Request**.

---

## 8. Dépannage

| Problème                                     | Solution                                                                 |
|----------------------------------------------|--------------------------------------------------------------------------|
| `port is already allocated` (5432/8000/5173) | Modifier le port correspondant dans `.env` (ex. `DB_PORT=5433`, `API_PORT=8080`). |
| `SQLSTATE ... could not connect`             | Attendre que `db` soit « healthy » (`docker compose ps`), puis relancer les migrations. |
| Frontend : `sh: vite: not found` en boucle   | Les dépendances s'installent au démarrage du conteneur ; laisser Vite finir l'installation (`docker compose logs -f frontend`). |
| `Composer detected ... require PHP >= 8.4`   | Aligner l'image du backend sur `php:8.4-fpm-alpine` puis `docker compose up -d --build backend`. |
| `Please provide a valid cache path` (Laravel)| `docker compose exec backend php artisan config:clear`.                  |
