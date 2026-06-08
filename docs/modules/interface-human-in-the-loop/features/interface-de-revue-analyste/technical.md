## 1. Composants impactés

### 1.1 Backend (FastAPI)

Endpoints concernés :

- `GET /reviews/documents/{id}`
  - Retourne :
    - Métadonnées du document
    - Données extraites (structured_data + confidence_scores)
    - Score global et sous-scores
    - Règles appliquées
    - Version des modèles

- `GET /documents/{id}/file-url` (ou équivalent)
  - Génère une URL S3 pré-signée temporaire pour affichage sécurisé.

Contraintes :
- Vérification systématique du JWT.
- Vérification du rôle (RBAC).
- Vérification du `tenant_id`.

---

### 1.2 Base de données (PostgreSQL)

Tables utilisées :

- `document`
- `extraction_result`
- `fraud_score`
- `review` (lecture seule dans ce contexte)
- `field_correction` (pour refléter la version actuelle des données)

Logique :
- Récupération des données via jointures par `document_id`.
- Filtrage obligatoire par `tenant_id`.
- Les données retournées doivent inclure les versions de modèles (`model_version`).

---

### 1.3 Frontend

Composants principaux :

- DocumentViewer
  - Intégration d’un visualiseur PDF/image.
  - Support zoom et multi-pages.

- ExtractionPanel
  - Affichage structuré dynamique selon type de document.
  - Mise en évidence des champs à faible confiance.

- ScorePanel
  - Affichage du score global.
  - Visualisation des sous-scores (barres ou indicateurs).

- ExplainabilityPanel
  - Liste des règles déclenchées.
  - Anomalies détectées.
  - Informations sur version modèle.

Contraintes :
- Chargement asynchrone des différentes sections.
- Gestion des états (loading, error, not found).
- Temps de réponse cible < 2 secondes (hors téléchargement fichier volumineux).

---

## 2. Sécurité

- Authentification via OAuth2/OIDC (JWT).
- Contrôle d’accès basé sur rôle.
- Vérification systématique du `tenant_id` côté backend.
- Accès au fichier via URL pré-signée S3 à durée limitée.
- Journalisation des accès à la fiche document.

---

## 3. Intégration au pipeline

- La fiche est accessible uniquement si le document a un statut compatible (`pending_review`, `validated`, `rejected`).
- Les données affichées correspondent à l’état persistant généré par le pipeline.
- Aucune logique de recalcul de score n’est exécutée dans cette fonctionnalité.

---

## 4. Contraintes non fonctionnelles

- Pagination et lazy loading si nécessaire pour documents volumineux.
- Architecture compatible multi-tenant à grande échelle.
- Conformité RGPD : respect des politiques de rétention (document non accessible si supprimé).
- Résilience aux accès concurrents.

---

## 5. Dépendances techniques

- Backend FastAPI
- PostgreSQL
- Stockage S3 compatible
- Service d’authentification (OAuth2/OIDC)
- Module Audit & Traçabilité

La fonctionnalité reste découplée du moteur IA : elle consomme uniquement les résultats persistés en base.