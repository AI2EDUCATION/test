# 📚 Guide de Déploiement — Concours M2 DS & IA
# AI2 — Applied Institute of Artificial Intelligence

## 📁 Fichiers livrés

| Fichier | Description |
|---------|-------------|
| `concours-m2.html` | Test d'admission (page publique candidats) |
| `admin-dashboard.html` | Tableau de bord résultats (accès admin) |

---

## 🗄️ ÉTAPE 1 — Configurer Supabase (base de données)

### 1.1 Créer le projet Supabase
1. Aller sur **https://supabase.com** → "Start your project" → connexion avec GitHub/Google
2. Cliquer **"New project"**
3. Nommer le projet : `ai2-concours-m2`
4. Choisir une région (ex: `West EU - Ireland`)
5. Définir un mot de passe base de données fort → **NOTEZ-LE**
6. Attendre ~2 min que le projet soit prêt

### 1.2 Créer la table `candidates`

Dans votre projet Supabase → **SQL Editor** → Exécuter ce script :

```sql
-- Activer l'extension UUID
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Table principale des candidats
CREATE TABLE candidates (
  id            UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  created_at    TIMESTAMPTZ DEFAULT NOW(),
  nom           TEXT NOT NULL,
  prenom        TEXT NOT NULL,
  email         TEXT NOT NULL,
  phone         TEXT,
  universite    TEXT,
  mention_m1    TEXT,
  specialite    TEXT,
  score_math    INTEGER DEFAULT 0,
  score_ml      INTEGER DEFAULT 0,
  score_dl      INTEGER DEFAULT 0,
  score_python  INTEGER DEFAULT 0,
  score_sql     INTEGER DEFAULT 0,
  total_score   INTEGER DEFAULT 0,
  max_score     INTEGER DEFAULT 30,
  percentage    NUMERIC(5,2) DEFAULT 0,
  eligibility   TEXT,
  answers       JSONB,
  time_taken    INTEGER
);

-- Index pour les recherches fréquentes
CREATE INDEX idx_candidates_email ON candidates(email);
CREATE INDEX idx_candidates_pct ON candidates(percentage DESC);
CREATE INDEX idx_candidates_created ON candidates(created_at DESC);

-- ═══════════════════════════════════════
-- Sécurité Row Level Security (RLS)
-- ═══════════════════════════════════════
ALTER TABLE candidates ENABLE ROW LEVEL SECURITY;

-- Autoriser les candidats (anon) à insérer leur résultat
CREATE POLICY "Candidats peuvent insérer"
  ON candidates FOR INSERT
  TO anon
  WITH CHECK (true);

-- Interdire la lecture aux utilisateurs anonymes
-- (seul le service_role key peut lire — pour le dashboard admin)
-- Note : pas de SELECT policy pour anon = lecture impossible sans service_role
```

### 1.3 Récupérer vos clés API

Supabase → **Settings** → **API** :

| Clé | Usage | Où configurer |
|-----|-------|---------------|
| **Project URL** | Les deux fichiers | `SUPABASE_URL` |
| **anon / public key** | `concours-m2.html` uniquement | `SUPABASE_ANON_KEY` |
| **service_role key** ⚠️ SECRET | `admin-dashboard.html` uniquement | `SUPABASE_SERVICE_KEY` |

> ⚠️ **La service_role key ne doit JAMAIS être exposée publiquement !**
> Elle est dans `admin-dashboard.html` qui doit rester en accès restreint.

---

## ⚙️ ÉTAPE 2 — Configurer les fichiers HTML

### Dans `concours-m2.html` (lignes ~720–725)

```javascript
const SUPABASE_URL = "https://XXXXXXXXXXX.supabase.co";  // ← votre URL
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR...";  // ← anon key
```

### Dans `admin-dashboard.html` (lignes ~480–485)

```javascript
const SUPABASE_URL = "https://XXXXXXXXXXX.supabase.co";   // ← même URL
const SUPABASE_SERVICE_KEY = "eyJhbGciOiJIUzI1NiIsInR..."; // ← service_role key
const ADMIN_PASSWORD = "VotreMotDePasseAdmin2025!";          // ← changez !
```

---

## 🌐 ÉTAPE 3 — Déployer en ligne

### Option A — Netlify (Recommandé, gratuit)
1. Aller sur **https://netlify.com** → connexion
2. **"Add new site"** → **"Deploy manually"**
3. Glisser-déposer `concours-m2.html` dans la zone de dépôt
4. ✅ Votre test est en ligne !
5. Renommer le site : Site settings → Site name → ex: `ai2-concours-m2`
6. URL : `https://ai2-concours-m2.netlify.app`

> Pour le dashboard admin : déployer `admin-dashboard.html` sur un autre site Netlify
> et **NE PAS** le partager publiquement (protégé par mot de passe).

### Option B — GitHub Pages (gratuit)
1. Créer un repo GitHub privé (⚠️ PRIVÉ pour le dashboard)
2. Uploader les deux fichiers
3. Settings → Pages → Source: main branch
4. URL générée : `https://votrenom.github.io/ai2-concours/concours-m2.html`

### Option C — Hébergement propre (Apache/Nginx)
Copiez simplement les fichiers `.html` dans votre répertoire web (`/var/www/html/`).
Aucune configuration serveur particulière n'est requise (HTML statique).

### Option D — Vercel (gratuit)
```bash
npm install -g vercel
vercel --public  # dans le dossier contenant concours-m2.html
```

---

## 📊 ÉTAPE 4 — Utiliser le Dashboard Admin

### Accès
1. Ouvrir `admin-dashboard.html` dans votre navigateur
2. Saisir le mot de passe défini dans `ADMIN_PASSWORD`
3. Le dashboard se connecte automatiquement à Supabase

### Fonctionnalités
- 📊 **KPIs** : Nombre total, admis, liste complémentaire, score moyen
- 📈 **Graphiques** : Répartition des décisions, scores par section, distribution
- 🔍 **Recherche** : Par nom, prénom, email, université
- 🎯 **Filtres** : Par décision d'admission
- 📋 **Tri** : Par date, score, nom
- 👁️ **Détail** : Vue complète pour chaque candidat
- ⬇️ **Export CSV** : Export filtrable vers Excel

### Accès direct Supabase (optionnel)
Pour une vue brute : Supabase → **Table Editor** → `candidates`

---

## 🔐 Sécurité — Recommandations

| Recommandation | Priorité |
|----------------|----------|
| Changer `ADMIN_PASSWORD` dans le dashboard | 🔴 CRITIQUE |
| Ne jamais committer la `service_role key` sur GitHub public | 🔴 CRITIQUE |
| Utiliser un repo GitHub privé pour les deux fichiers | 🟡 Important |
| Activer HTTPS (automatique sur Netlify/Vercel) | 🟡 Important |
| Restreindre l'accès au dashboard par IP (Netlify Identity) | 🟢 Optionnel |

---

## 📋 Critères d'admission configurés

| Score | Décision |
|-------|----------|
| ≥ 80% (≥ 24/30) | ✅ **Admis en Admission Directe** |
| 60–79% (18–23/30) | 📋 **Liste Complémentaire — Entretien Requis** |
| < 60% (< 18/30) | ❌ **Non Admissible** |

> Ces seuils sont modifiables dans `concours-m2.html` → fonction `getEligibility(pct)`

---

## ✏️ Personnaliser le test

### Modifier les questions
Dans `concours-m2.html`, chercher `const QUESTIONS = [` et modifier :
```javascript
{
  id: 0,        // identifiant unique
  s: 0,         // section (0=Math, 1=ML, 2=DL, 3=Python, 4=SQL)
  text: "Votre question ici...",
  opts: ["Option A","Option B","Option C","Option D"],
  correct: 1    // index de la bonne réponse (0=A, 1=B, 2=C, 3=D)
}
```

### Modifier la durée
Chercher `secondsLeft: 7200` → remplacer `7200` par la durée en secondes.

### Modifier le logo/nom
Chercher `AI2` dans le HTML et remplacer par le nom de votre école.

---

## 🔧 Support & Maintenance

- **Supabase documentation** : https://supabase.com/docs
- **Netlify documentation** : https://docs.netlify.com
- **Voir les soumissions** : Supabase → Table Editor → candidates
- **Requête SQL utile** :
  ```sql
  SELECT prenom, nom, email, percentage, eligibility, created_at
  FROM candidates
  ORDER BY percentage DESC;
  ```

---
*AI2 — Applied Institute of Artificial Intelligence · Concours M2 DS&IA · 2025–2026*
