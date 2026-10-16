# 🤖 CV Matcher Multi-Agents & Multimodal (n8n Workflow)

> **Système d'évaluation décisionnelle autonome de candidatures RH** basé sur une architecture multi-agents, vision multimodale et outils déterministes.

---

## 📋 Table des Matières
1. [Vue d'ensemble du Projet](#-vue-densemble-du-projet)
2. [Architecture du Workflow](#-architecture-du-workflow)
3. [Prérequis](#-prérequis)
4. [Guide d'Installation Étape par Étape](#-guide-dinstallation-étape-par-étape)
5. [Configuration des Identifiants (Credentials)](#-configuration-des-identifiants-credentials)
6. [Configuration du Google Sheet](#-configuration-du-google-sheet)
7. [Guide de Test & Démonstration](#-guide-de-test--démonstration)
8. [Structure du Dépôt Git Recommandée](#-structure-du-dépôt-git-recommandée)
9. [Dépannage & FAQ](#-dépannage--faq)

---

## 🎯 Vue d'ensemble du Projet

Ce projet orchestre un **comité d'évaluation IA** pour qualifier automatiquement des profils candidats :
- **Agent Vision (`gpt-4o-mini`) :** Authentification visuelle et contrôle des certificats/diplômes.
- **Agent Technique (`OpenAI Functions Agent`) :** Analyse des compétences via 3 Custom Tools (équivalences technologiques, arithmétique déterministe sur 50, questions d'entretien ciblées).
- **Agent RH (`Conversational Agent`) :** Analyse de longévité, benchmark salarial et questions comportementales STAR.
- **Agent Décision :** Synthèse pondérée (55% Tech / 45% RH) ramenée sur 100 et émission d'une décision officielle.
- **Intégrations :** Historisation automatique dans **Google Sheets** et envoi d'un **email exécutif stylisé (HTML)** via **Gmail**.

---

## 🏗️ Architecture du Workflow

```
[📥 Formulaire n8n]
    │  ├─ CV du candidat (PDF)
    │  ├─ Certificat / Diplôme (Image PNG)
    │  └─ Offre d'emploi (Texte)
    │
    ├──► [📝 Extraction Texte PDF] ──────────────┐
    ├──► [👁️ Agent Vision (Certificat)] ─────────┼──► [💻 Agent Technique] (3 Tools)
    │                                             │           │
    ├──► [🗄️ Ingestion RAG Vectorielle]           └──► [🤝 Agent RH] (3 Tools)
    │    (Loader + Splitter + Embeddings)                     │
    │                                                         ▼
    └───────────────────────────────────────────────► [🔀 Fusion des Rapports]
                                                              │
                                                              ▼
                                                     [⚖️ Agent Décision] (2 Tools)
                                                              │
                                            ┌─────────────────┴─────────────────┐
                                            ▼                                   ▼
                             [📊 Mise en forme JS]                   [📧 Nœud Gmail]
                                            ▼                         (Dossier HTML)
                             [📈 Google Sheets RH]
```

---

## ⚙️ Prérequis

Avant d'importer le workflow, assurez-vous de disposer de :
1. **Une instance n8n :**
   - Soit [n8n Cloud](https://n8n.io) (recommandé).
   - Soit n8n auto-hébergé via Docker ou npm (`npx n8n`).
2. **Une clé API OpenAI :**
   - Accès aux modèles `gpt-4o-mini` et `text-embedding-3-small`.
3. **Un compte Google :**
   - Pour connecter **Google Sheets** (historique des candidatures).
   - Pour connecter **Gmail** (envoi des fiches d'évaluation).

---

## 🚀 Guide d'Installation Étape par Étape

### 1. Importer le Workflow dans n8n
1. Ouvrez votre tableau de bord n8n.
2. Cliquez sur **Workflows** dans le menu de gauche, puis sur **Add Workflow** (ou le bouton **`+`**).
3. Cliquez sur les **trois petits points (`...`)** en haut à droite du canvas ➔ **Import from File**.
4. Sélectionnez le fichier JSON du projet : `workflow_cv_matcher.json`.
5. Le canvas complet avec tous les agents, outils et connexions apparaît automatiquement !

---

## 🔑 Configuration des Identifiants (Credentials)

Après l'import, configurez les 3 comptes de connexion :

### 1. Clé API OpenAI (Nœuds IA & Vision)
- Double-cliquez sur l'un des nœuds OpenAI (ex: `OpenAI (Technique)`).
- Dans le menu **Credential**, cliquez sur **Create New Credential** :
  - **API Key :** Collez votre clé `sk-...`
  - Cliquez sur **Save**.
- Cette même clé s'applique automatiquement à tous les sous-nœuds du workflow (`Vision`, `Embeddings`, `Agent RH`, `Agent Décision`).

### 2. Google Sheets OAuth2 (Persistance des données)
- Double-cliquez sur le nœud **`Google Sheets`**.
- Dans **Credential**, sélectionnez **Google Sheets OAuth2 API** ➔ **Connect my account**.
- Connectez-vous avec votre compte Google et cochez l'autorisation d'écriture dans Google Drive / Sheets.

### 3. Gmail OAuth2 (Envoi des emails de synthèse)
- Double-cliquez sur le nœud **`Gmail`**.
- Dans **Credential**, sélectionnez **Gmail OAuth2** ➔ **Connect my account**.
- Autorisez n8n à envoyer des messages depuis votre boîte de réception.

---

## 📊 Configuration du Google Sheet

1. Créez une nouvelle feuille de calcul sur [Google Sheets](https://sheets.google.com).
2. Nommez la première feuille (onglet) : `Candidatures`.
3. Sur la **Ligne 1**, copiez-collez exactement ces 8 en-têtes de colonnes :
   ```
   Date | Candidat | Poste | Score Tech | Score RH | Score Final | Recommandation | Feedback
   ```
4. Dans votre workflow n8n, double-cliquez sur le nœud **`Google Sheets`** :
   - **Document :** Sélectionnez votre feuille dans la liste déroulante (ou collez son URL).
   - **Sheet :** Sélectionnez `Candidatures`.

---

## 🧪 Guide de Test & Démonstration

Le dépôt inclut deux jeux de données d'évaluation types :

### Profil Test 1 : Sarah Martin (Profil Intermédiaire)
- **Fichiers à fournir :**
  - **CV :** `tests/CV_Sarah_Martin.pdf`
  - **Certificat :** `tests/Certificat_Sarah_Martin.png`
  - **Offre d'emploi :**
    ```text
    Poste : Data Scientist Senior
    Compétences obligatoires : Python, SQL, MLflow, Kubeflow, min 4 ans d'expérience.
    ```
- **Résultat attendu :**
  - Score Tech : ~`25-31 / 50` (Manque MLflow et Kubeflow).
  - Score RH : `45 / 50` (Stabilité 5 ans).
  - Score Final : ~`68-74 / 100` $\rightarrow$ **Badge Orange : ENTRETIEN COMPLÉMENTAIRE**.
  - Questions ciblées sur l'apprentissage des outils MLOps manquants.

---

### Profil Test 2 : Thomas Dubois (Profil Excellent)
- **Fichiers à fournir :**
  - **CV :** `tests/CV_Thomas_Dubois.pdf`
  - **Certificat :** `tests/Certificat_Thomas_Dubois.png`
  - **Offre d'emploi :**
    ```text
    Poste : Lead Data Scientist & MLOps Senior
    Compétences obligatoires : Python, SQL, MLflow, Kubeflow, Docker, AWS.
    ```
- **Résultat attendu :**
  - Score Tech : **`50 / 50`** (100% de matching + Certificat AWS).
  - Score RH : **`45 / 50`** (Stabilité 6 ans).
  - Score Final : **`95.5 / 100`** $\rightarrow$ **Badge Vert : RECRUTER**.
  - Questions d'architecture senior de haut niveau.

---

## 📁 Structure du Dépôt Git Recommandée

Organisez votre dépôt Git comme suit pour une clarté maximale :

```
cv-matcher-multiagents/
│
├── workflow_cv_matcher.json       # Export officiel du workflow n8n
├── README.md                      # Guide d'installation et d'exécution (ce fichier)
├── MASTER_BRIEF.md                # Spécification technique et architecture de référence
│
└── tests/                         # Données de test prêtes à l'emploi
    ├── CV_Sarah_Martin.pdf
    ├── Certificat_Sarah_Martin.png
    ├── CV_Thomas_Dubois.pdf
    └── Certificat_Thomas_Dubois.png
```

---

## ❓ Dépannage & FAQ

#### 1. L'URL du formulaire n'est pas accessible ?
> Dans n8n, assurez-vous d'avoir cliqué sur **`Test workflow`** au centre du canvas pour activer l'écoute du Webhook en mode test, OU activez le workflow (interrupteur en haut à droite) pour utiliser la **Production URL**.

#### 2. Les outils de l'Agent Technique ne s'allument pas ?
> Vérifiez que l'Agent Technique est bien configuré en mode **`OpenAI Functions Agent`** (et non `Conversational Agent`). Ce mode force l'appel déterministe via le Function Calling natif d'OpenAI.

#### 3. L'email n'est pas reçu ?
> Vérifiez dans les options du nœud Gmail que le champ **To** pointe bien vers votre adresse de test et que **Email Type** est configuré sur **`HTML`**.
