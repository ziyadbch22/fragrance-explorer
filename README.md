# 🌸 Fragrance Explorer

> Application full-stack (MERN) de découverte et de comparaison de parfums, avec algorithme de similarité olfactive et authentification sécurisée.

---

## Fonctionnalités

| Fonctionnalité | Description |
|---|---|
| **Exploration & recherche** | Catalogue de parfums filtrable par famille olfactive, genre, marque |
| **Fiche détaillée** | Pyramide olfactive (notes de tête / cœur / fond), longévité, sillage, intensité |
| **Comparateur** | Mise en parallèle de plusieurs parfums côte à côte |
| **Recommandations** | Parfums similaires par notes, par humeur (*mood*), ou par profil olfactif |
| **Quiz de profil** | Détermine le profil olfactif de l'utilisateur (famille et intensité préférées) |
| **Favoris & historique** | Sauvegarde de parfums favoris et historique de recherche par utilisateur |
| **Auth sécurisée** | Inscription / connexion par JWT, mots de passe hashés (bcrypt, salt 12 rounds) |

---

## Algorithme de similarité

Le cœur du projet : un moteur de similarité pondéré (`backend/utils/similarity.js`), pas un simple filtre par tags.

Il combine plusieurs signaux pour calculer un score de 0 à 100 entre deux parfums :

- **Similarité de Jaccard** sur les notes de tête, cœur et fond (avec une pondération plus forte pour les notes de cœur, qui définissent le caractère du parfum)
- **Bonus de famille olfactive** si les deux parfums partagent la même famille (florale, boisée, orientale...)
- **Proximité d'intensité** (échelle 1–10)
- **Recoupement d'ambiance** (*mood* : sexy, frais, élégant...)
- **Bonus de genre** (compatible si même genre ou l'un des deux est unisexe)

```
score = (similarité_notes × 0.60) + bonus_famille + score_intensité + score_mood + bonus_genre
```

---

## Stack technique

**Frontend** : React 18, React Router, Tailwind CSS, Framer Motion, Vite, Axios
**Backend** : Node.js, Express, MongoDB (Mongoose), JWT, bcrypt
**Source de données** : API externe [Fragella](https://www.fragella.com/) (catalogue de parfums), avec une couche de mapping (`fragellaService.js`) qui normalise le format externe vers le modèle interne de l'app

---

## Architecture

```
fragrance-explorer/
├── backend/
│   ├── server.js               # Point d'entrée Express + connexion MongoDB
│   ├── models/
│   │   ├── Fragrance.js        # Schéma parfum (notes, famille, longévité, sillage...)
│   │   └── User.js             # Schéma utilisateur (favoris, historique, profil olfactif)
│   ├── controllers/            # Logique métier (fragrances, recommandations, utilisateurs)
│   ├── routes/                 # Endpoints REST (/api/fragrances, /api/users, /api/recommendations)
│   ├── middleware/auth.js      # Protection des routes par JWT (protect / optionalAuth)
│   ├── services/fragellaService.js  # Wrapper + mapping de l'API Fragella
│   ├── utils/similarity.js     # Algorithme de similarité (Jaccard pondéré)
│   └── data/seed.js            # Jeu de données de démonstration
└── frontend/
    └── src/
        ├── pages/              # Home, Explore, FragranceDetail, Compare, MoodPage, ProfileQuiz, Login, Register
        ├── components/         # FragranceCard, LongevityGraph, Navbar
        ├── context/            # AuthContext, FavoritesContext
        └── services/api.js     # Client Axios vers l'API backend
```

---

## API — principaux endpoints

| Méthode | Route | Description |
|---|---|---|
| `GET` | `/api/fragrances` | Liste / recherche de parfums |
| `GET` | `/api/fragrances/:id/similar` | Parfums similaires à un parfum donné |
| `POST` | `/api/fragrances/compare` | Comparaison de plusieurs parfums |
| `GET` | `/api/recommendations/mood/:mood` | Recommandations par ambiance |
| `POST` | `/api/recommendations/profile` | Recommandations basées sur le profil olfactif |
| `POST` | `/api/users/register` / `/login` | Authentification (retourne un JWT) |
| `GET` | `/api/users/me` | Profil de l'utilisateur connecté (protégé) |
| `POST` | `/api/users/favorites/:fragranceId` | Ajouter/retirer un favori (protégé) |

---

## Installation

### Prérequis
- Node.js 18+
- MongoDB (local ou Atlas)
- Une clé API [Fragella](https://www.fragella.com/) *(optionnel si vous utilisez `seed.js` pour des données de démo locales)*

### Lancer le projet

```bash
# 1. Cloner le repo
git clone https://github.com/ziyadbch22/fragrance-explorer.git
cd fragrance-explorer

# 2. Backend
cd backend
npm install
cp .env.example .env    # puis renseigner MONGODB_URI, JWT_SECRET, FRAGELLA_API_KEY
npm run seed             # (optionnel) injecte un jeu de données de démo
npm run dev               # démarre le serveur sur http://localhost:5000

# 3. Frontend (dans un autre terminal)
cd frontend
npm install
npm run dev               # démarre l'app sur http://localhost:5173
```

---

## Ce que j'ai appris sur ce projet

- Concevoir un **algorithme de scoring multi-critères** plutôt qu'un simple filtre (pondération, normalisation, gestion des cas limites comme deux listes de notes vides)
- Sécuriser une authentification de bout en bout : hash + salt bcrypt, signature/vérification JWT, middleware de protection de routes
- Consommer et **normaliser une API externe** hétérogène (mapping de vocabulaire, valeurs manquantes, formats incohérents) vers un modèle de données interne propre
- Structurer un projet full-stack MERN de façon modulaire (séparation routes / controllers / services / models)

---

