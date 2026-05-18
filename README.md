<div align="center">

# 🗺️ CollectGraph

### *Optimisation de tournées de collecte de déchets par théorie des graphes*

**Application web interactive permettant de modéliser un réseau de collecte sous forme de graphe, de calculer les trajets optimaux entre points (Dijkstra) et d'assigner des équipes par zones via coloration de graphe.**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python](https://img.shields.io/badge/Python-3.8+-blue.svg?logo=python&logoColor=white)](https://www.python.org/)
[![Flask](https://img.shields.io/badge/Flask-2.0+-000000?logo=flask&logoColor=white)](https://flask.palletsprojects.com/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-12+-336791?logo=postgresql&logoColor=white)](https://www.postgresql.org/)
[![JavaScript](https://img.shields.io/badge/JavaScript-Canvas%20API-F7DF1E?logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API)

[**🧮 Algorithmes**](#-algorithmes-implémentés) · [**🚀 Installation**](#-installation) · [**🔌 API REST**](#-api-rest)

</div>

---

## 📋 Sommaire

- [À propos](#-à-propos)
- [Fonctionnalités](#-fonctionnalités)
- [Installation](#-installation)
- [Utilisation](#-guide-dutilisation)
- [API REST](#-api-rest)
- [Algorithmes](#-algorithmes-implémentés)
- [Technologies](#%EF%B8%8F-technologies-utilisées)
- [Structure du projet](#-structure-du-projet)
- [Auteur](#-auteur)
- [Licence](#-licence)

---

## 🎯 À propos

**CollectGraph** est un projet académique de **théorie des graphes appliquée** à un problème concret : l'optimisation des tournées de collecte de déchets en milieu urbain. L'application combine la **modélisation visuelle** d'un réseau de routes (graphe pondéré) avec deux algorithmes classiques :

- 🛣️ **Dijkstra** pour trouver le **chemin le plus court** entre deux points
- 🎨 **Coloration gloutonne** pour **répartir les zones** de collecte entre équipes ou jours de la semaine

L'interface est entièrement **interactive** : on dessine le graphe à la souris sur un Canvas HTML5, on visualise les résultats en temps réel, et les données sont **persistées** dans une base PostgreSQL.

---

## ✨ Fonctionnalités

- 🗺️ **Visualisation interactive** du graphe sur Canvas HTML5
- ✏️ **Édition dynamique** : ajout/suppression de nœuds et arêtes à la souris
- ⚖️ **Arêtes pondérées** (distance/temps)
- 🛣️ **Algorithme de Dijkstra** : chemin optimal entre deux nœuds
- 🎨 **Coloration de graphe** : assignation automatique d'équipes/jours
- 💾 **Persistance PostgreSQL** : les modifications sont sauvegardées
- 🔄 **API REST complète** : intégration possible avec d'autres systèmes

---

## 🚀 Installation

### Prérequis

- **Python 3.8** ou supérieur
- **PostgreSQL 12** ou supérieur
- pip

### 1. Cloner le repo

```bash
git clone https://github.com/SALLAH-JP/TG.git
cd TG
```

### 2. Environnement virtuel Python

```bash
python -m venv venv
# Windows
venv\Scripts\activate
# Linux / macOS
source venv/bin/activate
```

### 3. Dépendances Python

```bash
pip install flask flask-cors psycopg2-binary
```

### 4. Configurer PostgreSQL

Créer la base et importer le schéma :

```bash
# Créer la base
createdb TG

# Importer le schéma (à la racine du repo)
psql -d TG -f TG.sql
```

Schéma simplifié :

```sql
CREATE TABLE nodes (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    x FLOAT NOT NULL,
    y FLOAT NOT NULL
);

CREATE TABLE edges (
    id SERIAL PRIMARY KEY,
    from_node VARCHAR(50) NOT NULL,
    to_node VARCHAR(50) NOT NULL,
    weight INT NOT NULL,
    undirected BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (from_node) REFERENCES nodes(name) ON DELETE CASCADE,
    FOREIGN KEY (to_node) REFERENCES nodes(name) ON DELETE CASCADE
);
```

### 5. Configurer la connexion

Modifier les identifiants PostgreSQL dans `server.py` :

```python
def get_connection():
    return psycopg2.connect(
        dbname="TG",
        user="postgres",
        password="VOTRE_MOT_DE_PASSE",
        host="localhost",
        port=5432
    )
```

### 6. Lancer l'application

```bash
python server.py
```

L'application est accessible sur **http://localhost:5000**

---

## 🎮 Guide d'utilisation

### Interface principale

- **Canvas central** : visualisation du graphe (nœuds + arêtes pondérées)
- **Barre d'outils** : sélection de l'action en cours, choix source/destination
- **Panneau latéral** : légende des équipes et résultats des calculs

### Actions disponibles

#### ➕ Ajouter un nœud
1. Sélectionner *« Ajouter un nœud »* dans le menu
2. Cliquer **OK**
3. Cliquer sur le canvas pour le placer
4. Entrer un nom (auto-incrémenté si vide) puis **Entrée**

#### ❌ Supprimer un nœud
1. Sélectionner *« Supprimer un nœud »* → **OK**
2. Cliquer sur le nœud à supprimer
3. Les arêtes liées sont supprimées en cascade

#### ↔️ Ajouter une arête
1. Sélectionner *« Ajouter une arête »* → **OK**
2. Cliquer sur le nœud source
3. Cliquer sur le nœud destination
4. Entrer le poids (distance/temps) → **Entrée**

#### 🚮 Supprimer une arête
1. Sélectionner *« Supprimer une arête »* → **OK**
2. Cliquer sur l'arête

#### 🛣️ Chercher le chemin optimal
1. Choisir la source et la destination dans les menus déroulants
2. Cliquer **Rechercher (Dijkstra)**
3. Le chemin est surligné sur le canvas et la distance affichée

#### 🎨 Assigner les équipes
1. Cliquer **Colorier (jours/équipes)**
2. Les nœuds sont automatiquement colorés selon leur assignation
3. La légende affiche la correspondance couleur ↔ équipe

---

## 🔌 API REST

### GET `/graph`

Retourne le graphe complet.

```json
{
  "nodes": [
    {"name": "N1", "x": 100, "y": 150},
    {"name": "N2", "x": 300, "y": 250}
  ],
  "edges": [
    {"from": "N1", "to": "N2", "weight": 50, "undirected": false}
  ]
}
```

### POST `/graph/node`

Ajoute un nœud.

```json
{ "name": "N3", "x": 400, "y": 300 }
```

### DELETE `/graph/node`

Supprime un nœud (et ses arêtes en cascade).

```json
{ "name": "N3" }
```

### POST `/graph/edge`

Ajoute une arête pondérée.

```json
{ "from": "N1", "to": "N2", "weight": 50 }
```

### DELETE `/graph/edge`

Supprime une arête.

```json
{ "from": "N1", "to": "N2" }
```

### GET `/algo/dijkstra?src=N1&dst=N2`

Calcule le plus court chemin.

```json
{
  "path": ["N1", "N3", "N2"],
  "distance": 150
}
```

### GET `/algo/coloring`

Coloration du graphe (assignation d'équipes).

```json
{
  "N1": 1,
  "N2": 2,
  "N3": 1
}
```

---

## 🧮 Algorithmes implémentés

### 🛣️ Dijkstra

Calcule le **plus court chemin** entre deux nœuds dans un graphe pondéré non-négatif.

- **Complexité** : *O((V + E) log V)* avec file de priorité
- **Implémentation** : `logic.py`
- **Usage** : optimisation des trajets de collecte entre deux points

### 🎨 Coloration de graphe (gloutonne)

Assigne des couleurs (= équipes ou jours) aux nœuds de façon à ce que **deux nœuds adjacents n'aient jamais la même couleur**.

- **Stratégie** : tri des nœuds par **degré décroissant** puis affectation gloutonne
- **Objectif** : minimiser le nombre d'équipes/jours nécessaires pour couvrir toute la ville
- **Application** : planification hebdomadaire des tournées

---

## 🛠️ Technologies utilisées

### Backend
- **Flask** — micro-framework web Python
- **Flask-CORS** — gestion des requêtes cross-origin
- **psycopg2** — driver PostgreSQL

### Frontend
- **HTML5 / CSS3** — structure et styles
- **Canvas API** — rendu 2D interactif du graphe
- **JavaScript vanilla** — logique frontend, pas de framework
- **Fetch API** — communication avec le backend

### Base de données
- **PostgreSQL** — stockage des nœuds, arêtes et métadonnées

---

## 📁 Structure du projet

```
.
├── server.py              # Backend Flask + routes API
├── logic.py               # Algorithmes (Dijkstra, coloration)
├── TG.sql                 # Schéma SQL de la base
├── public/                # Frontend
│   ├── index.html         # Page principale
│   ├── app.js             # Logique Canvas + appels API
│   └── style.css          # Styles
├── projet final tg.docx   # Rapport du projet
└── LICENSE
```

---

## 📊 Exemple de cas d'usage

**Scénario** : Optimiser la collecte des déchets dans une ville de 30 quartiers.

1. **Modélisation** — Créer un nœud par quartier, des arêtes pour chaque route (poids = distance ou temps de parcours)
2. **Répartition** — Lancer la coloration : la ville est divisée en zones distinctes, chacune attribuée à une équipe différente
3. **Optimisation** — Pour chaque zone, utiliser Dijkstra pour planifier le trajet le plus rapide entre le dépôt et chaque point
4. **Persistance** — Le réseau est sauvegardé en base, modifiable en temps réel par les opérateurs

---

## 🐛 Dépannage

**`psycopg2.OperationalError: could not connect to server`**
→ PostgreSQL n'est pas démarré, ou les identifiants dans `server.py` sont incorrects.

**Canvas vide au démarrage**
→ La base est vide. Ajouter des nœuds manuellement via l'interface, ou importer un jeu de données initial dans `TG.sql`.

**Arête impossible à ajouter**
→ Vérifier que les deux nœuds existent et que l'arête n'existe pas déjà entre eux.

---

## 🔒 Sécurité

- **Requêtes paramétrées** : protection contre les injections SQL
- **CORS** configuré pour limiter les origines autorisées
- Gestion des erreurs côté serveur

---

## 📚 Contexte académique

Ce projet a été réalisé dans le cadre du cours de **Théorie des Graphes** de la **Licence Informatique Appliquée** à l'Université des Mascareignes. Le rapport complet est disponible dans le dépôt (`projet final tg.docx`).

---

## 👤 Auteur

**SALLAH Assiongbon Théodore Jean-Paul**
Étudiant en 3ème année — Licence Informatique Appliquée
🎓 Université des Mascareignes (Maurice)

[![GitHub](https://img.shields.io/badge/GitHub-SALLAH--JP-181717?logo=github)](https://github.com/SALLAH-JP)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Jean--Paul%20SALLAH-0A66C2?logo=linkedin)](https://www.linkedin.com/in/jeanpaul-sallah/)

---

## 📜 Licence

Ce projet est distribué sous licence **MIT** — voir le fichier [`LICENSE`](LICENSE).

---

<div align="center">

*Si ce projet vous a plu, n'hésitez pas à laisser une ⭐ !*

</div>
